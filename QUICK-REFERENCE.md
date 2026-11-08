---
title: Quick Reference - R + GNU Make
description: Copy-paste templates, examples, testing workflow, and common commands
keywords:
  - R
  - GNU Make
  - templates
  - examples
  - quick start
---

# Quick Reference: R + GNU Make Reproducible Research

## Script Argument Contract (Copy-Paste Template)

Every pipeline script defines `.args` conditionally: defaults when interactive, `commandArgs()` when called from Make.

```r
# Define arguments: interactive defaults or commandArgs from Make
.args <- if (interactive()) c(
  "data/default1.rds",
  "data/default2.rds",
  "R/shared_inputs.R",
  "output/default.csv"
) else commandArgs(trailingOnly = TRUE)

# Extract by position
input1 <- .args[1]
input2 <- .args[2]
helper <- .args[3]
output <- .args[4]

source(helper)

# Rest of script: read, compute, write
data1 <- read_rds(input1)
data2 <- read_rds(input2)
results <- your_function(data1, data2)
write_csv(results, output)
```

**Make calls it:** `Rscript $^ $@` (passes all prerequisites, then target)  
**Locally:** `Rscript R/myscript.R` (uses defaults, no arguments needed)

---

## Makefile Template (Copy-Paste)

```makefile
# Phony: aggregators only
.PHONY: all clean

# Directory prerequisites (order-only)
output output/figures output/tables:
	mkdir -p $@

# File target: input → output
output/processed.rds: data/raw.csv R/process.R R/shared_inputs.R | output
	Rscript $^ $@

# Multi-input target
output/scores.csv: output/model.rds data/test.rds R/score.R R/shared_inputs.R | output
	Rscript $^ $@

# Phony aggregator (convenience)
all: output/scores.csv

# Cleanup
clean:
	rm -rf output/
```

**Key syntax:**
- `$^` = all prerequisites in order
- `$@` = target
- `| output` = order-only (directory created, timestamp ignored)

---

## File Organization

```
project/
├── Makefile                 # Orchestration
├── R/
│   ├── shared_inputs.R      # Configuration (sourced by all)
│   ├── process_data.R       # Pipeline scripts (one job each)
│   ├── fit_model.R
│   ├── score_model.R
│   ├── visualize.R
│   ├── helpers.R            # Reusable functions (no stage prefix)
│   ├── models.R             # roxygen2-documented
│   └── utils.R
├── data/
│   ├── raw/                 # Source data
│   └── processed/           # Generated (gitignored)
├── output/
│   ├── figures/             # .png (committed)
│   ├── tables/              # .csv (committed)
│   └── models/              # .rds (gitignored or tracked)
└── paper/
    ├── paper.qmd
    └── bibliography.bib
```

**Stage prefix order:** `process_*.R` → `fit_*.R` → `score_*.R` → `visualise_*.R`

---

## Testing a New Script

### Step 1: Test Interactively (defaults)
```bash
Rscript R/myscript.R
ls output/
```

### Step 2: Test with Arguments (mimics Make)
```bash
Rscript R/myscript.R input1.rds input2.rds R/shared_inputs.R output.csv
```

### Step 3: Add Make Rule and Test
```makefile
output/result.csv: input1.rds input2.rds R/myscript.R R/shared_inputs.R | output
	Rscript $^ $@
```

```bash
make output/result.csv           # First run
make output/result.csv           # Should say "up to date"
touch input1.rds
make output/result.csv           # Should rebuild (input changed)
```

### Step 4: Dry Run
```bash
make --dry-run output/result.csv  # Show what would run
```

---

## Common Commands

```bash
# Render manuscript
quarto render paper/paper.qmd

# Run full pipeline
make all

# Single target (force rebuild)
make output/result.csv --always-make

# Show what would run
make --dry-run all

# Debug: show dependencies
make --debug all 2>&1 | head -50

# Clean all outputs
make clean

# Run specific targets
make output/figures/plot1.png output/tables/summary.csv
```

---

## Shared Configuration (shared_inputs.R)

```r
# R/shared_inputs.R

# Study design
COUNTRY <- "Country_Name"
SEED <- 42
ANALYSIS_YEARS <- 2017:2023

# Paths
DATA_DIR <- "data"
OUTPUT_DIR <- "output"

# Model parameters (no values 2+ scripts must agree on go here;
# those go in shared_inputs.R)
```

Every script:
```r
source("R/shared_inputs.R")  # Now have COUNTRY, SEED, etc.
# Never redefine these values
```

---

## No Print Output Rule

**Bad:**
```r
message(sprintf("Processed %d rows", nrow(data)))
cat("Model accuracy: ", acc, "\n")
print("Done!")
```

**Good:**
```r
# Write to output file
meta <- tibble(n_rows = nrow(data))
write_csv(meta, "output/metadata.csv")

# Only errors use print functions
if (nrow(data) == 0) {
  cli_abort("No data passed validation")
}
if (is_na(result)) {
  cli_warn("Result is NA; check input")
}
```

The manuscript reads `output/metadata.csv`, so every number appears in a committed file.

---

## Roxygen2 Function Template

```r
#' Descriptive title (one line)
#'
#' One-sentence summary of what it does.
#'
#' @details
#' How it works and every assumption:
#' - Expects columns x, y, group (numeric)
#' - Assumes x is sorted ascending
#' - Returns one row per group
#' - Assumes data is balanced (no missing groups)
#'
#' @param data A tibble with columns x, y, group.
#' @param threshold Numeric; cutoff for filtering.
#'
#' @return Tibble with one row per group.
#'
#' @export
#'
#' @examples
#' my_func(data, threshold = 0.5)
#'
my_func <- function(data, threshold = 0.5) {
  # body
}
```

After editing:
```bash
make docs  # Regenerates man/ and NAMESPACE
```

---

## Tidyverse Style Checklist

- [ ] `library(tidyverse)` + individual packages
- [ ] `dplyr::filter()`, `mutate()`, not base subsetting
- [ ] `tidyr::pivot_*()` not `gather()`, `spread()`
- [ ] `.by =` for grouping, not `group_by()` + `ungroup()`
- [ ] `map() + list_rbind()` not `map_dfr()`
- [ ] Native pipe `|>` not `%>%`
- [ ] Lambda `\(x)` not `function(x)`
- [ ] `slice_sample()` not `sample_n()`, `sample_frac()`
- [ ] Formatted with `air format R/`

---

## Example: 3-Stage Pipeline (Complete)

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

### R/shared_inputs.R
```r
SEED <- 42
ANALYSIS_YEARS <- 2017:2023
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
  mutate(across(where(is.numeric), scale))

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
preds <- predict(model, data)$predictions

scores <- tibble(
  auc = MLmetrics::AUC(data$outcome, preds),
  n_test = nrow(data)
)

write_csv(scores, output)
```

### Run It
```bash
make all            # Builds all in order
make output/model.rds --always-make  # Force rebuild one stage
make --dry-run      # Show what would run
```

---

## Error Signalling (Only These Two)

```r
# Violated precondition (stop execution)
if (!file.exists(input)) {
  cli_abort("File {input} not found")
}

# Validity concern (warn and continue)
if (nrow(data) < 100) {
  cli_warn("Data has {nrow(data)} rows; results may be unreliable")
}

# NOT these:
# message()          # No informational output
# print()            # Use cli_abort/warn instead
# cat()              # Narration goes in output files
# sprintf()          # Write results to CSV
```

---

## Humanizer Checklist (For Prose and Comments)

After editing prose or comments, remove:

- [ ] "Not X but Y" contrasts (state directly)
- [ ] "That is the real win" closers (unnecessary repetition)
- [ ] Forced triads (three items crammed together)
- [ ] Excessive dashes (—)
- [ ] Inflated language (pivotal, crucial, testament, featured)
- [ ] Chatbot phrases (I hope this helps, Let's dive in)
- [ ] Comments that just restate the code (rationale goes in Methods; comments explain WHY when rationale isn't obvious)

**Tool:** `skill humanizer` for quick polish

---

## Pattern Rules (Advanced)

```makefile
# Apply one rule to many targets
INPUTS := input1 input2 input3

# Pattern: transform each input
output/processed_%.rds: data/raw_%.csv R/process.R R/shared_inputs.R | output
	Rscript $^ $@

# Target all processed files
all_processed: $(patsubst data/raw_%.csv, output/processed_%.rds, $(wildcard data/raw_*.csv))

# Override for special case
output/processed_input1.rds: data/raw_input1_v2.csv R/process.R R/shared_inputs.R | output
	Rscript $^ $@
```

---

**Version:** 1.0.3  
**Last updated:** 8 November 2026
