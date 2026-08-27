# IDE Bayesian MCMC Stress Testing

An exploratory study of model selection, posterior uncertainty, and predictive
model averaging for infinite-distance extrapolation (IDE)-style problems in
quantum error correction.

The project asks a narrow question:

> When logical-error data are available at only a few finite code distances,
> how reliably can predictive model comparison identify a functional form for
> extrapolation toward infinite distance?

This repository is an independent statistical investigation inspired by IDE.
It is **not** the reference IDE implementation and does not audit or modify the
original Levenberg--Marquardt/bootstrap codebase.

## Project status

**Research prototype.** The notebooks demonstrate the intended workflow, but
the repository is not yet a validated end-to-end QEC benchmark. In particular:

- the principal coverage experiments use synthetic exponential data;
- the current PyMC models use a Gaussian approximation for transformed
  expectation values rather than a direct Binomial likelihood;
- model comparison uses exact leave-one-distance-out refitting, not PSIS-LOO;
- model weights are obtained by applying a softmax to held-out ELPD totals;
  this is predictive weighting and should not be confused with conventional
  evidence-based Bayesian model averaging;
- the Stim/PyMatching section is a stand-in circuit smoke test and is not yet
  connected to the main coverage experiment;
- some saved double-exponential fits show sampling warnings and must be
  reparameterized before their numerical results are treated as final.

These limitations are documented deliberately so that exploratory results are
not mistaken for submission-ready scientific conclusions.

## Current repository contents

| File | Purpose |
| --- | --- |
| `Main coverage-test suit.ipynb` | Synthetic-data generation, PyMC model fitting, exact leave-one-distance-out comparison, coverage experiments, predictive mixture, and a Stim/PyMatching smoke test. |
| `Severity-sweep robustness results.ipynb` | Earlier severity-sweep experiments. Some cells use an older variance specification and should be considered historical until regenerated with the main likelihood. |
| `Research results` | Saved exploratory output/artifact currently included in the repository. |
| `README.md` | Project description, reproduction instructions, and limitations. |

## Implemented experiment

### Synthetic data

The main notebook currently evaluates five code distances:

```text
d = {5, 7, 9, 11, 13}
```

Logical-error probabilities are generated from either a single- or
double-exponential curve. Binomial shot noise is then applied:

```math
k_d \sim \operatorname{Binomial}(n_{\text{shots}}, p_L(d)).
```

The simulated failure rate is converted into a logical expectation value used
by the present PyMC likelihood.

### Candidate models

The public notebook currently compares:

```math
E_{\text{single}}(d) = A + B e^{-Cd},
```

and

```math
E_{\text{double}}(d)
  = A + B_1 e^{-C_1 d} + B_2 e^{-C_2 d}.
```

The asymptotic parameter `A` represents the inferred infinite-distance limit.

### Predictive comparison

For every candidate model and held-out distance, the notebook:

1. removes one complete distance;
2. refits the model on the remaining distances;
3. evaluates the posterior predictive log density at the held-out distance;
4. sums the held-out contributions to obtain an exact LOO ELPD total.

Because a complete code distance is held out, this procedure tests transfer
between distance settings rather than merely leaving out individual shots at
an already observed distance.

### Predictive mixture

The current experimental mixture converts total ELPD values into softmax
weights and pools posterior draws of `A`. This is best described as an
**ELPD-weighted predictive mixture**. Future versions should compare it with
stacking, Bayesian-bootstrap pseudo-BMA, and evidence-based BMA.

## Installation

The project requires a recent Python 3 installation and Jupyter. Core packages
used by the notebooks include:

```text
numpy
pandas
matplotlib
scipy
pymc
arviz
stim
pymatching
```

Until a pinned dependency file is added, install these packages using your
normal Python environment-management workflow. Do not assume that results from
different PyMC, ArviZ, or sampler versions will be numerically identical.

## Running the notebooks

Clone the repository and start Jupyter from its root directory:

```bash
git clone https://github.com/Zikria11/IDE-Bayesian-MCMC-Stress-Testing.git
cd IDE-Bayesian-MCMC-Stress-Testing
jupyter lab
```

Then:

1. open `Main coverage-test suit.ipynb`;
2. restart the kernel;
3. run all cells from top to bottom;
4. inspect every sampler diagnostic before accepting the stored results;
5. treat `Severity-sweep robustness results.ipynb` as historical until its
   likelihood is synchronized with the main notebook.

The full coverage loops can be computationally expensive because exact LOO
requires a separate MCMC fit for every held-out distance, model, and simulated
dataset.

## Required diagnostic checks

Before reporting a run, verify at least:

- split-\(\hat R\) values are acceptably close to one;
- effective sample sizes are adequate for all parameters and the asymptote;
- there are no unresolved divergent transitions;
- no chains repeatedly reach the maximum tree depth;
- trace plots show satisfactory mixing;
- posterior conclusions are stable under defensible prior changes;
- Monte Carlo uncertainty is small relative to the reported effect.

A completed sampler call is not, by itself, evidence that the posterior is
trustworthy.

## Reproducibility policy

Any result used in a paper or poster should be traceable to:

- a committed source revision;
- a documented random seed;
- package and Python versions;
- the exact distance set and shot count;
- the data-generating function and its parameters;
- the candidate-model definitions and priors;
- sampler settings and diagnostics;
- the number of repeated coverage trials;
- a machine-readable result table generated by the run.

Numbers not reproducible from the public repository should not be presented as
project findings.

## Interpretation boundaries

The current experiment can investigate whether sparse-distance predictive
comparison distinguishes a single exponential from a double exponential under
the selected priors and simulation settings. It does not yet establish:

- a universal minimum number of code distances;
- a universal ELPD threshold for selecting an extrapolation ansatz;
- nominal calibration under arbitrary structural misspecification;
- superiority of predictive averaging over every component model;
- an end-to-end quantum-hardware or circuit-level IDE result;
- conclusions about the correctness of the original IDE implementation.

The current double-exponential data-generating process is included among the
candidate models. Consequently, those runs are not an `M`-open experiment in
which every candidate family is structurally misspecified.

## Planned work

- [ ] Connect decoded Stim/PyMatching failure counts directly to inference.
- [ ] Add a direct Binomial likelihood for logical-failure counts.
- [ ] Reparameterize the double-exponential model and eliminate sampling
      pathologies.
- [ ] Add exact highest-distance and leave-future-out validation.
- [ ] Compare stacking, BB-pseudo-BMA, and evidence-based BMA.
- [ ] Test data-generating processes outside every candidate family.
- [ ] Run adequately powered coverage studies with confidence intervals.
- [ ] Add pinned dependencies and automated reproducibility checks.
- [ ] Export tidy CSV/JSON result tables from every experiment.
- [ ] Add unit tests for data generation, likelihoods, weighting, and coverage.

## References

1. G. Umbrarescu, O. Higgott, and D. E. Browne, *Infinite Distance
   Extrapolation: How error mitigation can enhance quantum error correction*,
   arXiv:2603.11285 (2026).
2. M. A. Connell, I. Billig, and D. R. Phillips, *Does Bayesian Model
   Averaging improve polynomial extrapolations?*, Journal of Physics G 48,
   104001 (2021), arXiv:2106.05906.
3. A. Vehtari, A. Gelman, and J. Gabry, *Practical Bayesian model evaluation
   using leave-one-out cross-validation and WAIC*, Statistics and Computing 27,
   1413--1432 (2017).
4. Y. Yao, A. Vehtari, D. Simpson, and A. Gelman, *Using stacking to average
   Bayesian predictive distributions*, Bayesian Analysis 13, 917--1007 (2018).
5. R. H. Berk, *Limiting behavior of posterior distributions when the model is
   incorrect*, Annals of Mathematical Statistics 37, 51--58 (1966).

## Authors

- **Zikria Akhtar**
- **Muhammad Immad Ahmed Khan**

Department of Software Engineering  
Capital University of Science and Technology (CUST)  
Islamabad, Pakistan

## License

No software license has yet been added. Until a license is selected, normal
copyright restrictions apply and reuse rights are not granted automatically.
Add an explicit open-source license before describing the repository as fully
open source.
