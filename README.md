# r-gnumake-orchestrator

Generic patterns for orchestrating R analysis pipelines using GNU Make 3.81. Suitable for data science, ML, forecasting, and research workflows.

## Three Core Rules

1. **Files are targets:** Every .csv, .rds, .png is a Make target; only rebuild when inputs change
2. **Scripts accept arguments:** Order: input files → helper script → output file
3. **One source of truth:** `R/shared_inputs.R` holds all study-wide values

## Start Here

- **New to this?** → Read SKILL.md (complete reference)
- **Need a template?** → QUICK-REFERENCE.md (copy-paste examples)
- **Looking for something?** → INDEX.md (navigation guide)

## When to Use

✓ Data analysis pipelines with 5+ stages  
✓ Projects needing reproducibility  
✓ Multi-person teams  
✓ Large datasets  

✗ Throwaway scripts  
✗ Single-file analyses  
✗ Software packages  

## Quick Example

**Makefile:**
```makefile
output/results.csv: data/raw.rds R/process.R R/shared_inputs.R | output
	Rscript $^ $@
```

**R/process.R:**
```r
args <- commandArgs(trailingOnly = TRUE)
input <- if (length(args) >= 1) args[1] else "data/raw.rds"
helper <- if (length(args) >= 2) args[2] else "R/shared_inputs.R"
output <- if (length(args) >= 3) args[3] else "output/results.csv"

source(helper)
data <- read_rds(input)
results <- analyze(data)
write_csv(results, output)
```

**Run:**
```bash
make output/results.csv  # Only rebuilds if inputs changed
```

---

**License:** MIT  
**Version:** 1.0.0  
See SKILL.md for complete documentation.
