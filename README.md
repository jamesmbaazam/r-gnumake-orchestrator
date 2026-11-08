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

<details>
<summary><strong>Alternatives to GNU Make</strong></summary>

| Tool | Pros | Cons | When to Use |
|------|------|------|------------|
| [GNU Make](https://www.gnu.org/software/make/) | Universal, dependency tracking, parallel | Cryptic syntax (tabs, quoting) | Large pipelines, multi-language workflows |
| [Task](https://taskfile.dev/) | YAML syntax, friendly, task dependencies | R-specific docs limited | Medium pipelines, prefer YAML over Makefile syntax |
| [targets](https://docs.ropensci.org/targets/) | R-native, reproducibility-first, dynamic branching | R-only, steeper learning curve | R packages, complex dependency graphs |
| [Snakemake](https://snakemake.readthedocs.io/) | Python ecosystem, file-based, rules-based | Python-heavy, overkill for small pipelines | Python-heavy teams, bioinformatics workflows |
| [Nextflow](https://www.nextflow.io/) | Scalable, parallel, cloud-native | Complex, designed for HPC/cloud | Large-scale parallel pipelines, HPC clusters |
| [Make (hand-rolled)](#) | Complete control | Easy to write wrong, no standards | Education only |

</details>

**This skill uses GNU Make** because it's lightweight, preinstalled on most systems, and forces clean separation between orchestration and analysis logic.

## What the skill covers

`SKILL.md` is a guide for building reproducible R analysis pipelines with GNU Make. It combines two things: Make and R conventions for the pipeline itself, and best practices from the [tidyverse style guide](https://style.tidyverse.org/) and [design guide](https://design.tidyverse.org/).

**Pipeline patterns**

- **R script pattern:** every script takes its inputs and output as arguments, in a fixed order, and falls back to defaults when run interactively, so you can test a script on its own before wiring it into Make.
- **Makefile pattern:** each output file is a Make target, so only the stages whose inputs changed are rerun.
- **Shared configuration:** one file holds the values several scripts need, such as the seed or country.
- **File organisation and pattern rules:** how to lay out a project and write reusable Make rules.
- **Common patterns:** batch processing, generating all combinations of inputs, multi-output rules and aggregating results.

**Writing the R code**

- **Style and design:** naming, spacing, function design and code organisation follow the tidyverse guides. Functions go in separate files and are documented with roxygen2.
- **Package choices:** tidyverse functions come first, and deprecated functions are avoided. [data.table](https://rdatatable.gitlab.io/data.table/) is used only for intensive data manipulation, where speed matters.

**Working with the pipeline**

- **Validation workflow:** test a script interactively, then with arguments, then through Make, then with a dry run.
- **Worked example and checklist:** a three-stage pipeline you can follow, and a checklist for every new script.

## Resources to learn Make

- [GNU Make manual](https://www.gnu.org/software/make/manual/make.html): the complete official reference.
- [Ode to GNU Make](https://www.evan-soil.io/blog/ode-to-gnu-make/): a short, enthusiastic case for using Make.
- [Reproducibility with Make](https://book.the-turing-way.org/reproducible-research/make/): the chapter in The Turing Way.
- [minimal make](https://kbroman.org/minimal_make/): Karl Broman's quick tutorial for getting started.
- [Using GNU Make to manage the workflow of data analysis projects](https://www.jstatsoft.org/article/view/v094c01): a Journal of Statistical Software article.

## Citing this skill

If you find it useful, you are welcome to cite it, though you don't have to. GitHub's "Cite this repository" button (from [`CITATION.cff`](CITATION.cff)) gives APA and BibTeX.

> Azam J. r-gnumake-orchestrator: a Claude Code skill for orchestrating R analysis pipelines with GNU Make (v1.0.3) [software]. 2026. https://github.com/jamesmbaazam/r-gnumake-orchestrator

```bibtex
@software{azam_r_gnumake_orchestrator_2026,
  author  = {Azam, James},
  title   = {r-gnumake-orchestrator: a Claude Code skill for orchestrating R analysis pipelines with GNU Make},
  version = {1.0.3},
  year    = {2026},
  url     = {https://github.com/jamesmbaazam/r-gnumake-orchestrator},
  license = {MIT}
}
```

---

**License:** MIT  
**Version:** 1.0.3  
See SKILL.md for complete documentation.
