
## Overview

This models a disease outbreak in the city of Metropolis as a weighted contact
graph, where each node represents a resident and each edge represents a contact relationship
with an associated daily transmission probability. Using this model, you will implement
efficient graph representations, compute infection risk across a planning horizon, and design
a dynamic programming strategy to allocate a limited supply of antiviral doses as effectively
as possible.

---
```

---

## File Structure

```
disease_spread/
    graph/
        graph.py                  # Abstract base class — DO NOT EDIT
        vertex.py                 # Base Vertex class — DO NOT EDIT
        edge.py                   # Base Edge class — DO NOT EDIT
        adjacency_list.py         # Adjacency list (reference) — DO NOT EDIT
        adjacency_matrix.py       
    simulation/
        person.py                 # Person class (extends Vertex) — DO NOT EDIT
        city.py                   # Builds the contact graph from config — DO NOT EDIT
    transmission/
        monte_carlo.py            # Monte Carlo baseline (reference) — DO NOT EDIT
        task_b.py                 
    treatment/
        vaccination_program.py    # Brute-force baseline (reference) — DO NOT EDIT
        task_d.py                 
    utils/
        config_validator.py       # Config validation — DO NOT EDIT
        simulation_utils.py       # Simulation helpers — DO NOT EDIT
        timer.py                  # Timer utility — DO NOT EDIT
        visualise.py              # Visualisation — DO NOT EDIT
    visuals/                      # Generated PDF visualisations saved here
    simulate_outbreak.py          # Main entry point — DO NOT EDIT
    task_c_analysis.py            
    example_config.json           # Example configuration file
```

---

## Requirements

This project requires Python 3.13+ and a single external library:

```
matplotlib
```

Install with pip:
```bash
pip install matplotlib
```

Or with conda:
```bash
conda install matplotlib
```

---

## Running the Program

Copy `example_config.json` to `config.json` (or any name you like), edit as needed, then run:

```bash
python simulate_outbreak.py config.json
```

---

## Configuration

All simulation parameters are controlled via a JSON config file. The following keys are required:

| Key | Type | Description |
|-----|------|-------------|
| `seed` | `int` | Random seed for reproducibility |
| `num_residents` | `int` | Number of residents in the city (`> 0`) |
| `num_edges` | `int` | Exact number of edges to generate — at most `|V|*(|V|-1)/2` |
| `max_transmission_prob` | `float` | Maximum edge weight — in `[0.0002, 1.0]` |
| `vulnerability_range` | `[float, float]` | Min and max vulnerability values for residents |
| `dosage_range` | `[int, int]` | Min and max antiviral dosage requirements for residents |
| `graph_type` | `str` | Graph representation — `"list"` or `"matrix"` |
| `risk_solver` | `str` | Risk solver — `"monte_carlo"` or `"task_b"` |
| `time_horizon` | `int` | Planning horizon T in days (`> 0`) |
| `simulations` | `int` | Number of Monte Carlo simulations (`> 0`) |
| `total_doses` | `int` | Total antiviral doses available (`> 0`) |
| `vaccine_strategy` | `str` | Vaccine allocation strategy — `"brute_force"` or `"task_d"` |
| `run_vaccine` | `bool` | Whether to run the vaccine program (set `false` for Task C timing) |
| `print_struct` | `bool` | Print the graph structure to the console |
| `visualise` | `bool` | Generate and save a visualisation PDF |
| `visual_filename` | `str` | Filename (without extension) for the saved visualisation |

### Example Config

```json
{
    "seed": 42,
    "num_residents": 15,
    "num_edges": 30,
    "max_transmission_prob": 1.0,
    "vulnerability_range": [0.1, 1.0],
    "dosage_range": [1, 5],
    "graph_type": "list",
    "risk_solver": "monte_carlo",
    "time_horizon": 30,
    "simulations": 100,
    "total_doses": 20,
    "vaccine_strategy": "brute_force",
    "run_vaccine": true,
    "print_struct": false,
    "visualise": true,
    "visual_filename": "outbreak_visual"
}
```

---

## The Disease Model

The contact network is modelled as a weighted undirected graph G = (V, E) where:

- Each vertex V_i represents a resident of Metropolis with:
  - `state` — infected (`True`) or healthy (`False`)
  - `vulnerability` — float in the range defined by `vulnerability_range`
  - `dosage_requirement` — int in the range defined by `dosage_range`

- Each edge (V_i, V_j) carries a weight w_ij ∈ (0, `max_transmission_prob`] representing
  the daily transmission probability between residents V_i and V_j.

- Patient zero is assigned randomly using the seed.

### Infection Risk

The infection risk `r[i][t]` for resident V_i at day t is computed using the recurrence:

```
r[i][0] = 1.0  if V_i is patient zero, else 0.0

r[i][t] = 1 - (1 - r[i][t-1]) * product over Vj in N(Vi) of (1 - r[j][t-1] * w_ij)
```

The final column `r[i][T]` gives each resident's infection risk score, used directly
as their benefit value in the vaccine allocation program.

---

## Visualisation

If `visualise` is set to `true`, a PDF is saved to the `visuals/` folder. The layout
adapts based on what has been computed:

| Row | Left panel | Right panel |
|-----|-----------|-------------|
| 1 | Contact network | Graph representation (list or matrix) |
| 2 | Infection risk network | Full risk table r_{i,t} heatmap |
| 3 | Contact network after vaccination | Vaccination program summary |

Row 2 appears after the risk solver runs. Row 3 appears after the vaccine program runs.

### Example Output

The following is an example visualisation produced by the simulation with 12 residents,
20 edges, and seed 2:

![Example visualisation](visuals/example_output.png)

---
