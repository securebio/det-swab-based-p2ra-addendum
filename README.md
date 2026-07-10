# Estimating the sensitivity of wastewater metagenomic sequencing using nasal swabs

Simon Grimm, Dan Rice, Mike McLaren

Code and documentation supporting the NAO blog post [Estimating the sensitivity of wastewater metagenomic sequencing using nasal swabs](https://naobservatory.org/blog/swab-based-p2ra/)
and its [Appendix](https://naobservatory.github.io/swab-based-p2ra/).

## This fork

This fork extends the original analysis to additionally fit the model using a
**non-deduplicated** relative-abundance numerator. The denominator (total reads)
is unchanged; only the numerator differs: deduplicated human-virus reads
(`dedup_hv`, as in the original analysis) versus non-deduplicated reads
(`non_dedup_hv`). Both numerator columns are already present in the input tables
`tables/swabs-ra-summary.tsv` and `tables/ww-ra-summary.tsv`.

The non-deduplicated estimates are more convenient for plugging into models that
represent sequence duplication explicitly—e.g. as an independent parameter or as
a function of sequencing depth and library complexity.

## Reproducing the statistical analysis

To reproduce the model fits and diagnostics:

1. Clone this git repository.
2. From the R console, install the necessary R packages with: `install.packages(c("tidyverse", "here", "rstan"))`.
   RStan may require additional setup; see the [RStan docs](https://mc-stan.org/rstan/).
3. Run the model-fitting script with: `Rscript statistics/scripts/fit_model.R`.
   (This script should take 5--10 minutes to run.)
   
The model is fit twice, once for each definition of the relative-abundance
numerator: deduplicated reads (`dedup`) and non-deduplicated reads
(`non_dedup`). Each fit writes its results to a corresponding subdirectory of
`tables/`, so the two can be compared. For each variant the script outputs four
files (all of which are already checked into the repository), where `<variant>`
is `dedup` or `non_dedup`:

- `tables/<variant>/model_diagnostics.tsv` contains Stan diagnostic information,
  including the number of divergent transitions and the maximum rhat over all parameters.
- `tables/<variant>/posteriors.tsv` contains posterior samples for all viruses.
  Note that re-running the fitting script may shuffle the order of the posterior samples in this file.
- `tables/<variant>/{prevalence,ra01}_posterior_summary.tsv` contain posterior 15th,
  50th, and 85th percentiles for prevalence and $RA_p(1\%)$ for all viruses.

The downstream figures and notebooks (`scripts/violin_*_plot.py`,
`statistics/appendix/index.qmd`, `statistics/model-checking/index.qmd`) default
to the `dedup` variant via a one-line `VARIANT`/`variant` switch near the top of
each file; set it to `non_dedup` to regenerate those outputs from the
non-deduplicated fit.

## Comparing the two fits

`analysis/compare-fits/index.qmd` compares the `dedup` and `non_dedup` fits:
per-sample duplication rates, and the resulting differences in the posterior
estimates of prevalence and $RA_p(1\%)$. Render it with:
```
quarto render analysis/compare-fits/index.qmd
```
Broadly, dropping deduplication raises the $RA_p(1\%)$ estimates by roughly
`1 / (1 - duplication rate)` while leaving prevalence estimates largely
unchanged.

## Estimating library complexity

`analysis/complexity/index.qmd` estimates the library complexity (total number
of distinct DNA fragments) of composite WWTP CASPER samples processed by MU,
using vertebrate-virus read counts before and after deduplication together with
a saturation model. It reads
`analysis/complexity/data/casper-mu-vv-stats.tsv` and writes per-sample
complexity estimates to
`analysis/complexity/output/casper-mu-vv-stats-with-complexity.tsv`. Both this
output table and the rendered `analysis/complexity/index.html` are checked into
the repository, so they can be used without re-rendering. To regenerate them,
run:
```
quarto render analysis/complexity/index.qmd
```
These complexity estimates support models that represent sequence duplication
explicitly (see [This fork](#this-fork)).

## Publishing the appendix (NAO-internal only)

To publish an update to the appendix, make any changes, then run:
```
quarto publish gh-pages statistics/appendix/index.qmd
```
from the root of the repository.
