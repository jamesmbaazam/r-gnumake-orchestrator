# Changelog

All notable changes to this project are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and the project uses [Semantic Versioning](https://semver.org/).

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
