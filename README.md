# CLARITree: Cholesky and Lookahead Accelerations for Regression with Interpretable Piecewise Linear Trees

This repository contains the code and experiment artifacts for the ICML 2026 paper **"CLARITree: Cholesky and Lookahead Accelerations for Regression with Interpretable Piecewise Linear Trees."**

[![CLARITree teaser](https://raw.githubusercontent.com/Yixiao-Wang-Stats/CLARITree/main/results/images/teaser_new.png)](https://github.com/Yixiao-Wang-Stats/CLARITree/blob/main/results/images/teaser_new.pdf)

## Installation

Install the released Python package with:

```bash
pip install claritree
```

The PyPI distribution is named `claritree`; the Python import package remains
`clari_tree`:

```python
from clari_tree import CLARITree
```

To use the dataframe-based example, install the optional example dependencies:

```bash
pip install "claritree[examples]"
```

## Source Requirements

CLARITree requires Python 3.9+, CMake, a C++17 compiler, and Eigen 3.

Clone the repository:

```bash
git clone https://github.com/Yixiao-Wang-Stats/CLARITree.git
cd CLARITree
```

## C++ Demo

Build the C++ executable:

```bash
make
```

Run the built-in demo:

```bash
make test
```

This compiles `run_tree` and runs it on one of the included datasets. You can also call it directly:

```bash
./run_tree data/airfoil/splits/outer_0/train.csv 4.0 0.001 0.001 20 quantile
```

The command-line interface is:

```text
./run_tree <csv_file> <depth> <lambda> <kappa> [n_thresholds] [thresholds_strategy]
```

- `depth`: maximum tree depth.
- `lambda`: tree-complexity penalty.
- `kappa`: ridge penalty for the linear models in the leaves.
- `n_thresholds`: candidate split thresholds per feature.
- `thresholds_strategy`: `quantile` or `uniform`; defaults to `quantile`.

The executable creates a reproducible 80/20 train/test split and reports MSE, R2, and runtime for both Greedy and CLARITree. CSV files may have a header and must otherwise be numeric. The target is read from the final column, except for filenames containing `targetfirst`, where it is read from the first column.

Clean the C++ build output with:

```bash
make clean
```

## Python Build

For local Python development, use the rebuild helper:

```bash
./rebuild_clari_tree.sh
```

The script creates `.venv` if needed, builds the pybind11 extension, installs the package in editable mode, and verifies that the Python module imports correctly. To use a different virtual environment path:

```bash
CLARITREE_VENV=claritree-env ./rebuild_clari_tree.sh
```

After the build, activate the environment before running Python scripts:

```bash
source .venv/bin/activate
```

## Python Quick Start

Run the included Airfoil example from the repository root:

```bash
python scripts/example.py
```

The minimal API is:

```python
import pandas as pd
from clari_tree import CLARITree

data = pd.read_csv("data/airfoil/splits/outer_0/train.csv").to_numpy(float)
X, y = data[:, :-1], data[:, -1]

model = CLARITree(
    depth=4,
    lambda_=0.001,
    kappa=0.001,
    n_thresholds=20,
    thresholds_strategy="quantile",
    min_leaf_node_size=0,
)
model.fit(X, y)
predictions = model.predict(X)
model.print_tree()
```

Input CSV files should contain features followed by the target in the final column. Binary split features should be encoded as `0/1`; categorical features should be one-hot encoded before fitting.


## Repository Layout

- `clari_tree/`: Python package.
- `src/` and `include/`: C++ implementation and Python bindings.
- `scripts/`: experiment, table, and plotting scripts.
- `data/`: datasets and pre-generated train/test splits.
- `results/`: selected result tables and paper figures.

The public plotting entry points are:

```bash
python scripts/images/main_teaser.py
python scripts/images/plot_completion_rate.py
python scripts/images/plot_linear.py
```

## License

CLARITree is released under the [MIT License](LICENSE).
