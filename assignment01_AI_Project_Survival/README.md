# Assignment 1: AI Project Survival — Depth Anything V2 Reproduction Guide

## Project

**Repository URL:** https://github.com/DepthAnything/Depth-Anything-V2.git

**Exact commit:** `a561b849ebae10a6f5ef49e26c83cbbcd36c71bf`

**Inference task:** Monocular depth estimation (one RGB image in, a depth map image out)


## Repo map

**Environment:** `requirements.txt` in the repository root (there is no `pyproject.toml`). Neither the repository README nor `requirements.txt` pins a Python version or package versions. I used Python 3.10.

**Entry point:** `run.py`

**Model:** Depth Anything V2 Small (`vits`), checkpoint `depth_anything_v2_vits.pth` (about 95 MB). It is not included in the repository and must be downloaded manually into `checkpoints/`.

**Input:** the 20 example images in `assets/examples/` (`evidence/input.jpg` is `demo01.jpg`)

**Output:** `depth_vis/*.png` (`evidence/output.png` is `demo01.png`)

## Environment setup

Follow the steps in order. If the folder from Step 1 already exists from an earlier attempt, delete it or use another name so that you can create a clean, independent reproduction environment.

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

The output must be:

```text
a561b849ebae10a6f5ef49e26c83cbbcd36c71bf
```

### Step 3 — Check that uv is installed

```bash
uv --version
```

If this prints a version number, go to Step 4. If the command is not found, install uv (it is installed in your home directory, not system-wide):

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh

export PATH="$HOME/.local/bin:$PATH"

uv --version
```

### Step 4 — Create the virtual environment

```bash
uv python install 3.10

uv venv --python 3.10

source .venv/bin/activate
```

Check where you are and which Python you are using:

```bash
pwd

which python

python --version
```

Expected:

`pwd` ends with `Depth-Anything-V2-Reproduction/Depth-Anything-V2`

`which python` points to the `.venv/bin/python` inside this repository

`python --version` prints `Python 3.10.x` (I got `3.10.20`)

All following commands assume this environment is activated. If you open a new terminal, go back to this folder and run `source .venv/bin/activate` again.

### Step 5 — Install the Python dependencies

```bash
uv pip install -r requirements.txt
```

This takes about one minute on my machine (88 packages installed). Then check the key packages:

```bash
uv pip list | grep -i -E "torch|numpy|opencv"
```

Versions I observed (requirements are not pinned, so newer releases may appear later):

```text
numpy          2.2.6
opencv-python  5.1.0.95
torch          2.14.1
torchvision    0.29.2
```

Check that the imports work:

```bash
python -c "import torch, torchvision, cv2, matplotlib; print('All imports successful'); print('PyTorch:', torch.__version__); print('CUDA available:', torch.cuda.is_available())"
```

Expected output includes `All imports successful`. `CUDA available` may be `True` or `False`; both are fine (the Small model also runs on CPU).


## Inference

### Step 1 — Download the pretrained checkpoint

The checkpoint is not in the repository, so `run.py` fails without it (see "One real failure").

```bash
mkdir -p checkpoints

wget -O checkpoints/depth_anything_v2_vits.pth "https://huggingface.co/depth-anything/Depth-Anything-V2-Small/resolve/main/depth_anything_v2_vits.pth?download=true"
```

Do not add backslashes inside the quoted URL. Check the file size:

```bash
ls -lh checkpoints
```

The file should be about 95 MB. If it is 0 bytes or only a few KB, the download failed: delete it with `rm checkpoints/depth_anything_v2_vits.pth`, check the URL, and download again.

### Step 2 — Run inference

```bash
python run.py --encoder vits --img-path assets/examples --outdir depth_vis
```

`--encoder vits` selects the Small model and must match the checkpoint file name. The script prints `Progress 1/20` to `Progress 20/20`. The `xFormers not available` warning is harmless.

### Step 3 — Check the output

```bash
ls depth_vis

find depth_vis -type f -name '*.png' | wc -l
```

Expected: `demo01.png` to `demo20.png`, and the count is `20`.

### Step 4 — Compare with the evidence

Open `depth_vis/demo01.png` and compare it with `evidence/output.png`, and the original `assets/examples/demo01.jpg` with `evidence/input.jpg`. The result should be a colour-mapped depth image whose shapes match the photo. Tiny pixel differences between machines are acceptable.

**Evidence for comparison:** `evidence/input.jpg` and `evidence/output.png` in this folder (https://github.com/Xinnuo0/MLMPworkshop/tree/main/assignment01_AI_Project_Survival/evidence)

**Successful reproduction:** the command finished without errors, 20 PNG files exist, and `demo01.png` looks like `evidence/output.png`.

## One real failure

**Command:** `python run.py --encoder vitl --img-path assets/examples --outdir depth_vis`

**Error:** `FileNotFoundError: [Errno 2] No such file or directory: 'checkpoints/depth_anything_v2_vitl.pth'`

**Category:** CHECKPOINT / ASSET

**Root cause:** The repository does not include pretrained weights. `run.py` builds the path `checkpoints/depth_anything_v2_{encoder}.pth` and tries to load it, but the `checkpoints/` folder and the file did not exist. The traceback reached line 36 of `run.py`, which showed that the Python version, dependencies and command were fine.

**Minimal fix:** Create `checkpoints/`, download the Small weights (`depth_anything_v2_vits.pth`, 95 MB) into it, and run with `--encoder vits` so the requested file name matches the downloaded file. No additional Python packages were installed.


## AI agent check

**Which AI coding agent did you use?:** I used Github Copilot in VS Code, and also tried Codex for the initial promots.

**What advice did it give?:** Python 3.10 was recommended because neither the README nor `requirements.txt` specifies a version, and 3.10 is a conservative choice for PyTorch, torchvision, OpenCV and Gradio. For the inference command, Codex suggested `python -m pip install -r requirements.txt` for installation.

**What command or fix did you apply yourself?:** I created the environment myself using `uv venv --python 3.10`, then installed the dependencies with `uv pip install -r requirements.txt`, as the virtual environment created by `uv venv` does not include pip by default. I also downloaded the small checkpoint manually and change the relevant commands to use `--encoder vits`. 

**How did you verify the result yourself?:** I ran `python --version`, which returned 3.10.20, and used `which python` to confirm that Python was running the repository's `.venv` directory. I checked `ls -lh checkpoints`, which showed a file size of 95 MB. After running the inference command, the terminal showed progress 20/20 and `ls depth_vis` listed 20 PNG files. I opened `evidence/output.png` alongside `evidence/input.jpg` and checked that the object outlines in the output image matched those in the input image.
