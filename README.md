# 🛡️ Cybersecurity Anomaly Detection Showcase

This repository contains a high-performance deep learning pipeline designed to detect system anomalies using PyTorch. The project is specifically optimized for **Arch Linux** environments and **NVIDIA GPUs**, leveraging the `uv` package manager for stable and reproducible environment management.

This pipeline is a good example of cliff problems on parameters. You can read Report.md to get an overview.

---

## 🚀 Performance Optimizations

To overcome common CPU thermal bottlenecks and maximize training throughput on mobile i5/i7 workstations, this showcase implements several advanced strategies:

* **All-in-VRAM Loading**: The entire dataset is pre-loaded onto the GPU memory to eliminate PCIe bus latency and CPU-to-GPU transfer overhead during training.
* **Zero-Worker Data Pipeline**: By using `num_workers=0`, we bypass the common `CUDA IPC` initialization errors and reduce the heat-generating overhead of Python multiprocessing.
* **High Saturation Batching**: Utilizes a large batch size (e.g., 2048) to ensure high utilization of NVIDIA CUDA cores even with relatively shallow anomaly detection architectures.

---

## 🛠️ System Requirements

* **OS**: Arch Linux
* **GPU**: NVIDIA (CUDA 13.0+ recommended)
* **Drivers**: Latest `nvidia-utils`
* **Python**: 3.12 (managed via `uv`)

---

## 💻 Installation & Setup
Just clone the repository then do the following 

I recommend using [`uv`](https://github.com/astral-sh/uv) to manage dependencies and avoid environment pollution.

### 1. Initialize the Environment
```bash
# Set Python version to 3.12
uv python pin 3.12

# Sync dependencies using the PyTorch CUDA 13.0 index
# High timeout and single downloads are used for stability on slow connections
UV_HTTP_TIMEOUT=600 UV_CONCURRENT_DOWNLOADS=1 uv sync

# If you want jupyter inside venv then (Optional)
UV_HTTP_TIMEOUT=600 UV_CONCURRENT_DOWNLOADS=1 uv sync --extra notebook
```

### 2. Configure JupyterLab (Optional)
To use the `uv` environment within JupyterLab without manual activation, you must register a dedicated kernel. This ensures the global JupyterLab instance points directly to your project's local binaries.

```bash
# Register the kernel using the absolute path of the venv
./.venv/bin/python -m ipykernel install --user --name=anomaly-env --display-name "Python (Anomaly Project)"

# Launch JupyterLab then choose kernel you just created
jupyter-lab notebook.ipynb

# Launch JupyterLab using uv run(if you used sync with "--extra notebook")
uv run jupyter-lab notebook.ipynb
```

### 3. Just run all the cells
It should handle the case if you have cuda or not.

There are many manual checking codes you can play around as you like.

!!! You should not care about comments so much, since as I edited things, comments prepared by ai stopped being consistent !!!
