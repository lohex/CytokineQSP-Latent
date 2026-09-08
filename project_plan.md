# CytokineQSP-Latent — Project Plan

## Project summary

**CytokineQSP-Latent** combines mechanistic quantitative systems pharmacology (QSP) models with latent neural dynamics to infer cytokine signaling from sparse, heterogeneous perturbation data.

Its primary use case is **IL-4/IL-13-driven airway epithelial remodeling in type 2 asthma**. The scientific aim is to connect receptor binding, receptor trafficking, STAT6 activation, transcriptional programs, and slower epithelial remodeling while keeping directly measured and latent quantities distinct.

The project is organized into three evidence layers:

1. **Phase 1:** receptor-proximal IL-13 signaling and STAT6 kinetics, centered on Moraga et al.
2. **Phase 2:** early and intermediate transcriptional responses, initially using GSE56308.
3. **Phase 3:** chronic airway epithelial remodeling and donor heterogeneity, initially using GSE240741.

Phase 1 establishes a mechanistic kinetic core. Latent or neural components are introduced only where later transcriptional and phenotypic observations cannot be explained by that core.

## Scientific scope

### Biological pathway

The initial scope centers on IL-13 signaling through the type II IL-4 receptor system:

- IL-13 binding to IL-13Rα1,
- recruitment of IL-4Rα and formation of the signaling ternary complex,
- receptor internalization and endosomal trafficking,
- STAT6 phosphorylation and nuclear translocation,
- optional IL-13Rα2-mediated sequestration and clearance,
- early transcriptional activation,
- persistent effector and remodeling programs,
- chronic epithelial state transitions,
- donor-to-donor variability where supported by the data.

IL-13Rα2 is treated as an optional context-dependent module. It is relevant to cytokine clearance and tissue regulation, but it must not be inferred from A549 experiments that do not express appreciable IL-13Rα2.

### Modeling principle

The model should remain as small and mechanistically constrained as the data allow. Known receptor binding and trafficking structure should be represented mechanistically. Neural components are reserved for residual dynamics, nonlinear coupling, or observation mappings that are not adequately specified.

The common interface should support three model families:

1. a mechanistic receptor and STAT6 reference model,
2. a latent neural ODE baseline,
3. a hybrid universal differential equation combining the mechanistic core with learned transcriptional or remodeling terms.

Biophysical measurements from different assay contexts must not be pooled as if they measured identical parameters. Three-dimensional SPR or ITC constants, two-dimensional membrane association constants, and effective cellular rate constants require separate parameter definitions or explicit mapping assumptions.

## Phase 1 — Receptor-proximal IL-13 and STAT6 kinetics

### Primary experimental source

**Moraga et al. (2015), “Instructive roles for cytokine-receptor binding parameters in determining signaling and functional potency,”** is the primary Phase 1 source.

The study provides a particularly informative perturbation design:

- SPR-derived binding kinetics for wild-type IL-13 and engineered IL-13 variants,
- pSTAT6 dose-response measurements,
- time-resolved pSTAT6 measurements in A549 cells,
- STAT6 nuclear translocation measurements,
- graded IL-13Rα1 knockdown experiments,
- receptor and ligand endocytosis measurements,
- pharmacological perturbation of endocytosis,
- distal cellular responses,
- a published mechanistic model connecting binding, trafficking, and STAT6 activation.

Primary reference: [Moraga et al., Science Signaling 2015](https://doi.org/10.1126/scisignal.aab2677)

The paper, supplementary material, and extracted numerical data should be versioned as a source bundle with explicit provenance. Figure-derived values must be marked as digitized rather than raw measurements.

### Biophysical prior sources

Phase 1 uses the following studies as prior information rather than as interchangeable observations:

| Source | Main contribution | Intended model use |
|---|---|---|
| [Lupardus et al., Structure 2010](https://doi.org/10.1016/j.str.2010.01.003) | Three-dimensional SPR kinetics for IL-13 binding to IL-13Rα1 and IL-13Rα2 | Priors for initial ligand binding and the optional decoy-receptor module |
| [LaPorte et al., Cell 2008](https://doi.org/10.1016/j.cell.2007.12.030) | Sequential type II receptor assembly, ITC thermodynamics, IL-4Rα recruitment, and dose-dependent STAT6 kinetics in A549 cells | Priors for ternary complex formation and external validation of signaling delays |
| [Richter et al., Nature Communications 2017](https://doi.org/10.1038/ncomms15976) | Two-dimensional membrane association and dissociation, receptor dimerization, diffusion, and single-molecule cellular measurements | Priors for membrane-localized complex formation and effective complex lifetime |

Reported constants may differ substantially across constructs and assays. Priors should therefore include source-specific uncertainty and, where necessary, experiment-specific scaling parameters.

### Objective

Build and validate the smallest receptor-to-STAT6 model that can jointly explain:

1. wild-type IL-13 dose and time responses,
2. altered signaling caused by IL-13 affinity variants,
3. altered signaling after IL-13Rα1 knockdown,
4. changes caused by perturbing receptor endocytosis.

The goal is not merely to reproduce one pSTAT6 trajectory. The perturbations should constrain which combinations of binding, receptor abundance, trafficking, and signal-generation parameters are identifiable.

### Mechanistic model

The initial model should contain approximately **6 to 9 dynamic states**:

- extracellular free IL-13,
- free surface IL-13Rα1,
- binary IL-13:IL-13Rα1 complex,
- free surface IL-4Rα,
- ternary signaling complex,
- internalized signaling complex,
- phosphorylated STAT6,
- optional nuclear STAT6,
- optional adaptation or negative-feedback state.

An optional IL-13Rα2 extension may add:

- free IL-13Rα2,
- surface IL-13:IL-13Rα2 complex,
- internalized decoy complex or a lumped clearance state.

The first implementation should omit states that are not distinguishable from available measurements. Conservation laws and quasi-steady-state reductions should be evaluated explicitly rather than assumed.

### Prior and likelihood strategy

Use weakly or moderately informative distributions centered on the biophysical measurements. Do not fix all binding constants at their literature point estimates.

The calibration should distinguish:

- assay-level measurement noise,
- biological replicate variability,
- figure-digitization error,
- differences between soluble ectodomains and membrane receptors,
- differences between A549, HeLa, and reconstituted membrane systems,
- uncertainty in absolute receptor abundance.

A hierarchical likelihood or source-specific nuisance parameters are preferable to forcing all experiments onto one absolute scale.

### Data workflow

1. Register Moraga et al. and all supplementary files in a versioned source manifest.
2. Inventory every usable condition, dose, time point, variant, perturbation, replicate, and measurement modality.
3. Extract tabulated values directly when available.
4. Digitize figure-only observations with stored figure identifier, axis transformation, extraction method, and estimated digitization uncertainty.
5. Store binding constants and membrane kinetic measurements in separate, unit-aware tables.
6. Create a machine-readable mapping from each observation to model state, observable, scaling factor, and noise model.
7. Preserve processed Phase 1 data under `data/processed/phase1_receptor/`.
8. Add automated checks for units, condition labels, variant identities, and duplicated observations.

### Milestones

1. Audited experimental inventory and source manifest.
2. Reproducible extraction of Moraga binding, pSTAT6, nuclear-translocation, knockdown, and endocytosis data.
3. Structured prior tables from Lupardus, LaPorte, and Richter.
4. Reimplementation of the published mechanistic model or a documented minimal equivalent.
5. Calibration to a predefined subset of wild-type conditions.
6. Prediction of held-out doses and time points.
7. Prediction of held-out affinity variants.
8. Validation against receptor knockdown and endocytosis perturbations.
9. Profile-likelihood, posterior, or ensemble-based identifiability analysis.
10. Decision on whether an additional latent signaling state is supported.

### Deliverables

- machine-readable receptor-kinetic dataset with provenance,
- biophysical-prior table with units and assay context,
- mechanistic receptor and STAT6 model,
- parameter distributions and identifiability results,
- held-out perturbation predictions,
- reproducible configuration files and figures,
- documented decision on inclusion of nuclear STAT6, feedback, and IL-13Rα2 states.

### Validation strategy

Validation should exploit interventions rather than random point-wise splits:

- leave-one-dose-out prediction,
- leave-one-variant-out prediction,
- training on unperturbed cells and testing receptor knockdown,
- training without endocytosis inhibition and predicting the inhibitor experiment,
- external comparison with the LaPorte A549 dose and time data,
- posterior predictive checks stratified by experiment and modality,
- comparison with occupancy-only and no-trafficking baselines.

Time points from the same experimental trajectory are correlated and must not be treated as independent biological replicates.

## Phase 2 — Early and intermediate transcriptional response

### Dataset

**GSE56308** is the initial transcriptomic dataset for this phase. It contains IL-13 time courses at approximately 0, 2, 4, 8, 12, and 24 hours in primary adult human dermal fibroblasts, with two independently performed time courses. Parallel IL-4 experiments provide an additional shared-receptor comparison.

GEO record: [GSE56308](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE56308)

This dataset improves temporal coverage over GSE3183 and uses primary cells, but it is not an airway epithelial dataset. It should therefore constrain generic IL-13/STAT6-to-transcription dynamics without supporting tissue-specific airway claims.

GSE3183 is no longer the primary Phase 1 dataset. It may be retained as an optional A549 transcriptional comparator after its treatment design and metadata limitations have been accounted for.

### Objective

Connect the Phase 1 receptor and STAT6 core to early transcriptional modules. Test whether a compact mechanistic observation model is sufficient or whether one or two latent transcriptional coordinates improve prediction across time and between IL-13 and IL-4.

### Model extension

Phase 2 may add approximately **2 to 4 dynamic states**:

- immediate STAT6-responsive transcription,
- delayed transcriptional response,
- persistent effector program,
- optional negative-feedback or adaptation module.

The Phase 1 receptor parameters should be propagated as distributions rather than refitted freely. Cell-type differences should enter through receptor abundance, effective coupling, transcriptional rates, and observation mappings.

### Data workflow

1. Download and preserve GSE56308 raw data and annotations.
2. Audit the two time courses, treatment conditions, reference-channel design, and replicate structure.
3. Normalize all arrays with a documented and versioned procedure.
4. Define pathway-informed and data-derived transcriptional modules.
5. Map Phase 1 pSTAT6 or integrated signal exposure to the transcriptional observations.
6. Store processed data under `data/processed/phase2_transcriptomics/`.
7. Record every cross-dataset scaling or batch parameter explicitly.

### Milestones and validation

1. Reproducible GSE56308 ingestion and QC.
2. Stable transcriptional modules with biological annotation.
3. Mechanistic Phase 1 core coupled to a linear or saturating transcription model.
4. Latent neural ODE and hybrid alternatives using the same observations.
5. Hold out complete time points or one independent time course.
6. Test transfer between IL-13 and IL-4 while respecting their different receptor-binding order.
7. Compare models at matched effective complexity.
8. Quantify uncertainty from Phase 1 parameter transfer.

Genes from the same sample are not independent replicates. Resampling and validation should operate at the array, time-course, or condition level.

## Phase 3 — Chronic airway epithelial remodeling

### Dataset

**GSE240741** is retained for chronic, multi-day epithelial remodeling and donor-dependent state transitions.

GEO record: [GSE240741](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE240741)

### Objective

Extend the receptor and transcriptional model with slow epithelial remodeling, a dataset-specific observation model, and a donor-population layer. Phase 1 and Phase 2 parameters should be reused or regularized toward their posterior distributions where transfer is biologically defensible.

### Model extension

Phase 3 should introduce only the additional structure required for chronic data:

- one to three slow remodeling states,
- epithelial cell-state or composition observations,
- donor-specific random effects or hierarchical parameter distributions,
- coupling between cumulative acute signaling exposure and long-term remodeling,
- dataset-specific measurement noise and batch terms.

The expected combined model size is approximately **9 to 14 dynamic states**, depending on whether nuclear STAT6, feedback, and IL-13Rα2 are retained.

### Data workflow

1. Download and preserve GSE240741 source data and annotations.
2. Resolve donor, treatment, dose, duration, cell-state, and batch metadata.
3. Quantify donor coverage and missingness before adding population parameters.
4. Construct chronic response modules and cell-state observations.
5. Map compatible Phase 2 outputs to Phase 3 inputs.
6. Store processed data under `data/processed/phase3_remodeling/`.

### Milestones and validation

1. Donor-aware QC and metadata audit.
2. Chronic module and cell-state observation definitions.
3. Transfer of the acute core with uncertainty propagation.
4. Incremental addition of slow remodeling dynamics.
5. Hierarchical or mixed-effects donor model.
6. Leave-one-donor-out or grouped K-fold validation.
7. Held-out treatment-duration prediction.
8. Ablation of mechanistic, latent, and population components.
9. Calibration and predictive-interval coverage analysis.

## Cross-phase engineering plan

### Shared interfaces

All models should expose a common interface for:

- state initialization,
- right-hand-side evaluation,
- simulation under specified cytokine inputs,
- observation mapping,
- loss or likelihood computation,
- prior specification,
- parameter serialization,
- intervention and counterfactual simulation.

Dataset-specific logic belongs under `src/cytokineqsp_latent/data/datasets/`. Literature-derived kinetic extraction should use a dedicated source adapter rather than being mixed with GEO ingestion code.

Suggested dataset identifiers:

- `moraga2015`
- `lupardus2010`
- `laporte2008`
- `richter2017`
- `gse56308`
- `gse240741`

### Reproducibility

- Configurations are versioned under `configs/`.
- Raw copyrighted article files are not committed unless redistribution is permitted.
- Digitized data include source citation, figure and panel identifier, extraction version, and uncertainty.
- Processed artifacts record source accession or DOI, checksum where feasible, preprocessing version, and configuration.
- Units are explicit and validated programmatically.
- Random seeds and software versions are stored with fitted outputs.
- Tests include schema checks, unit checks, model shape checks, numerical smoke tests, and synthetic parameter-recovery tests.

## Decision gates

### Gate 0: Phase 1 extraction quality

Proceed to calibration only if the Moraga observations and literature priors can be represented with sufficient numerical precision and provenance. Otherwise restrict the first implementation to qualitative reproduction and synthetic validation.

### Gate 1: receptor-model identifiability

Retain only kinetic parameters or parameter combinations constrained by dose, affinity-variant, receptor-knockdown, or endocytosis perturbations. Weakly identifiable parameters should remain prior-dominated, fixed by explicit assumption, or be lumped.

### Gate 2: IL-13Rα2 extension

Add IL-13Rα2 only when an applicable dataset measures receptor abundance, cytokine clearance, or decoy-receptor perturbation. Do not infer this module solely from A549 pSTAT6 data.

### Gate 3: latent transcriptional layer

Introduce a latent transcriptional state only if it improves held-out GSE56308 trajectories or resolves structured residuals beyond uncertainty. Otherwise retain a direct mechanistic observation mapping.

### Gate 4: chronic extension

Add slow remodeling states incrementally. Each state must improve grouped validation or resolve a biologically interpretable residual pattern.

### Gate 5: donor hierarchy

Use hierarchical effects only for parameters supported across donors. Weakly informed donor-specific parameters should be partially pooled or fixed.

### Gate 6: neural capacity

Increase neural capacity only after checking optimizer stability, data leakage, kinetic-model misspecification, and observation-model misspecification. Report capacity and effective parameter count for every comparison.

## Expected outcome

The final repository should provide an interpretable, reusable framework that separates four evidence levels:

1. biophysical receptor interactions,
2. cellular receptor trafficking and STAT6 activation,
3. transcriptional response dynamics,
4. chronic airway epithelial remodeling.

The central test is whether a mechanistically anchored receptor and STAT6 model can be transferred through a compact latent transcriptional layer to chronic remodeling data, and which aspects remain cell-type-specific or unidentifiable from the available experiments.
