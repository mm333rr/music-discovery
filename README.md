# music-discovery

Claude skill — AI-powered music library discovery for The Capes homelab.

Analyzes Paco's Plex/Lidarr music library, derives its taste DNA, discovers
missing artists matching user-defined conditions, adds them to Lidarr, and
triggers Prowlarr searches.

## Infrastructure
- Plex music section: key=12, token in plex-collections/.env
- Lidarr: http://localhost:8686 (mbuntu via SSH)
- Prowlarr: http://localhost:9696 (mbuntu via SSH)
- Root folder: /tank/music/managed

## Trigger conditions
Use when user says: "find me artists I'm missing", "explore my library",
"what should I add", "who influenced X", "find pre-[era] artists",
"expand my ambient collection", "do music discovery", "add more artists like X",
"what's missing from my library", or any audit/explore/expand request.

## Structure
```
music-discovery/
├── SKILL.md                       ← Core skill (auto-injected)
├── references/
│   └── discovery-strategies.md   ← Condition-type playbooks
├── README.md
└── CHANGELOG.md
```

## Install
Package with skill-creator and upload to claude.ai → Settings → Skills.

## Update workflow
1. Edit SKILL.md / references/
2. git add -A && git commit -m "feat: <desc>" && git push
3. Repackage and re-upload
