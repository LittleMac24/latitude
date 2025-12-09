# Documentation

## How to run the code

### 1. Install dependencies (Poetry — recommended)

```bash
poetry install
poetry shell
```

### 4. Run the analysis (Jupyter)

```bash
jupyter notebook
```

Open `analysis.ipynb` and run cells top → bottom to reproduce the analysis.

### 5. View the final report (Quarto)

A rendered HTML report is included as `analysis-report.html`. To regenerate from the notebook/Quarto source:

```bash
quarto render analysis-report.ipynb --execute
```
