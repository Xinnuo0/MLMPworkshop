# Depth Anything V2 — Reproduction Guide

## 1. Project

**Repository URL:**  
https://github.com/DepthAnything/Depth-Anything-V2

**Exact commit:**  
`a561b849ebae10a6f5ef49e26c83cbbcd36c71bf`

**Inference task:** Monocular Depth Estimation

**Model:** Depth Anything V2 ViT-L

**Python version:** 3.10.20

**Input:** 20 example JPG images from `assets/examples/`

**Output:** 20 PNG depth visualisation images in `depth_vis/`

## 2. Repo Map

| Component | Location / Description |
|---|---|
| Environment | `requirements.txt` and `requirements-lock.txt` |
| Entry point | `run.py` |
| Model | Pretrained ViT-L checkpoint |
| Input | `assets/examples/` |
| Output | `depth_vis/*.png` |

The official repository contains the model implementation, example images and inference script.

## 3. Environment Setup

This guide is designed for Red Hat Enterprise Linux (RHEL) 8 or 9, x86_64.

It creates a clean, independent reproduction environment. It does not modify the system Python installation.

### Step 1 — Create a dedicated reproduction folder

Open a terminal and run:

```bash
cd ~/Desktop

mkdir -p Depth-Anything-V2-Reproduction

cd Depth-Anything-V2-Reproduction
```

### Step 2 — Clone the official repository

```bash
git clone https://github.com/DepthAnything/Depth-Anything-V2.git

cd Depth-Anything-V2

git checkout a561b849ebae10a6f5ef49e26c83cbbcd36c71bf
```

Verify the commit:

```bash
git rev-parse HEAD
```

The output must match:

```text
a561b849ebae10a6f5ef49e26c83cbbcd36c71bf
```

### Step 3 — Install system dependencies

```bash
sudo dnf install -y git curl mesa-libGL glib2 libgomp
```

### Step 4 — Install uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh

export PATH="$HOME/.local/bin:$PATH"

uv --version
```

### Step 5 — Install Python 3.10.20

Install the specified Python version:

```bash
uv python install 3.10.20
```

Create an independent virtual environment:

```bash
uv venv --python 3.10.20
```

Verify the Python version:

```bash
.venv/bin/python --version
```

Expected output:

```text
Python 3.10.20
```

### Step 6 — Install Python dependencies

The reproduction folder contains a dependency lockfile generated from the successfully tested environment.

Install the locked dependencies:

```bash
uv pip sync --python .venv/bin/python ../requirements-lock.txt
```

Verify the installation:

```bash
.venv/bin/python -c "import torch, torchvision, cv2, matplotlib; print('All imports successful'); print('PyTorch:', torch.__version__); print('Torchvision:', torchvision.__version__); print('CUDA available:', torch.cuda.is_available())"
```

Expected output should include:

```text
All imports successful
```

CUDA availability depends on the computer's GPU and driver configuration.

## 4. Inference

### Step 1 — Download the pretrained model

Create the checkpoint directory:

```bash
mkdir -p checkpoints
```

Download the ViT-L checkpoint:

```bash
curl -fL --retry 3 --retry-all-errors \
"https://huggingface.co/depth-anything/Depth-Anything-V2-Large/resolve/main/depth_anything_v2_vitl.pth" \
-o checkpoints/depth_anything_v2_vitl.pth
```

Check that the checkpoint exists:

```bash
test -s checkpoints/depth_anything_v2_vitl.pth && echo "Checkpoint download successful"
```

### Step 2 — Run inference

The repository provides 20 example images in:

```text
assets/examples/
```

Run the pretrained model:

```bash
.venv/bin/python run.py \
--encoder vitl \
--img-path assets/examples \
--outdir depth_vis
```

The inference script uses CUDA when available and otherwise runs on CPU, subject to the installed PyTorch configuration.

### Step 3 — Check the output

Count the generated PNG files:

```bash
find depth_vis -type f -iname '*.png' | wc -l
```

Expected output:

```text
20
```

Check the output directory size:

```bash
du -sh depth_vis/
```

The expected size is approximately 86 MB.

Inspect the generated images:

```bash
ls -lh depth_vis/
```

Expected files:

```text
demo01.png
demo02.png
demo03.png
...
demo20.png
```

**Successful reproduction:** All 20 example images have been processed and 20 PNG depth visualisations have been generated.



## 5. One Real Failure

**Category:** Checkpoint / asset

**Root cause:** The original `depth_anything_v2_vitl.pth` checkpoint was corrupted or incompletely downloaded.

**Observed impact:** The model could not be used reliably with the original checkpoint.

**Minimal fix:** Download the official Depth Anything V2 ViT-L checkpoint again and replace the corrupted or incomplete file.

**Verification:** Rerun the inference command:

```bash
python run.py --encoder vitl --img-path assets/examples --outdir depth_vis
```

The rerun processed all 20 example images and generated 20 PNG output files.

### Lesson learned

A model checkpoint is a critical project asset. When inference fails during model loading, verify that the checkpoint exists and is a complete, valid download before investigating unrelated dependencies.

## 6. Warnings

During execution, the following warning was observed:

```text
xFormers not available
```

Despite this warning, inference completed successfully for all 20 example images. The warning did not prevent successful inference, so it did not need to be resolved for this workshop run.

## 7. AI Agent Check

**Which AI coding agent did you use?:** GitHub Copilot

**What advice did it give?:** 

**What command or fix did you apply yourself?:**

**How did you verify the result yourself?:**
