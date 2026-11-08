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
| **SKILL.md** | ~8 KB | Complete reference: contract, patterns, examples, checklist, commons |
| **README.md** | ~7 KB | Overview, key concepts, when to use, how to use |
| **QUICK-REFERENCE.md** | ~9 KB | Copy-paste templates, commands, testing workflow, examples |
| **INDEX.md** | This file | Navigation guide |
| **manifest.json** | — | Skill metadata |

## Navigate by Task

### "I'm just starting with this skill"
1. Read **README.md** (5 min) for overview and key concepts
2. Then **QUICK-REFERENCE.md: File Organization** to see the directory layout
3. Look at **QUICK-REFERENCE.md: Example 3-Stage Pipeline** to see real code

### "I need to write a new pipeline script"
1. **QUICK-REFERENCE.md: Script Argument Contract** (copy the template)
2. Fill in your logic
3. Test interactively: `Rscript R/myscript.R`
4. Check **QUICK-REFERENCE.md: Testing a New Script** for next steps
5. Verify against **SKILL.md: Checklist for New Scripts**

### "I need to add a Make rule"
1. **QUICK-REFERENCE.md: Makefile Template** (copy the pattern)
2. Adapt for your targets
3. Test with `make --dry-run`
4. If complex, read **SKILL.md: Make Pattern Rules** for advanced patterns

### "I need to debug why Make isn't rebuilding"
1. **QUICK-REFERENCE.md: Common Commands** (force rebuild, dry-run)
2. **SKILL.md: Validation Workflow** (systematic testing)
3. Check: prerequisites listed in correct order? Dependencies spelled correctly?

### "I need to structure a reusable function"
1. **SKILL.md: Function Organization** (where it goes, naming)
2. **QUICK-REFERENCE.md: Roxygen2 Function Template** (documentation structure)
3. After editing, run `make docs` to regenerate `man/` and `NAMESPACE`

### "I'm refactoring an existing pipeline"
1. **SKILL.md: Project Structure** (see the layout)
2. **SKILL.md: Shared Configuration** (extract `R/shared_inputs.R`)
3. Add argument handling to scripts using **QUICK-REFERENCE.md: Script Argument Contract**
4. Replace phony Make targets with file targets using **SKILL.md: Makefile Pattern**
5. Verify with **SKILL.md: Checklist for New Scripts**

### "I'm teaching this to someone"
1. Give them **README.md** for overview
2. Walk through **QUICK-REFERENCE.md: Example 3-Stage Pipeline** with real code
3. Explain **QUICK-REFERENCE.md: Script Argument Contract**
4. Show them testing workflow: interactive → arguments → Make

### "I need style guidance"
- **SKILL.md: Style Guidelines** for Tidyverse packages, formatting, comments, error handling
- **QUICK-REFERENCE.md: Tidyverse Style Checklist** for verification
- **QUICK-REFERENCE.md: Humanizer Checklist** for writing quality

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

### README.md
- **When to Use:** Projects where this pattern fits
- **Three Core Rules:** Files, arguments, shared config
- **Key Concepts:** Shared configuration, pure functions, results in files
- **Pattern Rules:** Make % syntax
- **How to Use:** For new projects, refactoring, teaching
- **File Guide:** Which file to read for what
- **Common Tasks:** Quick routing to relevant sections
- **Validation Checklist:** Before committing

### QUICK-REFERENCE.md
- **Script Template:** Copy-paste argument handling
- **Makefile Template:** Copy-paste file target rules
- **File Organization:** Directory tree with annotations
- **Testing Workflow:** 4-step testing from interactive to Make
- **Common Commands:** Quarto, Make, debugging, cleanup
- **Shared Configuration:** How to structure study-wide settings
- **No Print Output:** Bad vs. good examples
- **Roxygen2 Template:** Function documentation structure
- **Tidyverse Checklist:** Package and syntax verification
- **Full Example:** 3-stage pipeline (Makefile + 3 R scripts)
- **Error Signalling:** cli_abort() and cli_warn() usage
- **Pattern Rules:** Advanced Make with %

## Decision Tree

```
What do I need to do?
│
├─ Start a new project
│  └─ README.md → QUICK-REFERENCE.md (File Org + Example)
│
├─ Write a new script
│  └─ QUICK-REFERENCE.md (Script Template) + SKILL.md (Checklist)
│
├─ Add a Make rule
│  └─ QUICK-REFERENCE.md (Makefile Template) + SKILL.md (Pattern Rules)
│
├─ Debug a build issue
│  └─ QUICK-REFERENCE.md (Common Commands) + SKILL.md (Validation)
│
├─ Structure a function
│  └─ SKILL.md (Function Organization) + QUICK-REFERENCE.md (Roxygen Template)
│
├─ Refactor code
│  └─ SKILL.md (Project Structure + Shared Config) + QUICK-REFERENCE.md (Script Template)
│
├─ Improve writing quality
│  └─ QUICK-REFERENCE.md (Humanizer Checklist) + SKILL.md (Style Guidelines)
│
└─ Teach someone
   └─ README.md (overview) + QUICK-REFERENCE.md (Example Pipeline)
```

## The Five Critical Sections

If you're in a hurry, read these five sections:

1. **SKILL.md: R Scripts and Makefile Conventions** — How scripts and Make interact
2. **QUICK-REFERENCE.md: Script Argument Contract** — Copy the template
3. **QUICK-REFERENCE.md: Makefile Template** — Copy the pattern
4. **QUICK-REFERENCE.md: Example 3-Stage Pipeline** — See it working
5. **SKILL.md: Checklist** — Verify your work

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
