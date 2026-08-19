# SigeSC — Synthetic Population Generation and Georeferenced Household Allocation

Code and reproduction materials for the paper:

> **Synthetic Population Generation and Georeferenced Household Allocation via Iterative Proportional Fitting and Integer Programming**
> Maria Eduarda Pinheiro, João Vitor Pamplona, Vitória Caroline Blau, Douglas Soares Gonçalves, Luiz-Rafael Santos, Hugo Jose Lara Urdaneta

## Overview

This repository implements a two-stage mathematical framework for generating georeferenced synthetic populations from publicly available Brazilian census data:

1. **Family Composition** (Section 2): Iterative Proportional Fitting (IPF) combined with integer rounding via MILP to synthesize households that preserve intra-household demographic correlations from census microdata.

2. **Household-to-Residence Allocation** (Section 3): A MILP formulated as a generalized assignment problem that maximizes a composite objective balancing allocation volume, housing-quality matching, and adherence to census tract targets.

The framework is applied to the state of Santa Catarina, Brazil (≈8.4 million inhabitants, 295 municipalities).

## Repository Structure

```
 SigeSC/
├── config.jl                              # Central path configuration
├── README.md
├── data/
│   ├── README.md                          # IBGE download instructions + microdata filtering
│   └── municipios_SC_IBGE2022.csv         # Municipality code list (295 entries)
│
├── src/
│   ├── utils.jl                           # Shared utilities (quality function, demographic types)
│   ├── 05_ipf_member_allocation.jl        # IPF + MILP rounding (Sections 2.2–2.3)
│   └── 10_allocation_model.jl             # Allocation MILP (Section 3)
│
├── analysis/
│   ├── allocation_quality.jl              # Allocation quality and violations (Figures 4–5)
│   └── family_composition_quality.jl      # Intra-household correlations (Figures 1–3)
│
└── output/                                # Generated datasets (see GitHub Release v1.0)
    ├── stage2_attributes/
    ├── stage3_population/
    ├── stage4_update_2025/
    └── results/
```

**Scope.** This repository contains the optimization models and validation scripts. Data preparation scripts (residence registry processing, population projection to 2025, census tract matching) are available upon request to the corresponding author.

## Requirements

- **Julia** ≥ 1.10
- **Packages**:

```julia
using Pkg
Pkg.add(["CSV", "DataFrames", "StatsBase", "JuMP", "HiGHS",
         "Random", "LinearAlgebra", "CairoMakie", "Plots",
         "StatsPlots"])
```

All optimization is performed with [HiGHS](https://highs.dev/) (open source, MIT license).

## Data

### Generated datasets (GitHub Release)

The pipeline-generated datasets required to run the code are available as zip files in the [GitHub Release (`v1.0`)](../../releases/tag/v1.0). Download and extract them into `output/` following this structure:

| File | Size (zip) | Used by |
|------|-----------|---------|
| `dataset_familias_2025.csv` | 10 MB | Allocation model |
| `residencias_2025.csv` | 45 MB | Allocation model |
| `dataset_pessoas_com_Ordem_final.csv` | 48 MB | Family composition validation (Figs. 1–3) |
| `correspondencia_setores.csv` | 176 KB | Census tract gender marginals for constraint (P2d) |
| `family_and_residences.csv`| 18 MB | Final allocation |

After downloading and extracting:

```
output/
├── stage2_attributes/
│   └── correspondencia_setores.csv
├── stage3_population/
│   ├── dataset_familias_2025.csv
│   └── dataset_pessoas_com_Ordem_final.csv
└── stage4_update_2025/
    └── residencias_2025.csv
```

### IBGE public data

Two additional files are sourced directly from IBGE (see [`data/README.md`](data/README.md) for download instructions):

- **Census aggregates by tract** (`Agregados_por_setores_caracteristicas_domicilio2_SC.csv`) — from the [2022 Census aggregates](https://www.ibge.gov.br/estatisticas/sociais/saude/22827-censo-demografico-2022.html)
- **2010 Census microdata** (`pop_filtrado.csv`) — filtered from the [2010 Census microdata sample](https://www.ibge.gov.br/estatisticas/sociais/populacao/9662-censo-demografico-2010.html); filtering procedure documented in `data/README.md`

## Key Results

- **Allocation rate**: 100% of synthetic families are allocated to georeferenced residences.
- **Demographic fidelity**: The residual unassigned population (33 563 individuals, corresponding to collective-dwelling residents) matches the official census count (33 556) to 99.98%.
- **Computational performance**: The allocation MILP is solved in under 25 seconds for all 295 municipalities, including Florianópolis (≈246 000 variables).
- **Intra-household correlations**: Synthesized households reproduce expected age, gender, and ethnic patterns (spousal age proximity, parent–child age gaps, ethnic homogamy).

## Citation

```bibtex
@article{pinheiro2026synthetic,
  title={Synthetic Population Generation and Georeferenced Household
         Allocation via Iterative Proportional Fitting and Integer Programming},
  author={Pinheiro, Maria Eduarda and Pamplona, Jo{\~a}o Vitor and
          Blau, Vit{\'o}ria Caroline and Gon{\c{c}}alves, Douglas Soares and
          Santos, Luiz-Rafael and Lara~Urdaneta, Hugo Jose},
  year={2026},
  note={Preprint}
}
```

## License

MIT

## Acknowledgements

This research was supported by CNPq under project CNPq/MCTI/FNDCT 444264/2024-8. M.E.P. was funded by CNPq grants 314788/2025-5 and partially by 409837/2025-3. V.C.B. was funded by CNPq grant 198186/2025-8.
