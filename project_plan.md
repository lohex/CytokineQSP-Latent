# CytokineQSP-Latent — Project Plan

## Project summary

**CytokineQSP-Latent** combines mechanistic quantitative systems pharmacology (QSP) models with latent neural dynamics to infer cytokine signaling from sparse, heterogeneous perturbation data.

Its primary use case is **IL-4/IL-13-driven airway epithelial remodeling in type 2 asthma**. The scientific aim is to connect early cytokine signaling and transcriptional programs to slower epithelial remodeling without pretending that the intracellular signaling layer is directly observed.

The project is organized in two dataset-driven phases. Phase 1 establishes the compact dynamic core. Phase 2 extends that core to chronic remodeling, heterogeneous donors, and richer observation models.

## Scientific scope

### Biological pathway

The initial scope centers on IL-4/IL-13 signaling through the shared type-2 cytokine receptor system and downstream transcriptional responses. The model should capture:

- effective receptor-proximal activation,
- one or two latent signaling coordinates,
- early transcriptional activation,
- slower effector and remodeling programs,
- chronic epithelial state transitions,
- donor-to-donor variability where supported by the data.

The latent states are phenomenological summaries. They are not automatically identifiable with a single molecular species such as phosphorylated STAT6 unless independent measurements justify that interpretation.

### Modeling principle

The model should remain as small and mechanistically constrained as the data allow. Neural components are used for residual dynamics, nonlinear coupling, or observation mappings that are not adequately specified mechanistically. They should not replace known causal structure without a measurable gain in predictive performance.

The common interface must support three model families:

1. a fully mechanistic reference model,
2. a latent neural ODE baseline,
3. a hybrid universal differential equation combining mechanistic and learned terms.

## Phase 1 — Early IL-13 signaling and transcriptional response

### Dataset

**GSE3183** is used for the early IL-13 response. The available time course includes approximately 0, 4, 12, and 24 hours with roughly three biological replicates per condition and time point.

### Objective

Build the smallest hybrid dynamical model that explains the early transcriptional response while representing uncertainty in the unobserved signaling layer.

### Model

The target size is **4–6 dynamic states**:

- (R): effective receptor-proximal activation,
- (Z_1): primary latent signaling coordinate,
- (Z_2): optional second latent coordinate,
- (E): early transcriptional response,
- (P): persistent effector program,
- (F): feedback or adaptation state.

The baseline mechanistic model should use a compact saturating activation and first-order relaxation structure. The neural ODE and hybrid UDE should use a small MLP, initially one or two hidden layers with approximately 8–32 hidden units. Model capacity must be treated as a controlled experimental variable.

### Data workflow

1. Download expression and sample metadata from GEO.
2. Preserve raw accession-derived files under `data/raw/GSE3183/`.
3. Harmonize sample identifiers, treatment, time, and replicate annotations.
4. Perform expression-level quality control and document exclusions.
5. Derive reproducible gene modules or low-dimensional observations.
6. Store processed matrices and metadata under `data/processed/phase1/`.

### Milestones

1. Reproducible ingestion and QC notebook.
2. Predefined or data-derived response modules with biological annotation.
3. Mechanistic baseline fitted with replicate-aware losses.
4. Neural ODE and hybrid UDE baselines using the same observation interface.
5. Model comparison using held-out conditions or time points and bootstrap uncertainty.
6. Identifiability and sensitivity analysis for the compact state model.

### Deliverables

- processed Phase 1 dataset and metadata schema,
- module definitions and observation mappings,
- fitted mechanistic, neural, and hybrid models,
- parameter and trajectory uncertainty estimates,
- comparison figures and reproducible configuration files,
- a documented decision on whether (Z_2) or additional states are supported.

### Validation strategy

Validation must avoid treating genes from the same sample as independent observations. The primary units of resampling are biological replicates and experimental conditions.

Use:

- leave-one-time-point-out prediction where estimable,
- replicate-level bootstrap intervals,
- multiple random initializations for latent models,
- comparison against constant, interpolation, and simple linear-dynamics baselines,
- parameter-profile or local sensitivity analysis,
- posterior predictive or residual checks stratified by time and module.

Model selection should favor the smallest model whose predictive uncertainty and held-out error are not materially worse than a larger alternative.

## Phase 2 — Chronic epithelial remodeling

### Dataset

**GSE240741** is used for chronic, multi-day epithelial remodeling and donor-dependent state transitions.

### Objective

Extend the Phase 1 dynamic core with slow remodeling, a dataset-specific observation model, and a donor-population layer. The early-response parameters should be reused or regularized toward Phase 1 estimates where the data support transfer.

### Model extension

Phase 2 should introduce only the additional structure required for chronic data:

- one to three slow remodeling states,
- cell-state or epithelial-composition observations,
- donor-specific random effects or hierarchical parameter distributions,
- optional coupling between acute signaling exposure and long-term remodeling,
- dataset-specific measurement noise and batch terms.

The expected total size is approximately **6–10 dynamic states**, plus an observation model and a low-dimensional donor hierarchy. A learned residual component should remain small relative to the number of independent donors and perturbation conditions.

### Data workflow

1. Download and preserve GSE240741 source data and annotations.
2. Resolve donor, treatment, dose, duration, cell-state, and batch metadata.
3. Quantify donor coverage and missingness before model design.
4. Construct chronic response modules and cell-state observations.
5. Map compatible Phase 1 outputs to Phase 2 inputs.
6. Store processed matrices under `data/processed/phase2/`.

### Milestones

1. Donor-aware QC and metadata audit.
2. Chronic module and cell-state observation definitions.
3. Transfer of the Phase 1 core with uncertainty propagation.
4. Addition of slow remodeling dynamics.
5. Hierarchical or mixed-effects donor model.
6. External validation of acute components against GSE3183.
7. Ablation of mechanistic, latent, and population components.

### Deliverables

- processed Phase 2 dataset and donor metadata,
- extended chronic-remodeling model,
- population parameter estimates and donor-level predictions,
- cross-dataset validation results,
- calibrated uncertainty for trajectories and derived response scores,
- final benchmark comparing mechanistic, neural, and hybrid variants.

### Validation strategy

Use grouped splits that keep all observations from a donor together. Depending on the number of donors and experimental coverage:

- leave-one-donor-out or grouped K-fold validation,
- held-out treatment-duration combinations,
- Phase 1-to-Phase 2 transfer tests,
- Phase 2-to-GSE3183 external checks for the acute response,
- bootstrap over donors rather than individual genes or time points,
- calibration curves and predictive interval coverage,
- ablations for slow states, donor effects, and learned residuals.

Performance claims should be reported with uncertainty and compared at matched parameter count where possible.

## Cross-phase engineering plan

### Shared interfaces

All models should expose a common interface for:

- state initialization,
- right-hand-side evaluation,
- simulation under specified cytokine inputs,
- observation mapping,
- loss computation,
- parameter serialization,
- intervention or counterfactual simulation.

Dataset-specific logic belongs under `src/cytokineqsp_latent/data/datasets/`. Reusable preprocessing, observation, inference, and plotting code belongs in the package rather than notebooks.

### Reproducibility

- Configurations are versioned under `configs/`.
- Raw data are not committed.
- Processed artifacts record source accession, checksum where feasible, preprocessing version, and configuration.
- Every result should be reproducible from a configuration and an immutable input-data manifest.
- Random seeds and software versions must be stored with fitted outputs.
- Tests should include data-schema checks, model shape checks, numerical smoke tests, and at least one end-to-end synthetic recovery test.

## Decision gates

### Gate 1: Phase 1 observability

Proceed to a two-coordinate latent signaling layer only if it improves held-out prediction or residual structure beyond uncertainty. Otherwise retain one latent coordinate.

### Gate 2: chronic extension

Add slow states incrementally. Each added state must improve grouped validation or resolve a biologically interpretable residual pattern.

### Gate 3: donor hierarchy

Use hierarchical effects only for parameters supported across donors. Weakly informed donor-specific parameters should be partially pooled or fixed.

### Gate 4: neural capacity

Increase neural capacity only after checking optimizer stability, data leakage, and observation-model misspecification. Report capacity and effective parameter count for every comparison.

## Expected outcome

The final repository should provide an interpretable, reusable framework for testing which aspects of cytokine-driven epithelial remodeling can be explained by compact mechanistic dynamics, which require latent nonlinear structure, and which remain unidentifiable from the available perturbation data.
