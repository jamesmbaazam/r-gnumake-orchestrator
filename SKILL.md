# R + GNU Make Reproducible Research Patterns

A guide to orchestrating analysis pipelines in R using GNU Make 3.81 with file-target workflows and a standardized script interface contract.

## Overview

This skill captures generic patterns for reproducible data analysis:
- **Scripts as pure functions:** Accept arguments, read inputs, write outputs, exit
- **Makefiles with file targets:** Only rebuild when dependencies change
- **Shared configuration:** One source of truth across all scripts
- **Tidyverse ecosystem:** Consistent, modern R packages
- **roxygen2 documentation:** Every reusable function is documented
- **No output printing:** Results live in committed data files only

These patterns scale from small exploratory analyses to large research compendiums with dozens of stages and thousands of generated artifacts.

## Core Contract

### R Script Interface

Every pipeline script defines `.args` conditionally: interactive defaults if running in the console, `commandArgs()` if called from Make.

```r
# Define arguments: defaults for interactive, commandArgs when called from Make
.args <- if (interactive()) c(
  "data/default1.rds",
  "data/default2.rds",
  "R/shared_inputs.R",
  "output/default.csv"
) else commandArgs(trailingOnly = TRUE)

# Extract from .args by position
input1 <- .args[1]
input2 <- .args[2]
helper <- .args[3]
output <- .args[4]

source(helper)  # loads COUNTRY, SEED, OUTPUT_DIR, etc.

# Now: read inputs, compute, write output, exit
data <- read_rds(input1)
results <- process(data)
write_csv(results, output)
```

**In Make:** `Rscript $^ $@` passes all prerequisites in order, then the target.  
**Interactively:** `Rscript R/myscript.R` uses `.args` defaults (no arguments needed).

### Makefile Pattern

Use file targets (not phony rules). Only rebuild when prerequisites change.

```makefile
# Directory creation (order-only, doesn't trigger rebuilds)
output output/figures output/tables:
	mkdir -p $@

# Single input → single output
output/processed.rds: data/raw.csv R/process.R R/shared_inputs.R | output
	Rscript $^ $@

# Multiple inputs → single output
output/scores.csv: output/model.rds data/test.rds R/score.R R/shared_inputs.R | output
	Rscript $^ $@

# Phony targets for convenience only (not tracked by Make)
.PHONY: all
all: output/scores.csv
```

**Key syntax:**
- `$^` = all prerequisites in order
- `$@` = target filename
- `| output` = order-only prerequisite (directory created, but timestamp ignored)

### Shared Configuration

One file holds every value two or more scripts need. No duplication across scripts.

```r
# R/shared_inputs.R
COUNTRY <- "Afghanistan"
HORIZON <- as.integer(Sys.getenv("HORIZON", "1"))
SEED <- 42
OUTPUT_DIR <- file.path("output", paste0("horizon_", HORIZON))
VALIDATION_YEARS <- 2017:2023

# Every script sources this, never redefines it
source("R/shared_inputs.R")
```

## File Organization

```
project/
├── Makefile                 # Orchestration rules (file targets)
├── R/
│   ├── shared_inputs.R      # Configuration (sourced by all scripts)
│   ├── process_data.R       # Pipeline scripts
│   ├── fit_model.R          # (one job per script)
│   ├── score_model.R
│   ├── helpers.R            # Reusable functions (no stage prefix)
│   ├── models.R             # roxygen2-documented
│   └── utils.R
├── data/
│   ├── raw/                 # Source data
│   ├── processed/           # Generated (gitignored or DVC-tracked)
│   └── features/
├── output/
│   ├── figures/             # Committed (so manuscript renders in CI)
│   ├── tables/              # .csv files with all numbers
│   ├── models/              # Fitted model objects (.rds)
│   └── predictions/         # .csv with model outputs
└── paper/
    ├── paper.qmd            # Manuscript (reads output/*.csv)
    └── bibliography.bib
```

**Stage prefixes sort in execution order:**
- `process_*.R` → `fit_*.R` → `score_*.R` → `visualise_*.R`

**Function files have no prefix** so documentation tools can distinguish them from pipeline scripts.

## Make Pattern Rules

Use pattern rules (%) for repetitive targets. Define once, apply to many.

```makefile
# Pattern: process each input file
INPUTS := file1 file2 file3

# Generic rule: transform any %.csv to output/processed_%.rds
output/processed_%.rds: input/%.csv R/process.R R/shared_inputs.R | output
	Rscript $^ $@

# Convenience target: all processed files
all_processed: $(patsubst input/%.csv, output/processed_%.rds, $(wildcard input/*.csv))

# Override for special case: file1 needs different input
output/processed_file1.rds: input/file1_v2.csv R/process.R R/shared_inputs.R | output
	Rscript $^ $@
```

**Advanced: multi-output rules**

If a single script produces multiple outputs, list them both as targets. Make treats both as up-to-date after one execution.

```makefile
output/model.rds output/metadata.json: R/fit.R data/train.rds R/shared_inputs.R | output
	Rscript $^ output/model.rds output/metadata.json
```

## Function Organization

### Pipeline Scripts

One job per script. Each stage reads what the previous stage wrote.

```r
# R/fit_model.R
# Purpose: Fit a model to prepared data
# Inputs: data/prepared.rds, R/shared_inputs.R
# Outputs: output/model.rds, output/model_meta.csv

args <- commandArgs(trailingOnly = TRUE)
data_file <- if (length(args) >= 1) args[1] else "data/prepared.rds"
helper <- if (length(args) >= 2) args[2] else "R/shared_inputs.R"
model_file <- if (length(args) >= 3) args[3] else "output/model.rds"
meta_file <- if (length(args) >= 4) args[4] else "output/model_meta.csv"

source(helper)

data <- read_rds(data_file)
model <- fit_model(data, seed = SEED)

write_rds(model, model_file)
write_csv(
  tibble(
    n_training = nrow(data),
    n_features = ncol(data) - 1,
    model_type = "random_forest"
  ),
  meta_file
)
```

### Function Files

Reusable functions go in separate files (no stage prefix). Document with roxygen2.

```r
# R/models.R

#' Fit a model to data
#'
#' Trains a random forest using specified parameters.
#'
#' @details
#' Assumptions about inputs:
#' - data has column `outcome` (binary: 0/1) and all predictors are numeric
#' - seed is an integer for reproducibility
#'
#' @param data A tibble with outcome and predictor columns.
#' @param seed Integer for random number generation.
#' @param mtry Integer; number of variables to sample per split.
#'
#' @return A fitted model object (ranger class).
#'
#' @export
fit_model <- function(data, seed = 42, mtry = floor(sqrt(ncol(data) - 1))) {
  model <- ranger(outcome ~ ., data = data, seed = seed, mtry = mtry)
  model
}
```

Run `make docs` to regenerate `man/` and `NAMESPACE` from roxygen blocks.

### Shared Configuration

Central location for study-wide parameters. Every script sources it.

```r
# R/shared_inputs.R

# Study design
COUNTRY <- "Country_Name"
SEED <- 42
ANALYSIS_YEARS <- 2017:2023

# Paths (computed, not hardcoded)
DATA_DIR <- "data"
OUTPUT_DIR <- "output"
HORIZON <- as.integer(Sys.getenv("HORIZON", "1"))

# Model hyperparameters
RF_MTRY <- NULL  # sqrt(p)
RF_NTREES <- 500

# Do not define anything else here.
# Stage-specific values stay in their stage's script.
```

## Style Guidelines

### Tidyverse Stack

Standardize on these packages:

```r
library(tidyverse)   # core: dplyr, tidyr, ggplot2, readr, purrr
library(fs)          # filesystem operations (dir_create, file_exists)
library(cli)         # error signalling (cli_abort, cli_warn)
library(here)        # relative paths (here("R", "script.R"))
library(scales)      # plot formatting
library(yardstick)   # model metrics
```

**Never use:**
- Base subsetting or `aggregate()` (use `dplyr::filter()`, `dplyr::summarise()`)
- `map_dfr()` (use `map()` + `list_rbind()`)
- `group_by()` + `ungroup()` (use `.by =` argument)
- `gather()`, `spread()` (use `pivot_longer()`, `pivot_wider()`)
- `%>%` (use native pipe `|>`)
- `function(x)` (use lambda `\(x)`)
- Superseded functions (check lifecycle badge on help page)

### Code Formatting

Use air formatter (enforces 80-character lines, 2-space indent):

```bash
air format R/                  # format entire directory
air format R/myscript.R        # format one file
```

Settings in `air.toml` at repo root (tracked, not personal).

### Comments

Comment **what** the code does, not **why** (rationale goes in Methods section of paper).

```r
# Bad
# Use a random forest because it's robust to non-linear relationships
model <- ranger(outcome ~ ., data = data)

# Good
# Fit a random forest with default hyperparameters
model <- ranger(outcome ~ ., data = data)
```

### No Output Printing

Results live only in committed CSV, RDS, and PNG files.
No `message()`, `print()`, `cat()`, or `sprintf()` for informational output.
Exceptions: `cli_abort()` and `cli_warn()` (control flow only).

```r
# Bad
message(sprintf("Processed %d rows", nrow(data)))
cat("Model trained\n")

# Good (write to file instead)
meta <- tibble(n_rows = nrow(data))
write_csv(meta, "output/metadata.csv")
```

The manuscript reads `output/metadata.csv` at render time, so every number appears in committed files.

## Validation Workflow

When writing a new script, test progressively before adding Make rules.

### Step 1: Test Interactively (with defaults)

```bash
# Does it work with no arguments?
Rscript R/myscript.R
# Check output files were created
ls -lh output/
```

### Step 2: Test with Arguments (mimics Make)

```bash
# Pass arguments as Make would
Rscript R/myscript.R \
  data/raw.csv \
  R/shared_inputs.R \
  output/result.rds
# Compare with Step 1 output
```

### Step 3: Add Make Rule and Test

```bash
# In Makefile:
output/result.rds: data/raw.csv R/myscript.R R/shared_inputs.R | output
	Rscript $^ $@

# Test it
make output/result.rds

# Verify: modifying data should trigger rebuild
touch data/raw.csv
make output/result.rds  # Should re-run

# Verify: modifying output/result.rds (not dependency) doesn't trigger rebuild
touch output/result.rds
make output/result.rds  # Should say "up to date"
```

### Step 4: Dry Run

```bash
make --dry-run output/result.rds  # Show what would run without running
```

## Example: Simple 3-Stage Pipeline

### Makefile

```makefile
.PHONY: all
all: output/summary.csv

output:
	mkdir -p $@

output/processed.rds: data/raw.csv R/process.R R/shared_inputs.R | output
	Rscript $^ $@

output/model.rds: output/processed.rds R/fit.R R/shared_inputs.R | output
	Rscript $^ $@

output/summary.csv: output/model.rds output/processed.rds R/score.R R/shared_inputs.R | output
	Rscript $^ $@
```

### R/process.R

```r
.args <- if (interactive()) c(
  "data/raw.csv",
  "R/shared_inputs.R",
  "output/processed.rds"
) else commandArgs(trailingOnly = TRUE)

raw_file <- .args[1]
helper <- .args[2]
output <- .args[3]

source(helper)

data <- read_csv(raw_file) |>
  filter(year %in% ANALYSIS_YEARS) |>
  mutate(across(everything(), as.numeric))

write_rds(data, output)
```

### R/fit.R

```r
.args <- if (interactive()) c(
  "output/processed.rds",
  "R/shared_inputs.R",
  "output/model.rds"
) else commandArgs(trailingOnly = TRUE)

data_file <- .args[1]
helper <- .args[2]
output <- .args[3]

source(helper)

data <- read_rds(data_file)
model <- ranger(outcome ~ ., data = data, seed = SEED)
write_rds(model, output)
```

### R/score.R

```r
.args <- if (interactive()) c(
  "output/model.rds",
  "output/processed.rds",
  "R/shared_inputs.R",
  "output/summary.csv"
) else commandArgs(trailingOnly = TRUE)

model_file <- .args[1]
data_file <- .args[2]
helper <- .args[3]
output <- .args[4]

source(helper)

model <- read_rds(model_file)
data <- read_rds(data_file)
predictions <- predict(model, data)$predictions

scores <- tibble(
  auc = AUC::auc(data$outcome, predictions),
  n = nrow(data)
)

write_csv(scores, output)
```

### Run It

```bash
make            # Builds all in order (process → fit → score)
make output/model.rds --always-make  # Force rebuild just this step
make --dry-run  # Show what would run
```

## Checklist for New Scripts

When writing a new pipeline stage:

- [ ] Script accepts arguments in order: inputs → helper → output
- [ ] Script includes `.args` defaults for interactive testing
- [ ] Script sources `R/shared_inputs.R` (never defines study-wide values)
- [ ] Script reads all inputs, computes, writes all outputs
- [ ] No `print()`, `message()`, or `cat()` (use `cli_abort()`, `cli_warn()` for errors only)
- [ ] All results written to committed CSV/RDS/PNG files
- [ ] Dependencies listed correctly in Makefile (in order)
- [ ] Order-only directory prerequisites use `|` syntax
- [ ] Tested interactively, then with arguments, then via Make
- [ ] Makefile rule uses `$^` and `$@` syntax
- [ ] Code formatted with `air format R/scriptname.R`
- [ ] Reusable functions extracted to separate files and documented with roxygen2

## Common Patterns

### Batch Processing Multiple Inputs

```makefile
PROVINCES := province1 province2 province3

# Pattern: process each province
output/summary_%.csv: data/raw_%.csv R/process.R R/shared_inputs.R | output
	Rscript $^ $@

# Target: all provinces
all_summaries: $(patsubst %, output/summary_%.csv, ${PROVINCES})
```

### Generating All Combinations

```makefile
MODELS := rf xgb catboost
HORIZONS := 1 3 6 12

# Generate one target per combination
TARGETS := $(foreach m, ${MODELS}, $(foreach h, ${HORIZONS}, output/${m}_horizon_${h}.rds))

all_models: ${TARGETS}

output/%.rds: data/prepared.rds R/fit.R R/shared_inputs.R | output
	Rscript $^ $@
```

### Multi-Output Rules

```makefile
# One script produces multiple outputs
output/model.rds output/importance.csv: data/train.rds R/fit.R R/shared_inputs.R | output
	Rscript $< R/shared_inputs.R output/model.rds output/importance.csv
```

### Aggregating Results

```makefile
# Combine per-province results into summary
output/summary_all.csv: $(patsubst %, output/summary_%.csv, ${PROVINCES}) R/aggregate.R | output
	Rscript $^ $@
```

## References

- **Tidyverse style guide:** https://style.tidyverse.org/
- **GNU Make manual:** https://www.gnu.org/software/make/manual/
- **roxygen2:** https://roxygen2.r-lib.org/
- **air formatter:** https://posit-dev.github.io/air/

---

**Skill version:** 1.0.0  
**Generic patterns extracted from:** Research project reference implementations  
**Last updated:** 2 November 2026
