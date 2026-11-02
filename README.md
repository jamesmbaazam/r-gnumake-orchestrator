# R + GNU Make Reproducible Research Skill

Generic patterns for reproducible analysis pipelines in R orchestrated by GNU Make 3.81.

## Why This Skill Exists

Research projects often choose between two extremes:
- **Interactive notebooks:** Exploratory, readable, but impossible to re-run or audit months later
- **Production software:** Robust and reproducible, but overengineered for research-scale work

This skill documents patterns that work at research scale: small, understandable scripts that respect data dependencies and orchestration by Make, using tools widely available (GNU Make 3.81, R, standard Unix).

## When to Use This

✓ **Good fit:**
- Data analysis pipelines with 5+ stages
- Projects where reproducibility matters (research, regulatory reporting)
- Multi-person teams that need to audit workflows
- Datasets too large for Jupyter notebooks
- Projects with many interdependent files (figures, tables, models)

✗ **Not a fit:**
- Throwaway exploratory scripts (use a notebook)
- Software packages (use devtools/usethis instead)
- Interactive dashboards (use Shiny or RMarkdown)
- Single-script analyses

## Three Core Rules

### 1. Files Are Targets

Every .csv, .rds, .png is a Make target. Make rebuilds only when inputs change—not every time you run `make`.

```makefile
# File targets (not phony): only rebuild if inputs changed
output/results.csv: input/data.rds R/script.R | output
	Rscript $^ $@

# Without Make: you must manually track which files to re-run
```

### 2. Scripts Accept Arguments

Scripts take arguments in fixed order: input files → helper script → output file.
Every script includes defaults so you can test locally without Make.

```r
args <- commandArgs(trailingOnly = TRUE)
input <- if (length(args) >= 1) args[1] else "data/default.rds"
helper <- if (length(args) >= 2) args[2] else "R/shared_inputs.R"
output <- if (length(args) >= 3) args[3] else "output/default.csv"
source(helper)
# ... compute and write output
```

### 3. One Source of Truth

`R/shared_inputs.R` holds every value two or more scripts need. No duplication.

```r
# Every script does this
source("R/shared_inputs.R")  # defines COUNTRY, SEED, OUTPUT_DIR, etc.
```

## Key Concepts

### Shared Configuration

One file holds study-wide settings. Every script sources it. Never redefines it.

```r
# R/shared_inputs.R
COUNTRY <- "Example"
SEED <- 42
ANALYSIS_YEARS <- 2017:2023
```

Every other script:
```r
source("R/shared_inputs.R")
# Now you have COUNTRY, SEED, ANALYSIS_YEARS available
```

### Pure Function Scripts

Scripts are stateless: read input files, compute, write output files, exit.
No global state. No implicit defaults.

```r
# Good: inputs and output are explicit
args <- commandArgs(trailingOnly = TRUE)
data <- read_rds(args[1])
results <- analyze(data)
write_csv(results, args[3])

# Bad: implicit filenames in the script
data <- read_rds("data/mydata.rds")  # hardcoded, not flexible
write_csv(results, "output/results.csv")  # hardcoded
```

### Results, Not Narration

The manuscript reads committed CSV and PNG files. No number is typed by hand.
No `print()`, `message()`, or `cat()` for informational output.
Only `cli_abort()` and `cli_warn()` for error flow.

```r
# Bad
message(sprintf("Processed %d rows", nrow(data)))

# Good
meta <- tibble(n_rows = nrow(data))
write_csv(meta, "output/metadata.csv")
# Paper reads output/metadata.csv at render time
```

### Pattern Rules

Use Make's `%` pattern to apply one rule to many targets.

```makefile
# Process any *.csv to output/processed_*.rds
output/processed_%.rds: input/%.csv R/process.R R/shared_inputs.R | output
	Rscript $^ $@

# Convenience: all processed files
all_processed: $(patsubst input/%.csv, output/processed_%.rds, $(wildcard input/*.csv))
```

## How to Use

### Starting a New Project

1. Create directory structure:
   ```bash
   mkdir -p project/{R,data,output,paper}
   cd project
   ```

2. Copy Makefile template from QUICK-REFERENCE.md

3. Create `R/shared_inputs.R` with study-wide parameters

4. Create first pipeline script following the contract (see SKILL.md)

5. Test interactively, then add Make rule

### Refactoring Existing Code

1. Identify shared values used by 2+ scripts
2. Move to `R/shared_inputs.R`
3. Add argument handling to scripts
4. Replace phony Make targets with file targets
5. Test with `make --dry-run`

### Teaching Others

- Show QUICK-REFERENCE.md: File Organization
- Walk through a 3-stage example (QUICK-REFERENCE.md: Example Pipeline)
- Explain the argument contract (SKILL.md: R Script Interface)

## File Guide

| File | Read When | Purpose |
|------|-----------|---------|
| **SKILL.md** | First time, or deep dives | Complete reference: contract, patterns, examples, checklist |
| **README.md** | Quick orientation | This file; overview and key concepts |
| **QUICK-REFERENCE.md** | Working on a script | Copy-paste templates, common commands, testing workflow |
| **INDEX.md** | Finding something specific | Navigation guide with task-based routing |

## Common Tasks

### "I need to write a new script"
1. Read QUICK-REFERENCE.md: Script Argument Contract
2. Copy the template
3. Test interactively
4. Add Make rule
5. Check against SKILL.md: Checklist

### "I need to add a Make rule"
1. Copy template from QUICK-REFERENCE.md: Makefile Template
2. Understand patterns in SKILL.md: Make Pattern Rules
3. Test with `make --dry-run`

### "I need to debug something"
1. QUICK-REFERENCE.md: Common Commands (force rebuild, dry-run)
2. SKILL.md: Validation Workflow (test progressively)

### "I need to structure a function"
1. SKILL.md: Function Organization
2. QUICK-REFERENCE.md: Roxygen2 Function Template
3. Run `make docs` to regenerate documentation

## Validation Checklist

Before committing a new pipeline stage:

- [ ] Script accepts arguments: inputs → helper → output
- [ ] Script includes `.args` defaults
- [ ] Script sources `R/shared_inputs.R`
- [ ] No `print()`, `message()`, or `cat()` (only `cli_abort()`, `cli_warn()`)
- [ ] All results in committed CSV/RDS/PNG
- [ ] Makefile uses `$^` and `$@`
- [ ] Order-only directories use `|`
- [ ] Tested interactively → with args → via Make
- [ ] Code formatted with `air format R/`
- [ ] Reusable functions in separate files with roxygen2

## Tools Used

- **GNU Make 3.81:** File-target orchestration (available everywhere)
- **R:** Analysis (pure functions, roxygen2 docs)
- **tidyverse:** Modern data manipulation (dplyr, tidyr, ggplot2, readr, purrr)
- **air:** Code formatter (80-char lines, 2-space indent)
- **cli:** Error signalling (`cli_abort()`, `cli_warn()`)

## References

- Tidyverse style guide: https://style.tidyverse.org/
- GNU Make manual: https://www.gnu.org/software/make/manual/
- roxygen2: https://roxygen2.r-lib.org/
- air formatter: https://posit-dev.github.io/air/

---

**Skill version:** 1.0.0  
**Last updated:** 2 November 2026  
**Generic patterns from:** Research project reference implementations
