# CASIO: Calibrated Surrogate-Informed Optimization for Shear Wall Structural Design

Official implementation of **CASIO** (*Calibrated Surrogate-Informed Optimization*), a surrogate-informed framework for FEA-efficient section sizing and material selection in high-rise shear wall structures.

## Overview

CASIO accelerates shear wall sizing for a prescribed structural layout, subject to the design checks implemented in this repository. It has three core components:

1. **Layout-Conditioned Global Surrogates (`FuseGNN`)**: Two independently trained graph-tabular networks (`LayoutParamGNN` in the codebase) combine room-adjacency graphs, global layout statistics, and design parameters to predict feasibility and reinforcement mass before finite element analysis (FEA).
2. **Online Local Calibration**: During optimization, FEA results from the target building update local calibrators without retraining the global networks. The feasibility calibrator uses Platt scaling. The reinforcement calibrator predicts residuals using ridge regression, Gaussian process regression, or LightGBM, depending on the number of available samples.
3. **Decision-Oriented FEA Allocation**: Calibrated predictions guide conservative feasibility screening and feasibility-adjusted cost ranking to prioritize candidates for OpenSees evaluation. The workflow supports GA, PSO, and random search. Final designs are evaluated with FEA and the implemented design checks.

## Repository Structure

```text
CASIO/
├── src/
│   ├── shearwall_modeling/      # Parametric modeling, OpenSees FEA, and reinforcement checks
│   ├── surrogate/              # FuseGNN, feature extraction, and dataset preparation
│   ├── shearwall_optimization/  # Screening, ranking, online calibration, and search algorithms
│   ├── data_engine/             # Shared geometry and graph processing
│   ├── shearwall_pred/          # Shared graph construction helpers
│   └── misc/                    # Common utilities
└── scripts/
    ├── shearwall_analyzer_main.py       # Example driver for single-design FEA and checks
    ├── parametric_dataset.py            # Parametric sampling and batch FEA
    ├── shearwall_optimize_main.py       # CASIO and full-FEA optimization entry point
    └── surrogate/
        ├── prepare_dataset.py           # Convert FEA results to surrogate training records
        ├── build_layout_features.py     # Compute and merge global layout statistics
        ├── make_group_splits.py         # Split records by layout for training, validation, and testing
        ├── train_gnn_final_pass.py      # Train the feasibility classifier
        ├── train_gnn_steel.py           # Train the reinforcement mass regressor
        └── predict_gnn_final_pass.py    # Run and evaluate feasibility classifier inference
```

## Environment & Dependencies

The source project uses **Python 3.12**. This standalone repository has not yet been validated end to end. Install PyTorch for your CPU or CUDA environment, then install the remaining dependencies:

```bash
pip install torch_geometric openseespy numpy pandas pyarrow scikit-learn lightgbm optuna networkx shapely ezdxf matplotlib joblib tqdm
```

## Data & Directory Conventions

Engineering drawings, FEA datasets, and pretrained model artifacts are not included. The inherited defaults use Windows-style paths. On other platforms, pass paths explicitly. Run commands from the repository root.

- `data/dxf/cad_json_data/fem_raw/`: layout JSON files in the format expected by the structural evaluator and room-graph builder.
- `data/dxf/fem_raw/`: matching DXF files for the optional member-graph representation. Room graphs are built from the layout JSON alone.
- `data/parametric/surrogate_dataset/splits/`: Parquet records split by layout.
- `data/parametric/ckpt/`: the optimization CLI's default location for surrogate artifacts. Training writes artifacts under `data/parametric/surrogate_dataset/` by default. Pass the files produced by training explicitly when running optimization.

## Quick Start

Use `python -m <module> --help` from the repository root to inspect the arguments for the commands below. `scripts/shearwall_analyzer_main.py` is an example driver with a preset JSON path and does not implement `--help`.

The following commands show the workflow using the inherited Windows defaults. They require local input data and have not been run end to end in this repository.

```bash
# 1. Generate FEA samples and prepare the training dataset.
python -m scripts.parametric_dataset
python -m scripts.surrogate.prepare_dataset --source-dir outputs/result/parametric_dataset
python -m scripts.surrogate.build_layout_features
python -m scripts.surrogate.make_group_splits

# 2. Train the two surrogates using room graphs.
python -m scripts.surrogate.train_gnn_final_pass --graph-repr room
python -m scripts.surrogate.train_gnn_steel --graph-repr room
```

These commands show the room-graph workflow; the training hyperparameter defaults differ from the paper settings.

To enable CASIO with GA, pass your layout, both trained artifacts, and the room-graph cache generated during training. The cache and layout statistics must contain the target layout. If you stored the layout statistics outside the default location, pass the file with `--ga-surrogate-layout-features` and `--ga-cost-layout-features`. The same surrogate options apply to `pso` and `random` despite the `--ga-` prefix.

```bash
python -m scripts.shearwall_optimize_main --algorithm ga --layout-path path/to/layout.json \
  --ga-surrogate-screen --ga-surrogate-artifact path/to/gnn_final_pass.pt \
  --ga-surrogate-cost-preselect --ga-cost-steel-artifact path/to/gnn_steel.pt \
  --ga-surrogate-graph-cache data/parametric/surrogate_dataset/gnn_room_graph_cache \
  --ga-cost-graph-cache data/parametric/surrogate_dataset/gnn_room_graph_cache \
  --ga-local-calibration
```

For a full-FEA baseline, omit the surrogate and calibration flags.

## Citation

If you use this code or methodology in your research, please cite the manuscript:

```bibtex
@unpublished{Leng2026CASIO,
  title  = {A Calibrated Surrogate-Informed Optimization Framework for FEA-Efficient Shear Wall Structural Design},
  author = {Leng, Hao and Gao, Yuqing and Zhou, Ying and Zhang, Zheng},
  year   = {2026},
  note   = {Manuscript under review}
}
```
