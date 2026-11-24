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


## 6. AI Agent Check

**Which AI coding agent did you use?:** GitHub Copilot

**What advice did it give?:** 

**What command or fix did you apply yourself?:**

**How did you verify the result yourself?:**



# Assignment 1: AI Project Survival

## Project

- **Repository URL:** https://github.com/DepthAnything/Depth-Anything-V2.git
- **Exact commit:** `a561b849ebae10a6f5ef49e26c83cbbcd36c71bf`
- **Inference task:** Monocular depth estimation (one RGB image in, a depth map image out)

## Repo map

- **Environment:** `requirements.txt` in the repository root (no `pyproject.toml`). The README and `requirements.txt` do not pin a Python version or package versions. I used Python 3.10.
- **Entry point:** `run.py`
- **Model:** Depth Anything V2 Small (`vits`), checkpoint `depth_anything_v2_vits.pth` (about 95 MB). It is not included in the repository and must be downloaded manually into `checkpoints/`.
- **Input:** `assets/examples/demo01.jpg` (copied to `evidence/input.jpg`)
- **Output:** `depth_vis/demo01.png` (copied to `evidence/output.png`)

## Environment setup

Work in a separate folder from this submission repository (`~/workshop02_projects/`). Replace nothing: the commit SHA below is the exact version I used.

```bash
mkdir -p ~/workshop02_projects
cd ~/workshop02_projects
git clone https://github.com/DepthAnything/Depth-Anything-V2.git Depth-Anything-V2_fresh
cd Depth-Anything-V2_fresh
git checkout a561b849ebae10a6f5ef49e26c83cbbcd36c71bf
uv python install 3.10
uv venv --python 3.10
source .venv/bin/activate
uv pip install -r requirements.txt
```

Checks after activation: `which python` should point to `.venv/bin/python` inside the cloned repository, and `python --version` should print `Python 3.10.x` (I got 3.10.20).

Download the pretrained checkpoint (it is not in the repository):

```bash
mkdir -p checkpoints
wget -O checkpoints/depth_anything_v2_vits.pth "https://huggingface.co/depth-anything/Depth-Anything-V2-Small/resolve/main/depth_anything_v2_vits.pth?download=true"
ls -lh checkpoints
```

The file should be about 95 MB. A size of 0 or a few KB means the download failed.

## Inference

Run the pretrained Small model on the example images (the folder contains 20 images; the script processes all of them):

```bash
python run.py --encoder vits --img-path assets/examples --outdir depth_vis
```

The `xFormers not available` warning is harmless. Outputs are saved to `depth_vis/` as `demo01.png` to `demo20.png`.

Copy one input/output pair into this repository as evidence (same image, `demo01`):

```bash
cp assets/examples/demo01.jpg ~/MLMPworkshop/assignment01_AI_Project_Survival/evidence/input.jpg
cp depth_vis/demo01.png ~/MLMPworkshop/assignment01_AI_Project_Survival/evidence/output.png
```

## One real failure

- **Command:** `python run.py --encoder vitl --img-path assets/examples --outdir depth_vis`
- **Error:** `FileNotFoundError: [Errno 2] No such file or directory: 'checkpoints/depth_anything_v2_vitl.pth'`
- **Category:** CHECKPOINT / ASSET
- **Root cause:** The repository does not include pretrained weights. `run.py` builds the path `checkpoints/depth_anything_v2_{encoder}.pth` and tries to load it, but the `checkpoints/` folder and file did not exist. The traceback reached line 36 of `run.py`, which showed that the Python version, dependencies and command were fine.
- **Minimal fix:** Create `checkpoints/`, download the Small weights (`depth_anything_v2_vits.pth`, 95 MB) into it, and run with `--encoder vits` so the requested file name matches the downloaded file. No additional Python packages were installed.
- **Side note:** My first download attempt returned `404 Not Found` because I had typed backslashes inside the quoted URL (they appeared as `%5C` in the request). It also left a 0-byte file, which I checked with `ls -lh checkpoints`, deleted with `rm`, and then downloaded again with the correct URL.

## AI agent check

- **Which AI coding agent did you use?:** GitHub Copilot in VS Code (Codex was also tried for the first prompts). I also used Claude in a chat window to explain commands and the debugging steps.
- **What advice did it give?:** Python 3.10 was recommended because neither the README nor `requirements.txt` specifies a version, and 3.10 is a conservative choice for PyTorch, torchvision, OpenCV and Gradio. For the inference command, Copilot suggested the Large model (`vitl`) with `depth_anything_v2_vitl.pth`. Codex suggested `python -m pip install -r requirements.txt` for installation.
- **What command or fix did you apply yourself?:** I created the environment myself with `uv venv --python 3.10`, installed with `uv pip install -r requirements.txt` (as the course instructs, because a `uv venv` environment does not include pip by default), downloaded the Small checkpoint manually, and changed the command to `--encoder vits`.
- **How did you verify the result yourself?:** I ran `python --version` (3.10.20) and `which python` (path inside the repository `.venv`). I checked `ls -lh checkpoints` (95M). After running the inference command, the terminal showed progress 20/20 and `ls depth_vis` listed 20 PNG files. I opened `evidence/output.png` next to `evidence/input.jpg` and [describe what you saw: e.g. nearer objects appear brighter and the object outlines match the input image].
