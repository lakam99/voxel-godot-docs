# Voxel Biome World documentation

This repository is the long-form documentation library for [Voxel Biome World Godot](https://github.com/lakam99/voxel-godot). It holds architecture notes, design references, implementation plans, migration records, and verification reports that are useful to preserve but would obscure the game's concise, current developer guidance.

## Start here

- [Architecture and authority](architecture/README.md)
- [Gameplay systems](gameplay/README.md)
- [World generation](world-generation/README.md)
- [Migrations and streaming](migrations/README.md)
- [Citadel engineering record](engineering/citadel/README.md)
- [Game design and story](game-design/README.md)
- [Art direction and visual record](art-direction/README.md)
- [Systems and procedural ecology](systems/README.md)
- [Source index](catalog/source-index.csv)

## Which source is authoritative?

The game repository's [`AGENTS.md`](https://github.com/lakam99/voxel-godot/blob/master/AGENTS.md), current code, and tests define current behavior. This repository preserves plans and evidence as they stood at their recorded dates; a historical report is not proof that its proposal shipped or remains current. Prefer the current runtime and the source repository's active contract docs whenever they differ from an archived report.

## Initial import

The first collection was reorganized from game repository commit [`cd2ac5e9ce2bdcd35280819c64ba77deede9430d`](https://github.com/lakam99/voxel-godot/tree/cd2ac5e9ce2bdcd35280819c64ba77deede9430d). `catalog/source-index.csv` records each imported document's original path, destination, source commit, and source hash. The original snapshot remains in the game repository's Git history; this repository provides a navigable, categorized copy. The reorganized game-repository documentation is at commit [`997a585d155a0534d3fe3847fae70bc769e3e325`](https://github.com/lakam99/voxel-godot/tree/997a585d155a0534d3fe3847fae70bc769e3e325).

## Contributing

Keep current, short operational guidance next to the code it governs. Put dated investigations, phase reports, migration evidence, and superseded plans here. Use descriptive lowercase filenames and stable topic folders; include the date in a filename only when chronology is part of the document's identity. When adding an imported or moved document, update the source index and the relevant topic index.

## External generated artifacts not preserved

Some historical reports link to run outputs under the game's `artifacts/` directory. Generated, untracked outputs were not part of the committed documentation snapshot and are not included here. Links redirected to this note identify references whose original generated payload is unavailable; the source report and its recorded conclusions are preserved.
