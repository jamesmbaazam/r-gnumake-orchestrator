# Changelog

All notable changes to this project are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project uses [Semantic Versioning](https://semver.org/).

## [1.0.4] - 2026-11-08

### Added
- README section "What the skill covers", summarising the pipeline patterns and the tidyverse style and design guidance, with data.table reserved for intensive data manipulation.

### Removed
- `QUICK-REFERENCE.md` and `INDEX.md`. `SKILL.md` and `README.md` are now the only guides.

### Changed
- Convention is now one script, one output: the Pipeline Scripts example writes only `model.rds` and the metadata moves to its own `summarise_training.R` script. The multi-output Make rules are replaced by a "One Output per Script" pattern, and the checklist asks for exactly one output file.
- Renamed the "Core Contract" section in `SKILL.md` to "R Scripts and Makefile Conventions".
- Renamed "R Script Interface" to "R Script Pattern".
- The Pipeline Scripts example now uses the same `.args` pattern as the R Script Pattern section.
- README: the "Alternatives to GNU Make" table is now collapsible.
- README title is now "R and GNU Make Orchestrator" instead of the repository name.
- README intro now links to GNU Make.
- Removed the specific GNU Make version (3.81) from the README, `SKILL.md` and `manifest.json` descriptions.

## [1.0.3] - 2026-11-08

### Changed
- README: removed the "Three Core Rules", "Start Here", "When to Use" and "Quick Example" sections, and made "Resources to learn Make" a plain section instead of a collapsible one.

## [1.0.2] - 2026-11-08

### Added
- "Using GNU Make to manage the workflow of data analysis projects" (Journal of Statistical Software) to the Make learning resources.

## [1.0.1] - 2026-11-08

### Added
- Collapsible "Resources to learn Make" section in the README.

## [1.0.0] - 2026-11-05

### Added
- Claude skill (`SKILL.md`) with patterns for orchestrating R analysis pipelines using GNU Make: file-target workflows and a standard `commandArgs` script contract.
- `README.md` with installation instructions, an explanation of GNU Make and a comparison with alternatives.
- `INDEX.md` and `QUICK-REFERENCE.md` companion documents, with YAML headers on all skill documents.
- Tidyverse style and design patterns (with attribution) and package choice guidelines (non-deprecated functions, `data.table` deferral).
- `CITATION.cff` and a "Citing this skill" section in the README.
- MIT licence and `.gitignore`.

### Changed
- Comment rule now asks for comments that explain why, not what.
- Argument-passing pattern corrected to match actual usage.
- `manifest.json` version aligned with the documented version (was `0.1.0`).
