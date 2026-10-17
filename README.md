# Planta Física: Treated-Water Allocation with a Genetic Algorithm

Final project for **MA2015.202 (Bio-inspired)**. We use a genetic algorithm (GA) to decide how much treated water to send to four campus buildings so that they need as little potable water as possible, without going over the treatment plant's capacity.

**Team**
- Ángel Nicolás Landa Tapia (A01668133)
- Luis Fernando Salas López (A01770922)
- Claudio Ahuatzi Quintana (A01661936)

## Problem

The model uses aggregate water demand for March 2025 in four buildings, always in this order:

| Building   | Demand (m³) | Gene domain       |
|------------|------------:|-------------------|
| AULAS 1    | 847.00      | k₁ ∈ {0, …, 8}    |
| BIBLIOTECA | 462.00      | k₂ ∈ {0, …, 8}    |
| CEDETEC    | 284.21      | k₃ ∈ {0, …, 6}    |
| AULAS 2    | 683.00      | k₄ ∈ {0, …, 8}    |

A chromosome `k = (k₁, k₂, k₃, k₄)` is a list of integers. Each gene is a reuse fraction `xᵢ = kᵢ / 10`, so a gene of 6 means 60% treated water. This gives 5,103 possible combinations.

- **Treated water:** `T = Σ Dᵢ · kᵢ / 10`
- **Potable water:** `P = 2276.21 − T`
- **Excess over capacity:** `E = max(0, T − 1410)`

The goal is to minimize `P` while keeping `T ≤ 1410 m³`. The 1,410 m³ figure is a teaching capacity set by the instructor, not a capacity confirmed by the plant.

**Fitness (lower is better):**
- Feasible (`E ≤ 1e-8`): `F = P / 2276.21`, which is always ≤ 1
- Infeasible: `F = 1 + E / 1410`, which is always > 1

## Genetic algorithm

| Component       | Implementation                                                        |
|-----------------|-----------------------------------------------------------------------|
| Initialization  | Each gene is sampled uniformly within its domain                      |
| Selection       | Binary tournament: the lower `F` of two distinct individuals wins     |
| Crossover       | One-point crossover; each building keeps its position                 |
| Mutation        | Each gene, with probability `p`, moves up or down one valid level     |
| Replacement     | Generational, with one elite individual kept unchanged                |

Baseline configuration: 20 individuals, 20 generations, `p = 0.25`, seed 42. The comparison run changes only the mutation probability to `p = 0.50`.

**Development variant:** the mutation operator is replaced by a capacity-aware mutation (`mutate_capacity_aware`), which allows an upward step only when the plan stays within capacity. The baseline and the variant are compared under the same configuration with seed 42.

## Repository contents

| File | Description |
|------|-------------|
| [`planta_fisica_operator_variant.ipynb`](planta_fisica_operator_variant.ipynb) | **Cumulative notebook (Development submission).** All previous sections unchanged, followed by Sections 14–21: the limitation of the baseline mutation, the capacity-aware mutation variant, the seed-42 comparison against the baseline, a 100-seed scope check and reproducibility checks (B1/B2). |
| [`planta_fisica_minimum_solution.ipynb`](planta_fisica_minimum_solution.ipynb) | The supplied baseline GA: evaluator, operators, the 0.25 vs 0.50 mutation comparison, a convergence plot and reproducibility checks. Also includes our explanation of one generation, our interpretation of the comparison, an exhaustive check of all 5,103 plans, the model limitations and our check of the hand calculations. |
| [`Planta_Física_Challenge_DEL_1.ipynb`](Planta_Física_Challenge_DEL_1.ipynb) | Our Research deliverable: A1 (formulation and assumptions) and A2 (hand calculation of `(6,6,6,6)` and `(8,8,6,8)`, compared against the evaluator). |
| `pyproject.toml`, `uv.lock` | Project dependencies, managed with [uv](https://docs.astral.sh/uv/). |

### Hand-calculated reference results

| Candidate   | T (m³)   | P (m³)  | E (m³)  | Feasible | F      |
|-------------|---------:|--------:|--------:|:--------:|-------:|
| (6,6,6,6)   | 1365.726 | 910.484 | 0       | Yes      | 0.4    |
| (8,8,6,8)   | 1764.126 | 512.084 | 354.126 | No       | 1.2512 |

## Setup

Requires Python ≥ 3.13 and uv.

```bash
uv sync --locked
uv run jupyter lab
```

Restart the kernel and run each notebook from top to bottom. With the fixed seed, the notebooks produce the same results every run.

## Limitations

This is a teaching model. It uses one month of aggregate demand, and the capacity and per-building reuse limits are assumptions. Its results are not a validated hydraulic or sanitary recommendation.
