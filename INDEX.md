---
title: Navigation Guide - R + GNU Make Skill
description: Task-based routing and decision tree to find information
keywords:
  - navigation
  - task routing
  - how-to
---

# R + GNU Make Reproducible Research Skill — Navigation

## Three Core Rules

1. **Files are targets:** Every .csv, .rds, .png is a Make target. Make only rebuilds when inputs change.
2. **Scripts accept arguments:** Order: input files → helper script → output file
3. **One source of truth:** `R/shared_inputs.R` holds every value 2+ scripts need

## Files in This Skill

| File | Size | Purpose |
|------|------|---------|
| **SKILL.md** | — | Complete reference: conventions, patterns, examples, checklist |
| **README.md** | — | Overview, installation, what the skill covers, learning resources |
| **INDEX.md** | This file | Navigation guide |
| **manifest.json** | — | Skill metadata |

## Navigate by Task

### "I'm just starting with this skill"
1. Read **README.md** (5 min) for overview and key concepts
2. Then **SKILL.md: File Organization** to see the directory layout
3. Look at **SKILL.md: Example: Simple 3-Stage Pipeline** to see real code

### "I need to write a new pipeline script"
1. **SKILL.md: R Script Pattern** (copy the template)
2. Fill in your logic
3. Test interactively: `Rscript R/myscript.R`
4. Check **SKILL.md: Validation Workflow** for next steps
5. Verify against **SKILL.md: Checklist for New Scripts**

### "I need to add a Make rule"
1. **SKILL.md: Makefile Pattern** (copy the pattern)
2. Adapt for your targets
3. Test with `make --dry-run`
4. If complex, read **SKILL.md: Make Pattern Rules** for advanced patterns

### "I need to debug why Make isn't rebuilding"
1. **SKILL.md: Validation Workflow** (systematic testing, dry-run)
2. Force a rebuild with `make <target> --always-make`
3. Check: prerequisites listed in correct order? Dependencies spelled correctly?

### "I need to structure a reusable function"
1. **SKILL.md: Function Organization** (where it goes, naming)
2. **SKILL.md: Function Files** (roxygen2 documentation structure)
3. After editing, run `make docs` to regenerate `man/` and `NAMESPACE`

### "I'm refactoring an existing pipeline"
1. **SKILL.md: Project Structure** (see the layout)
2. **SKILL.md: Shared Configuration** (extract `R/shared_inputs.R`)
3. Add argument handling to scripts using **SKILL.md: R Script Pattern**
4. Replace phony Make targets with file targets using **SKILL.md: Makefile Pattern**
5. Verify with **SKILL.md: Checklist for New Scripts**

### "I'm teaching this to someone"
1. Give them **README.md** for overview
2. Walk through **SKILL.md: Example: Simple 3-Stage Pipeline** with real code
3. Explain **SKILL.md: R Script Pattern**
4. Show them testing workflow: interactive → arguments → Make

### "I need style guidance"
- **SKILL.md: Style Guidelines** for Tidyverse packages, formatting, comments, error handling
- **SKILL.md: Tidyverse Style & Design Principles** for naming, function design and package choices

## Key Concepts by Section

### SKILL.md
- **Overview:** Why these patterns exist
- **R Scripts and Makefile Conventions:** Exact script argument order and Makefile syntax
- **File Organization:** Directory structure, naming conventions
- **Make Pattern Rules:** Pattern rules (%), special targets, advanced examples
- **Function Organization:** Pipeline scripts vs. function libraries vs. configuration
- **Style Guidelines:** Tidyverse stack, formatting, comments, no output printing
- **Validation Workflow:** Progressive testing (interactive → args → Make → dry-run)
- **Example:** Complete 3-stage pipeline with Makefile and all R scripts
- **Checklist:** Verification before committing
- **Common Patterns:** Batch processing, multi-output rules, aggregation

## Decision Tree

```
What do I need to do?
│
├─ Start a new project
│  └─ README.md → SKILL.md (File Organization + Example)
│
├─ Write a new script
│  └─ SKILL.md (R Script Pattern + Checklist)
│
├─ Add a Make rule
│  └─ SKILL.md (Makefile Pattern + Make Pattern Rules)
│
├─ Debug a build issue
│  └─ SKILL.md (Validation Workflow)
│
├─ Structure a function
│  └─ SKILL.md (Function Organization)
│
├─ Refactor code
│  └─ SKILL.md (File Organization + Shared Configuration + R Script Pattern)
│
├─ Improve code style
│  └─ SKILL.md (Style Guidelines + Tidyverse Style & Design Principles)
│
└─ Teach someone
   └─ README.md (overview) + SKILL.md (Example)
```

## The Five Critical Sections

If you're in a hurry, read these five sections:

1. **SKILL.md: R Scripts and Makefile Conventions** — How scripts and Make interact
2. **SKILL.md: R Script Pattern** — Copy the template
3. **SKILL.md: Makefile Pattern** — Copy the pattern
4. **SKILL.md: Example: Simple 3-Stage Pipeline** — See it working
5. **SKILL.md: Checklist for New Scripts** — Verify your work

These five sections contain everything needed to start a project. Everything else is reference material.

## Tools and Commands

```bash
# Format code
air format R/myscript.R

# Generate function documentation
make docs

# Test script interactively
Rscript R/myscript.R

# Test with arguments
Rscript R/myscript.R input1.rds input2.rds R/shared_inputs.R output.csv

# Test via Make
make output/result.csv

# Show what would run (dry-run)
make --dry-run output/result.csv

# Force rebuild
make output/result.csv --always-make

# Render manuscript
quarto render paper/paper.qmd
```

## Cross-References

**Tidyverse style:** https://style.tidyverse.org/  
**GNU Make manual:** https://www.gnu.org/software/make/manual/  
**roxygen2:** https://roxygen2.r-lib.org/  
**air formatter:** https://posit-dev.github.io/air/

---

**Skill version:** 1.0.3  
**Generic patterns extracted from:** Research project implementations  
**Last updated:** 8 November 2026  
**Status:** Ready for use
