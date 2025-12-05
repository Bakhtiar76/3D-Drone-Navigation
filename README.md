# AI-Enhanced 3D Reconstruction from 2D Drone Imagery for Autonomous Drone Navigation in Complex Environments

This repository implements a hybrid pipeline that combines **reconstructs accurate 3D scenes from 2D drone imagery** using **Instant-NGP** and enables **safe autonomous navigation** in complex environments via **Reinforcement Learning-based differentiable trajectory optimization**.

The entire system runs in a **Docker-based Linux environment** with full GPU acceleration, ensuring reproducible setup even with challenging dependencies like `tiny-cuda-nn`.

---

## Dataset Downloads

Download the datasets and place them inside a `data/nerf_synthetic` folder at the root of the repository.

### 1. LEGO Dataset (for 3D reconstruction testing)
Link: (https://drive.google.com/drive/folders/1cK3UDIJqKAAm7zyrxRYVFJ0BRMgrwhh4)  
Place in:  
`data/nerf_synthetic/lego/`

### 2. Stonehenge Synthetic Dataset (for navigation tasks)
Link: (https://drive.google.com/drive/folders/1yPItq_6C_zqV2WyzqMZdMcJnXXRODYx3)  
Place in:  
`data/nerf_synthetic/stonehenge/`

---

## Quick Start with Docker

Instant-NGP depends on `tiny-cuda-nn`, which only builds reliably on Linux. We provide a fully configured Dockerfile.

```bash
# Clone the repository
git clone https://github.com/Bakhtiar76/3D-Drone-Navigation.git
cd 3D-Drone-Navigation

# Build the Docker image
docker build -t drone-nerf:latest .

# Run container with GPU access and mount current directory
docker run -it --gpus all \
    -v $(pwd):/workspace \
    -w /workspace \
    drone-nerf:latest

# Inside the container, activate the environment
conda activate drone_nerf
```
---

## Running the Pipeline
A. Train Instant-NGP on LEGO Dataset
```bash
python scripts/train_nerf.py \
    --data ./data/nerf_synthetic/lego \
    --save_model ./results/lego_ngp.pt
```
B. Train Stonehenge Scene & Export Density Field
```bash
python scripts/train_nerf.py \
    --data ./data/nerf_synthetic/stonehenge \
    --export_density ./results/density.npy
```
C. RL-Based Trajectory Optimization
```bash
python scripts/optimize_path.py \
    --density ./results/density.npy \
    --save_path ./results/path.npy
```

## Blender Simulation Setup
This project uses Blender as a photorealistic simulation environment.

*   Install the latest version of Blender
*   Ensure blender is available in your terminal/PATH
*   Your scene must contain a Camera object


### Run Simulation & Visualization
```bash
# Create cache folder if needed
mkdir -p sim_img_cache

python simulate.py data/nerf_synthetic/stonehenge \
    --workspace stonehenge_nerf \
    -O --bound 0 --scale 1.0 --dt_gamma 0
```
viz_func.py will render observations from optimized poses, which simulate.py uses for pose estimation and closed-loop testing.

---

## System Requirements

* NVIDIA GPU (tested on RTX 3060 / 3080 / 4090)
* Docker with NVIDIA Container Toolkit (--gpus all support)
* CUDA 11.7+ and compatible drivers
