# CytokineQSP-Latent

CytokineQSP-Latent combines mechanistic quantitative systems pharmacology (QSP) models with latent neural dynamics to infer cytokine signaling from sparse, heterogeneous perturbation data.

## Primary use case

The primary use case is modeling IL-4/IL-13-driven airway epithelial remodeling in type 2 asthma. The project links early cytokine-induced signaling and transcriptional responses to slower epithelial state transitions while retaining interpretable mechanistic structure and explicit uncertainty in unobserved signaling layers.

## Project phases

1. **Phase 1: early response** using GSE3183 to develop and externally constrain a compact hybrid dynamical model of the early IL-13 response.
2. **Phase 2: chronic remodeling** using GSE240741 to extend the model with slow remodeling, observation, and donor-population layers.

See [project_plan.md](project_plan.md) for the scientific plan and validation strategy.

## Repository layout

- `configs/`: phase-specific and model-specific configurations
- `data/`: raw, intermediate, and processed data locations
- `notebooks/`: numbered analyses for both phases
- `src/cytokineqsp_latent/`: reusable data, modeling, inference, simulation, analysis, and plotting code
- `tests/`: unit and integration tests
- `results/`: reproducible phase-specific outputs

Raw datasets and generated results are not committed. Each tracked data directory contains only a placeholder until populated by the corresponding workflow.
