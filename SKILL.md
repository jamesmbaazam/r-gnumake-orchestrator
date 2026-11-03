---
title: R + GNU Make Reproducible Research Patterns
description: Generic patterns for orchestrating data analysis pipelines in R using GNU Make with file-target workflows and standardized script contracts
keywords:
  - R
  - GNU Make
  - reproducible research
  - pipeline orchestration
  - workflow automation
  - Tidyverse
  - data analysis
---

# R + GNU Make Reproducible Research Patterns

**This is a Claude skill.** Load with `/skill r-gnumake-orchestrator` in Claude to access patterns for building reproducible data analysis pipelines.

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

Comment **why** the code does something, not **what** (only where the rationale isn't obvious from reading the code). Avoid comments that just restate the code.

```r
# Bad: repeats what the code obviously does
# Fit a random forest with default hyperparameters
model <- ranger(outcome ~ ., data = data)

# Good: explains why we made this choice
# Random forest over CART here because we need confidence intervals from quantile regression
model <- ranger(outcome ~ ., data = data)

# Bad: obvious from the code
# Filter rows where year is in ANALYSIS_YEARS
data <- data |> filter(year %in% ANALYSIS_YEARS)

# Good: explains the non-obvious reasoning
# Exclude year 2015 due to incomplete surveillance coverage (see Methods)
data <- data |> filter(year >= 2016)

# No comment needed: the code is self-explanatory
data <- data |>
  select(district, year, outcome) |>
  filter(!is.na(outcome))
```

Write comments in the manuscript's Methods section when explaining a methodological choice; use code comments only when the decision is unexpected or non-standard.

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

## Tidyverse Style & Design Principles

This skill builds on the [Tidyverse Style Guide](https://style.tidyverse.org/) and [Tidyverse Design Principles](https://github.com/tidyverse/design/) for consistency with the modern R ecosystem. Key patterns extracted with attribution:

### Naming Conventions
*Adapted from [Tidyverse Style: Object Names](https://style.tidyverse.org/syntax.html#sec-objectnames)*

- **Use snake_case:** `day_one`, not `dayOne` or `day.one`
- **Verbs for functions:** `add_row()`, not `row_adder()`
- **Nouns for variables:** `model_data`, not `process_data_for_model`
- **Avoid re-using common function names:** Don't shadow `T`, `c`, `mean`, etc.

```r
# Good: verb describes action
fit_model <- function(data, formula) { ... }

# Good: noun describes content
training_data <- data |> filter(year < 2020)

# Bad: noun where verb expected
row_adder <- function(x, y) x + y

# Bad: shadows base function
mean <- function(x) sum(x)
```

### Function Design
*Adapted from [Tidyverse Design: Defaults — Short and Sweet](https://design.tidyverse.org/) and [Style: Functions](https://style.tidyverse.org/functions.html)*

- **Separate data from control arguments:** Data goes first (no name needed), controls go last (always named)
- **Prefer early returns:** Exit early with explicit `return()` for error cases; let last expression be the success value
- **Return first argument invisibly for side-effects:** Enables piping for functions that print, plot, or save

```r
# Good: data first, control args last
score_model <- function(data, model, metrics = c("auc", "pr")) {
  # data and model are positional, metrics is named control
}

# Good: early return for special cases
validate_data <- function(data) {
  if (nrow(data) == 0) return(NULL)
  if (anyNA(data)) return(NULL)
  data  # success value returned implicitly
}

# Good: invisible return for side-effects (saves to disk, returns object)
save_results <- function(results, path) {
  write_csv(results, path)
  invisible(results)  # Allows: results |> save_results(path) |> next_stage()
}
```

### Spacing & Syntax
*Adapted from [Tidyverse Style: Syntax](https://style.tidyverse.org/syntax.html)*

- **Spaces around infix operators:** `x <- a + b`, not `x<-a+b`
- **No spaces inside function calls:** `mean(x)`, not `mean( x )`
- **Space after comma:** `x[1, 2]`, not `x[1,2]`
- **Lambda syntax (modern):** `\(x) x + 1` instead of `function(x) x + 1` (single-line only)
- **Native pipe:** `|>` instead of `%>%`

```r
# Good
results <- data |>
  filter(year > 2015) |>
  mutate(across(everything(), as.numeric))

# Good: modern lambda for single-line functions
mapping <- list_rbind(map(datasets, \(x) fit_model(x)))

# Bad: multi-line lambda (use function() instead)
mapping <- map(datasets, \(x) {
  fit_model(x)
})

# Bad: old pipe and function syntax
results <- data %>%
  filter(year > 2015) %>%
  mutate_all(as.numeric)
```

### Code Organization
*Adapted from [Tidyverse Design: Organize Code](https://github.com/tidyverse/design/)*

- **One thought per paragraph:** Separate logical blocks with blank lines (sparingly)
- **Avoid empty lines at function start/end**
- **Group related operations together:** Data transformation, then computation, then output

```r
# Good: clear structure
fit_tree <- function(data, outcome_col) {
  # Prepare data
  data_clean <- data |>
    filter(!is.na({{ outcome_col }})) |>
    select(where(is.numeric))

  # Fit model
  model <- rpart(outcome_col ~ ., data = data_clean)

  # Return
  model
}

# Bad: no logical separation
fit_tree <- function(data, outcome_col) {
  data_clean <- data |> filter(!is.na({{ outcome_col }}))
  data_clean <- data_clean |> select(where(is.numeric))
  model <- rpart(outcome_col ~ ., data = data_clean)
  return(model)
}
```

### Package Choices

**Use only current (non-deprecated) functions.** When you find a deprecated function in the codebase, replace it with its successor—check the lifecycle badge on the help page.

Common replacements:
- `mutate_at()` / `_if()` / `_all()` → `across()`
- `gather()` / `spread()` → `pivot_longer()` / `pivot_wider()`
- `sample_n()` / `sample_frac()` → `slice_sample()`
- `top_n()` → `slice_max()` / `slice_min()`
- `map_dfr()` / `map_dfc()` → `map()` then `list_rbind()` / `list_cbind()`

**For heavy data operations, consider dplyr first; defer to data.table only when performance is critical.**

Use dplyr (tidyverse) for:
- Most data manipulation (it's clear, familiar, and fast enough)
- Exploratory work
- Pipeline scripts where clarity matters more than microseconds

Use [data.table](https://cran.r-project.org/web/packages/data.table/vignettes/) for:
- Joining or grouping multi-GB datasets
- Real-time streaming or repeated operations
- Explicit performance requirement (document in a comment)

```r
# dplyr: clear, idiomatic, fast enough for most pipelines
data |>
  filter(year >= 2015) |>
  group_by(district) |>
  summarise(total = sum(outcome), .groups = "drop")

# data.table: when you need sub-second performance on large data
# Benchmarked: dplyr took 8s, data.table takes 0.3s on 500M rows
dt <- as.data.table(data)
dt[year >= 2015, .(total = sum(outcome)), by = district]
```

### Documentation
*Adapted from [Tidyverse Style: Documentation](https://style.tidyverse.org/documentation.html)*

- **roxygen2 blocks:** `#' @param`, `#' @return`, `#' @details` (markdown format)
- **Describe what function does, then how it works:** Lead with intent, follow with mechanism
- **Document parameters that aren't obvious:** What class? What values expected?

```r
#' Fit a tree model to training data
#'
#' Splits the outcome variable into train/test using date-based rolling origins,
#' then fits a regression tree to each training window.
#'
#' @param data A tibble with columns: `year`, `month`, `outcome`, predictors...
#' @param outcome_col Quoted column name of numeric outcome
#' @param min_year,max_year Calendar years defining the analysis period
#'
#' @return A tibble with columns: `origin_year`, `model`, `n_train`
#'
#' @details
#' Rolling origins expand forward from min_year, refit annually, and predict
#' one-step ahead. This harness matches the operational pipeline in Voorman et al. (2023).
#'
#' @export
fit_tree <- function(data, outcome_col, min_year, max_year) {
  # implementation
}
```

## References

- **Tidyverse style guide:** https://style.tidyverse.org/
- **Tidyverse design principles:** https://github.com/tidyverse/design/
- **GNU Make manual:** https://www.gnu.org/software/make/manual/
- **roxygen2:** https://roxygen2.r-lib.org/
- **air formatter:** https://posit-dev.github.io/air/

---

**Skill version:** 1.0.0  
**Generic patterns extracted from:** Research project reference implementations  
**Last updated:** 2 November 2026
