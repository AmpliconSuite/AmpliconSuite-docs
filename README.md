# AmpliconSuite Docs

This repository contains shared documentation for file formats and conventions used across AmpliconSuite tools.

Start with the canonical file format documentation:

- [File formats overview](docs/file_formats/index.md)
- [Cycles file](docs/file_formats/cycles.md)
- [Graph file](docs/file_formats/graph.md)
- [Copy-number bedGraph](docs/file_formats/bedgraph.md)
- [Amplicon intervals BED](docs/file_formats/intervals_bed.md)
- [Sashimi and cycle visualizations](docs/file_formats/visualizations.md)

## Agent skills

`skills/` holds [agent skills](https://agentskills.io) that teach AI coding
assistants (Claude Code, Codex, Cursor and others) to use AmpliconSuite tools
correctly.

| Skill | What it does |
|---|---|
| [`amplicon-repository`](skills/amplicon-repository/SKILL.md) | Answer questions from [AmpliconRepository](https://ampliconrepository.org) through its REST API |

Install for every agent on your machine:

```bash
npx skills add AmpliconSuite/AmpliconSuite-docs -g
```

Add `--skill <name>` to install one skill only.
