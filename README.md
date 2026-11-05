# r-gnumake-orchestrator

**A Claude skill** for generic patterns to orchestrate R analysis pipelines using GNU Make 3.81. Suitable for data science, ML, forecasting, and research workflows.

## Installation

### Option 1: Clone into Claude Skills (Recommended)

```bash
git clone https://github.com/jamesmbaazam/r-gnumake-orchestrator.git \
  ~/.claude/skills/r-gnumake-orchestrator
```

Then load in Claude with `/skill r-gnumake-orchestrator`.

### Option 2: Reference from GitHub

Just link to this repository in Claude:

```
/skill https://github.com/jamesmbaazam/r-gnumake-orchestrator
```

### Option 3: Copy Files to Your Project

```bash
cp -r ~/.claude/skills/r-gnumake-orchestrator/SKILL.md your-project/docs/
```

## Why GNU Make with R?

Data analysis pipelines involve many interdependent stages (prepare → fit → score → visualize). GNU Make excels at this because:

- **Dependency tracking:** Automatically reruns only the stages whose inputs changed—skip expensive retraining when data prep fails
- **Universal:** Works everywhere (Linux, macOS, Windows with WSL); no language-specific tool needed
- **Language-agnostic:** Orchestrates R, Python, shell, any executable in one workflow
- **Parallel execution:** `make -j4` runs independent targets simultaneously
- **Human-readable:** Makefiles are plain text; version control and review are straightforward

### Alternatives to GNU Make

| Tool | Pros | Cons | When to Use |
|------|------|------|------------|
| [GNU Make](https://www.gnu.org/software/make/) | Universal, dependency tracking, parallel | Cryptic syntax (tabs, quoting) | Large pipelines, multi-language workflows |
| [Task](https://taskfile.dev/) | YAML syntax, friendly, task dependencies | R-specific docs limited | Medium pipelines, prefer YAML over Makefile syntax |
| [targets](https://docs.ropensci.org/targets/) | R-native, reproducibility-first, dynamic branching | R-only, steeper learning curve | R packages, complex dependency graphs |
| [Snakemake](https://snakemake.readthedocs.io/) | Python ecosystem, file-based, rules-based | Python-heavy, overkill for small pipelines | Python-heavy teams, bioinformatics workflows |
| [Nextflow](https://www.nextflow.io/) | Scalable, parallel, cloud-native | Complex, designed for HPC/cloud | Large-scale parallel pipelines, HPC clusters |
| [Make (hand-rolled)](#) | Complete control | Easy to write wrong, no standards | Education only |

**This skill uses GNU Make** because it's lightweight, preinstalled on most systems, and forces clean separation between orchestration and analysis logic.

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

## Citing this skill

If you find it useful, you are welcome to cite it, though you don't have to. GitHub's "Cite this repository" button (from [`CITATION.cff`](CITATION.cff)) gives APA and BibTeX.

> Azam J. r-gnumake-orchestrator: a Claude Code skill for orchestrating R analysis pipelines with GNU Make (v1.0.0) [software]. 2026. https://github.com/jamesmbaazam/r-gnumake-orchestrator

```bibtex
@software{azam_r_gnumake_orchestrator_2026,
  author  = {Azam, James},
  title   = {r-gnumake-orchestrator: a Claude Code skill for orchestrating R analysis pipelines with GNU Make},
  version = {1.0.0},
  year    = {2026},
  url     = {https://github.com/jamesmbaazam/r-gnumake-orchestrator},
  license = {MIT}
}
```

---

**License:** MIT  
**Version:** 1.0.0  
See SKILL.md for complete documentation.
