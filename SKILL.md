---
name: music-discovery
description: >
  Analyzes the current state of Paco's Plex/Lidarr music library, identifies the
  library's taste DNA and aesthetic threads, then discovers and adds new artists
  that are missing but belong there -- filtered by any condition the user specifies
  (era, genre, geography, gender, influence chain, collaborators, mood, instrument,
  label, etc.). After discovery, automatically adds artists to Lidarr and triggers
  Prowlarr searches. Use this skill whenever the user says anything like: "find me
  artists I'm missing", "explore my library", "what should I add", "who influenced X
  in my library", "find pre-[era] artists", "expand my jazz/classical/ambient
  collection", "do music discovery", "add more artists like X", "what's missing from
  my library", or any request to audit/explore/expand the music collection. Also use
  when the user asks to search for specific albums or artists in Prowlarr or Lidarr.
---

# Music Discovery Skill

Analyzes Paco's Plex music library, derives its taste DNA, finds missing artists
matching a user-defined condition, adds them to Lidarr, and triggers downloads.

**Infrastructure:** See capes-skill for full homelab reference. Quick facts:
- Plex music section key: **12**, token: `XDMas6E2Nq7-MCGkeeDE`, URL: `http://localhost:32400`
- Lidarr: `http://localhost:8686`, API key: `62a5ca4b92a841c19066b4472763fbef` (on mbuntu via SSH)
- Prowlarr: `http://localhost:9696`, API key: `29591703fa374f098085bdb6b5e22b6f` (on mbuntu)
- Root folder: `/tank/music/managed`, Quality profile ID: 1 (Any), Metadata profile ID: 1 (Standard)
- All Lidarr/Prowlarr calls run via: `ssh mbuntu "python3 << 'EOF' ... EOF"`

---

## Phase 1 -- Read the Library

Always start by pulling the live artist list from Plex (source of truth) AND Lidarr.

**CRITICAL:** Build the exclusion set from BOTH names (lowercase) AND MusicBrainz IDs.
Lidarr stores many artists in native script (Cyrillic, Arabic, etc.) so name-only
matching will miss them. MBIDs are script-agnostic.

```python
import urllib.request, xml.etree.ElementTree as ET, json
PLEX_URL = 'http://localhost:32400'
PLEX_TOKEN = 'XDMas6E2Nq7-MCGkeeDE'
with urllib.request.urlopen(f'{PLEX_URL}/library/sections/12/all?X-Plex-Token={PLEX_TOKEN}') as r:
    root = ET.parse(r).getroot()
plex_artists = {d.get('title', '').lower() for d in root.findall('Directory')}

LIDARR = 'http://localhost:8686/api/v1'
KEY = '62a5ca4b92a841c19066b4472763fbef'
req = urllib.request.Request(f'{LIDARR}/artist', headers={'X-Api-Key': KEY})
with urllib.request.urlopen(req) as r:
    lidarr_artists = json.load(r)
lidarr_names = {a['artistName'].lower() for a in lidarr_artists}
lidarr_mbids = {a.get('foreignArtistId', '') for a in lidarr_artists}
exclusion_names = plex_artists | lidarr_names
print(f'Plex: {len(plex_artists)} | Lidarr: {len(lidarr_artists)} | Exclusion: {len(exclusion_names)} names + {len(lidarr_mbids)} MBIDs')
```

---

## Phase 2 -- Analyze Taste DNA

Before any discovery, derive the library's aesthetic threads. Look for:
- **Genre clusters** -- what genres dominate? What subgenres?
- **Era clusters** -- what decades/periods are represented?
- **Mood/texture** -- ambient, kinetic, melancholic, euphoric, abrasive?
- **Production aesthetic** -- lo-fi, maximalist, electronic, acoustic, orchestral?
- **Influence chains** -- who are the "roots" artists vs "branches"?
- **Geographic clusters** -- UK, Japan, US, Iceland, etc.?
- **Missing roots** -- genres well-represented at "branch" level but missing foundational roots?

Write this analysis out explicitly before generating any candidates.

---

## Phase 3 -- Discover Missing Artists

Apply the user's condition as a filter over the taste DNA analysis.

Rules:
1. Must not be in exclusion set (check name AND MBID)
2. Must have a genuine, articulable connection to an existing library thread
3. Connection must be specific -- not "they're both jazz" but cite actual influence
4. Aim for 15-25 candidates
5. Prioritize depth -- a foundational artist with 5 connections beats a tangential one

For each candidate: name, active years, library thread connections, why they belong,
and a "start with" album recommendation.

---

## Phase 4 -- Add to Lidarr

**Batch size warning:** Adding >20 artists at once backs up Lidarr's serial command
queue for 2-4 hours. Add in groups of 15-20. Confirm queue health after each group.

**Use name-based lookup as primary.** `lidarr:{mbid}` prefix is unreliable in SSH
heredocs due to colon-escaping. Always inspect returned name + disambiguation field.

```python
# Run on mbuntu via: ssh mbuntu "python3 << 'EOF' ... EOF"
# HEREDOC SAFETY: use 'EOF' (single-quoted). Never use emoji -- causes SyntaxError.
# Status strings: ADDED, EXISTS, FAILED, NOT_FOUND, WARNING (plain ASCII only)

import urllib.request, json, urllib.parse, time

LIDARR = 'http://localhost:8686/api/v1'
KEY = '62a5ca4b92a841c19066b4472763fbef'
ROOT_FOLDER = '/tank/music/managed'
QUALITY_PROFILE_ID = 1
METADATA_PROFILE_ID = 1

def lidarr_get(path):
    req = urllib.request.Request(f'{LIDARR}{path}', headers={'X-Api-Key': KEY})
    with urllib.request.urlopen(req) as r:
        return json.load(r)

def lookup_artist(name, disambig_hint=None):
    # Name-based lookup with disambiguation verification.
    # disambig_hint: substring to match in the disambiguation field (optional).
    # Falls through top 5 results. On 503 rate limit: sleeps 10s, retries once.
    q = urllib.parse.quote(name)
    try:
        results = lidarr_get(f'/artist/lookup?term={q}')
    except urllib.error.HTTPError as e:
        if e.code == 503:
            print(f'Rate limited on {name}, waiting 10s...')
            time.sleep(10)
            results = lidarr_get(f'/artist/lookup?term={q}')
        else:
            raise
    if not results:
        return None
    if disambig_hint:
        for r in results[:5]:
            if disambig_hint.lower() in r.get('disambiguation', '').lower():
                return r
    first = results[0]
    first_name = first.get('artistName', '').lower()
    search_words = [w for w in name.lower().split() if len(w) > 3]
    if any(word in first_name for word in search_words):
        return first
    print(f'WARNING: {name} returned [{first.get("artistName")}] ({first.get("disambiguation","")}) - check manually')
    return first

def add_artist(artist_data):
    artist_data.update({
        'qualityProfileId': QUALITY_PROFILE_ID,
        'metadataProfileId': METADATA_PROFILE_ID,
        'rootFolderPath': ROOT_FOLDER,
        'monitored': True,
        'addOptions': {'monitor': 'all', 'searchForMissingAlbums': True}
    })
    body = json.dumps(artist_data).encode()
    req = urllib.request.Request(
        f'{LIDARR}/artist', data=body, method='POST',
        headers={'X-Api-Key': KEY, 'Content-Type': 'application/json'}
    )
    try:
        with urllib.request.urlopen(req) as r:
            resp = json.load(r)
            return True, resp.get('artistName', '?')
    except urllib.error.HTTPError as e:
        body_text = e.read().decode()
        already = 'already' in body_text.lower() or 'duplicate' in body_text.lower()
        return False, 'EXISTS' if already else body_text[:120]

# ARTISTS: list of (display_name, search_term, disambig_hint_or_None)
ARTISTS = [
    # ('Miles Davis', 'Miles Davis', 'jazz trumpeter'),
]

results_log = []
for display_name, search_term, disambig in ARTISTS:
    artist_data = lookup_artist(search_term, disambig)
    time.sleep(1.5)
    if not artist_data:
        print(f'NOT_FOUND: {display_name}')
        results_log.append((display_name, 'NOT_FOUND'))
        continue
    returned_name = artist_data.get('artistName', '?')
    mbid = artist_data.get('foreignArtistId', '?')
    ok, msg = add_artist(artist_data)
    if ok:
        print(f'ADDED: {returned_name} (mbid={mbid})')
        results_log.append((display_name, 'ADDED', returned_name))
    elif msg == 'EXISTS':
        print(f'EXISTS: {display_name}')
        results_log.append((display_name, 'EXISTS', returned_name))
    else:
        print(f'FAILED: {display_name}: {msg}')
        results_log.append((display_name, 'FAILED', ''))
    time.sleep(1.5)

print('\n=== SUMMARY ===')
for entry in results_log:
    name, status = entry[0], entry[1]
    stored = entry[2] if len(entry) > 2 else ''
    print(f'{status:10} {name}  ->  {stored}')
```

---

## Phase 5 -- Trigger Searches

```python
req = urllib.request.Request(f'{LIDARR}/artist', headers={'X-Api-Key': KEY})
with urllib.request.urlopen(req) as r:
    all_artists = json.load(r)

added_stored_names = {entry[2].lower() for entry in results_log if entry[1] == 'ADDED' and len(entry) > 2}
new_artists = [a for a in all_artists if a['artistName'].lower() in added_stored_names]

print(f'Triggering searches for {len(new_artists)} artists...')
for a in new_artists:
    body = json.dumps({'name': 'ArtistSearch', 'artistId': a['id']}).encode()
    req = urllib.request.Request(f'{LIDARR}/command', data=body, method='POST',
        headers={'X-Api-Key': KEY, 'Content-Type': 'application/json'})
    with urllib.request.urlopen(req) as r:
        cmd = json.load(r)
    print(f'  Search: {a["artistName"]} cmd#{cmd.get("id")}')
    time.sleep(0.3)

body = json.dumps({'name': 'DownloadedAlbumsScan'}).encode()
req = urllib.request.Request(f'{LIDARR}/command', data=body, method='POST',
    headers={'X-Api-Key': KEY, 'Content-Type': 'application/json'})
with urllib.request.urlopen(req) as r:
    cmd = json.load(r)
print(f'Import scan: cmd#{cmd.get("id")}')
```

---

## Phase 6 -- Verify and Report

**IMPORTANT: 0 albums immediately after adding is NORMAL.** RefreshArtist commands
are queued; MusicBrainz indexing is async. Only flag as a problem after 2+ hours.

**Check command queue health first** (not album counts):

```python
# NOTE: /api/v1/command returns a BARE LIST -- do NOT call .get('records') on it.
req = urllib.request.Request(f'{LIDARR}/command?pageSize=100', headers={'X-Api-Key': KEY})
with urllib.request.urlopen(req) as r:
    cmds = json.load(r)
from collections import Counter
print(f'Queue: {len(cmds)} total | By status: {dict(Counter(c.get("status","?") for c in cmds))}')
# Healthy: started=1-3, queued=N. Problem: started=0, queued>0 -> restart Lidarr
```

---

## Condition Types and Discovery Strategies

See `references/discovery-strategies.md` for detailed playbooks per condition type.

| Condition | Strategy |
|---|---|
| **Era** | Historical roots of existing genres; foundational figures from that period |
| **Genre** | Map existing artists to adjacent genres; find genre originators |
| **Geography** | Cluster existing artists by country; find scene peers and predecessors |
| **Influence chain** | Pick an existing artist; trace backwards through their influences |
| **Collaborators** | Find who existing artists collaborated with that isn't in the library |
| **Gender/identity** | Audit representation gaps in existing genre clusters |
| **Instrument** | Find artists centered on underrepresented instruments |
| **Label** | Find label-mates of existing artists on historically significant labels |
| **Mood** | Identify emotional register gaps in the library |

---

## Quality Profile Guidance

- **"Any" (ID: 1):** Grabs fastest available. Good for discovery/exploration.
- **"Lossless" (ID: 2):** FLAC only. Strongly recommended for classical and jazz.
- **"Standard" (ID: 3):** MP3 320. Middle ground.

---

## Known Quirks (Battle-Tested)

**Rate limiting (503):** Always sleep 1.5s between lookups. On 503, sleep 10s, retry once.

**Large batches back up the queue:** 40 artists = ~100+ commands = 2-4 hours to clear. Keep adds to <=20.

**`/api/v1/command` is a bare list:** Do NOT call `.get('records')` on it. Iterate directly.

**`lidarr:{mbid}` unreliable in SSH heredocs:** Use name-based lookup. Write a temp file if MBID lookup needed.

**Non-Latin stored names:** Shostakovich = Cyrillic, Umm Kulthum = Arabic. Always check MBID set too.

**Wrong first result:** "Leadbelly" returns "Leadbelly Legacy Band". Always verify name plausibility. Pass disambig_hint for ambiguous searches.

**0 albums after add is normal:** Recheck after 30-60 min. Flag only after 2+ hours.

**Emoji in SSH heredoc crashes Python:** Use plain ASCII status strings only.

**Prowlarr direct `/search` times out:** Use Lidarr ArtistSearch command instead.

**results_log must track stored names:** Log the returned name from POST response (not input name). Use it to match artists in Phase 5.
