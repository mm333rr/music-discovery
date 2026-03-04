# Discovery Strategies Reference

Detailed playbooks for each discovery condition type.

---

## Era-Based Discovery

**Goal:** Find foundational artists from a specific time period whose work
directly feeds into genres/aesthetics already in the library.

**Steps:**
1. Identify which genres in the library have deep historical roots in the target era
2. Find the "missing roots" -- genre originators active during that period
3. Trace the influence chain: [era artist] -> [intermediate] -> [library artist]
4. Verify the connection is specific and articulable (not just genre overlap)

**Example:** Library has Boards of Canada -> trace back -> Tangerine Dream already
there -> trace back to pre-1950 -> Edgard Varese (noise/electronic precursor)

**Pitfalls:**
- Don't add classical composers just because they're pre-1950 if there's no
  genuine connection to the library's actual aesthetic
- Avoid "hall of fame" choices -- only add if the connection is genuine
- Classical catalogs are enormous; flag for user to configure monitoring scope

---

## Genre-Based Discovery

**Goal:** Find artists in adjacent or foundational genres not yet represented.

**Steps:**
1. Map the library's genre distribution
2. Identify adjacent genres that are completely absent
3. Find the 3-5 most essential artists for each absent adjacent genre
4. Prioritize artists who bridge between the absent genre and existing library genres

**Adjacency map (partial):**
- Ambient -> Drone, Dark Ambient, New Age, Lowercase, Field Recording
- Electronic -> IDM, Glitch, Microsound, Electro-Acoustic, Club
- Jazz -> Free Jazz, Modal, Bebop, Cool, Latin Jazz, Jazz-Funk
- Classical -> Minimalism, Spectralism, Serialism, Impressionism, Romantic
- Post-Rock -> Math Rock, Slowcore, Shoegaze, Noise Rock
- Soul/R&B -> Gospel, Funk, Blues, Doo-Wop, Swing

---

## Influence Chain Discovery

**Goal:** Walk backwards (or forwards) through an influence chain from existing artists.

**Backwards (roots):** Who influenced this artist?
**Forwards (branches):** Who did this artist inspire who isn't in the library?
**Lateral (peers):** Who was making similar work at the same time?

**Steps:**
1. Pick 3-5 anchor artists from the library
2. For each, identify 2-3 direct influences/influencees not in the library
3. Cross-check -- any name appearing multiple times is a high-priority add
4. Verify they're not already in Lidarr under a different name variant

**Example chains from The Capes library:**
- Brian Eno -> Harold Budd (already in) -> Laraaji (already in) -> Terry Riley (missing root)
- Burial -> Massive Attack (in) -> Portishead (in) -> Tricky (missing lateral)
- Nujabes -> J Dilla (not in library) -> Madlib (not in library)
- Bjork -> The Sugarcubes (early Bjork) -> Mum (Icelandic peer, missing)

---

## Geographic Discovery

**Goal:** Find artists from geographic scenes/movements already represented.

**Steps:**
1. Cluster existing library artists by country/city/scene
2. Identify which geographic clusters are well-represented at the "branch" level
   but missing key scene originators or peers
3. Find 3-5 essential artists per under-represented scene

**Existing geographic clusters in The Capes library:**
- Iceland: Bjork, Sigur Ros, Olafur Arnalds, Johann Johannsson -> missing: Mum, Amiina, Sin Fang
- Japan: YMO, Hiroshi Yoshimura, Ryuichi Sakamoto, Nujabes -> missing: Haruomi Hosono (solo), Midori Takada, Susumu Yokota
- UK (Bristol): Massive Attack, Portishead, Tricky -> all 3 in library
- UK (experimental): Burial, Four Tet, Aphex Twin -> missing: Autechre, u-Ziq, Squarepusher
- Germany: Tangerine Dream, Klaus Schulze, Gas -> missing: Cluster, Harmonia, Neu!, Ash Ra Tempel
- France: Daft Punk, Air, Yelle -> missing: Serge Gainsbourg, Brigitte Fontaine, Bernard Parmegiani

---

## Collaborator Discovery

**Goal:** Find artists who collaborated with existing library artists but aren't in the library.

**Steps:**
1. For each anchor artist, look up notable collaborators
2. Filter to collaborators who have significant solo/lead work
3. Add those who fit the library's aesthetic even outside the collaboration

**Known collaborator gaps in The Capes library:**
- Brian Eno collaborated with -> David Bowie (Berlin trilogy), Robert Fripp, Daniel Lanois
- Alva Noto + Ryuichi Sakamoto -> Fennesz (collaborated on Sala Santa Cecilia)
- Massive Attack -> Elizabeth Fraser, Horace Andy, Daddy G solo
- Nicolas Jaar -> Dave Harrington (Darkside), Against All Logic (Jaar alias)

---

## Representation Audit Discovery

**Goal:** Surface meaningful gaps in artist representation by gender, geography,
language, or identity within genres already in the library.

**Steps:**
1. For each genre cluster, note gender/identity distribution of existing artists
2. Identify genres where representation is skewed
3. Find excellent artists who fill genuine gaps (quality first, representation framing)

**Current library notes:**
- Electronic/ambient is male-dominated -> missing: Matmos, Caterina Barbieri, Grouper (in),
  Puce Mary, Moor Mother, Actress (in)
- Jazz discovery is all male -> missing: Mary Lou Williams, Abbey Lincoln,
  Alice Coltrane, Carla Bley, Marian McPartland
- Classical is all male -> Ruth Crawford Seeger added, missing: Clara Schumann,
  Kaija Saariaho, Sofia Gubaidulina, Eliane Radigue

---

## Mood/Texture Discovery

**Goal:** Identify emotional or textural gaps in the library.

**Library mood map:**
- Melancholic/introspective: Grouper, Weyes Blood, Lana Del Rey, Mitski, Bon Iver -> well represented
- Abrasive/industrial: Nine Inch Nails, Ben Frost, Arca -> partially represented
- Euphoric/ecstatic: Robyn, Charli xcx, Lady Gaga -> pop only; missing electronic euphoria (Underworld, Orbital)
- Meditative/devotional: Hammock, Stars of the Lid, Nils Frahm -> well represented
- Playful/absurdist: Laurie Anderson -> almost entirely missing; missing: The Residents, Moondog, Raymond Scott
- Ritualistic/ceremonial: Johann Johannsson, Olafur Arnalds -> partial; missing: Dead Can Dance, Lisa Gerrard

---

## Quick Candidate Checklist

Before adding any artist, verify:
- [ ] Not already in Plex (case-insensitive check)
- [ ] Not already in Lidarr (case-insensitive check, also check alternate name spellings)
- [ ] Has a genuine articulable connection to >= 1 existing library thread
- [ ] MusicBrainz has meaningful catalog data (>0 albums)
- [ ] MBID confirmed via lidarr:{mbid} lookup for classical/jazz (not just name search)
- [ ] Disambiguation field checked if multiple artists share the name
