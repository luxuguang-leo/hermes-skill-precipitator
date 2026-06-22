# Hermes Skill Evolution

> Full lifecycle management for Hermes Agent skills — discover, forge, maintain, and retire.

**Zero modifications to Hermes core.** Two subsystems work together:

| Subsystem | Direction | Goal |
|-----------|-----------|------|
| **Mining** | Bottom-up | Discover patterns → create skills |
| **Reflection** | Top-down | Clean stale → maintain health |

```
sessions → mining → skill → usage → reflection → archive
               ↑                           ↓
               └── self-evolution loop ────┘
```

---

## Mining — Auto-Discover New Skills

Mines session history for repeated workflow patterns, clusters them, and generates SKILL.md candidates.

```
945 sessions → 155 cases → 14 skill candidates (real data run)
```

### Components

| Component | What it does | When |
|-----------|-------------|------|
| **Hook** | Incrementally scans new sessions, extracts cases, checks threshold | Cron daily 12:00, silent |
| **CLI** | Scan, cluster, forge, report, install | Manual |
| **Case DB** | Accumulated case files with tool signatures, user intent, n-gram patterns | Persistent |

### Clustering Algorithm

Four weighted dimensions determine case similarity:

| Dimension | Weight | What it captures |
|-----------|--------|-----------------|
| Tool signature | 0.30 | 8 categories: SHELL, BROWSER, CODE, WEB, FILE, CRON, EMAIL, NOTIFY |
| User intent | 0.30 | install-setup, research, fix-debug, search-find, etc. |
| N-gram sequence | 0.20 | Bigram/trigram of tool categories (e.g. `SHELL>SHELL>FILE`) |
| Keyword similarity | 0.20 | Chinese word tokenization + Jaccard similarity |

---

## Reflection — System Health Maintenance

Three operations keep Hermes running lean.

| Operation | What it does |
|-----------|-------------|
| **Scan** | Weekly health check: memory, zombies, kanban |
| **Consolidate** | Memory dedup: merge, compress, rollback-safe |
| **Evolve** | Archive stale skills, check crons, automate maintenance |

### Key Metrics

| Metric | Alert threshold |
|--------|----------------|
| Memory water level | >80% of 2.2KB / 1.4KB limit |
| Zombie skills | SKILL.md unmodified >60d (excl. stable) |
| Kanban blockers | Task stuck >48 hours |
| Session activity | 7-day count >200 or <10 |

---

## Cron Schedule

| Cron | Schedule | Purpose |
|------|----------|---------|
| `skill-evolution-hook` | Daily 12:00 | Silently accumulate new cases |
| `unified-weekly-maintenance` | Sunday 03:00 | Full scan + report only if issues |

---

## Quick Start

```bash
git clone git@github.com:luxuguang-leo/hermes-skill-evolution.git
cd hermes-skill-evolution
python3 install.py

# Mining: discover patterns
python3 scripts/skill_evolution.py scan --limit 500
python3 scripts/skill_evolution.py cluster --threshold 0.45
python3 scripts/skill_evolution.py forge --min-cases 3
python3 scripts/skill_evolution.py install <candidate-name>

# Reflection: maintain health
python3 scripts/reflection/scan.py --days 7
python3 scripts/reflection/consolidate.py --auto-apply
python3 scripts/reflection/evolve.py --report

# Daily hook (add to cron)
python3 scripts/skill_evolution_hook.py
```

---

## File Structure

```
~/.hermes/
├── scripts/
│   ├── mining/              # Pattern discovery
│   │   ├── miner.py, forge.py, hook.py, validator.py, signatures.py
│   │   └── ARCHITECTURE.md
│   ├── reflection/          # Health maintenance
│   │   ├── scan.py, consolidate.py, evolve.py
│   ├── skill_evolution.py
│   └── skill_evolution_hook.py
├── reflection/scan-report.json
├── agent/cases/ candidates/ .case_index.json
└── skills/hermes/skill-evolution/ .archive/
```

---

## Design Principles

1. **Silent by default** — Only speak when actionable.
2. **User gates all actions** — No auto-delete, no auto-create.
3. **Zero core modifications** — No changes to `run_agent.py` or gateway.
4. **Lossless archives** — Archived skills can always be restored.

---

## License

MIT — [github.com/luxuguang-leo/hermes-skill-evolution](https://github.com/luxuguang-leo/hermes-skill-evolution)
