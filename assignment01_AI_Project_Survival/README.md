# Assignment 1: AI Project Survival — Depth Anything V2

## Project

- **Repository URL:** https://github.com/DepthAnything/Depth-Anything-V2.git
- **Exact commit:** `a561b849ebae10a6f5ef49e26c83cbbcd36c71bf`
- **Inference task:** Monocular depth estimation (one RGB image in, a depth map image out)

## Repo map

- **Environment:** `requirements.txt` in the repository root (no `pyproject.toml`). Python and package versions are not pinned. I used Python 3.10.
- **Entry point:** `run.py`
- **Model:** Depth Anything V2 Small (`vits`), checkpoint `depth_anything_v2_vits.pth` (about 95 MB), not included in the repository and downloaded manually into `checkpoints/`
- **Input:** the 20 images in `assets/examples/` (`evidence/input.jpg` is `demo01.jpg`)
- **Output:** `depth_vis/*.png` (`evidence/output.png` is `demo01.png`)

## Environment setup

Create a dedicated folder and clone a fresh copy:

```bash
cd ~/Desktop

mkdir -p Depth-Anything-V2-Reproduction

cd Depth-Anything-V2-Reproduction

git clone https://github.com/DepthAnything/Depth-Anything-V2.git

cd Depth-Anything-V2

git checkout a561b849ebae10a6f5ef49e26c83cbbcd36c71bf

git rev-parse HEAD
```

`git rev-parse HEAD` must print `a561b849ebae10a6f5ef49e26c83cbbcd36c71bf`.

Create and activate the virtual environment:

```bash
uv python install 3.10

uv venv --python 3.10

source .venv/bin/activate

which python

python --version
```

`which python` must point to the `.venv/bin/python` inside this repository. `python --version` must print a Python 3.10 version (mine: `3.10.20`).

Install the dependencies:

```bash
uv pip install -r requirements.txt
```

Versions installed on my computer (requirements are not pinned): `torch 2.14.1`, `torchvision 0.29.2`, `numpy 2.2.6`, `opencv-python 5.1.0.95`.

## Inference

Download the pretrained checkpoint:

```bash
mkdir -p checkpoints

wget -O checkpoints/depth_anything_v2_vits.pth "https://huggingface.co/depth-anything/Depth-Anything-V2-Small/resolve/main/depth_anything_v2_vits.pth?download=true"

ls -lh checkpoints
```

The file should be about 95 MB.

Run the pretrained model:

```bash
python run.py --encoder vits --img-path assets/examples --outdir depth_vis
```

The script prints `Progress 1/20` to `Progress 20/20`. The `xFormers not available` warning is harmless.

Check the output:

```bash
ls depth_vis
```

Expected: `demo01.png` to `demo20.png` (20 files). Open `depth_vis/demo01.png` and compare it with `evidence/output.png` (input: `assets/examples/demo01.jpg` and `evidence/input.jpg`). Evidence folder: https://github.com/Xinnuo0/MLMPworkshop/tree/main/assignment01_AI_Project_Survival/evidence


## One real failure

**Category:** CHECKPOINT / ASSET

**Root cause:** The repository does not include pretrained weights. `run.py` builds the path `checkpoints/depth_anything_v2_{encoder}.pth` and tries to load it, but the `checkpoints/` folder and the file did not exist. The traceback reached line 36 of `run.py`, which showed that the Python version, dependencies and command were fine.

**Minimal fix:** Create `checkpoints/`, download the Small weights (`depth_anything_v2_vits.pth`, 95 MB) into it, and run with `--encoder vits` so the requested file name matches the downloaded file. No additional Python packages were installed.


## AI agent check

**Which AI coding agent did you use?:** I used Github Copilot in VS Code, and also tried Codex for the initial promots. I also used Claude.

**What advice did it give?:** Since neither the README nor `requirements.txt` specifies a version, Copilot suggested using Python 3.10. The 3.10 is a conservative choice for PyTorch, torchvision, OpenCV and Gradio. For the inference command, Copilot suggested the Large model (`vitl`) with `depth_anything_v2_vitl.pth`. Codex suggested `python -m pip install -r requirements.txt` for installation.
I also asked Claude for advice and decided to use the smaller model (`vits`).

**What command or fix did you apply yourself?:** I created the environment myself using `uv venv --python 3.10`, then installed the dependencies with `uv pip install -r requirements.txt`, as the virtual environment created by `uv venv` does not include pip by default. I decided not to use the large model recommended by Copilot. Since this task only required one successful inference run and the larger model files take up considerably more space, I decided to ask Claude for advice and chose the smalle model (`vits`) instead. I manually downloaded the small checkpoint and change the commands to `--encoder vits`. 

**How did you verify the result yourself?:** I ran `python --version`, which returned 3.10.20, and used `which python` to confirm that Python was running the repository's `.venv` directory. I checked `ls -lh checkpoints`, which showed a file size of 95 MB. After running the inference command, the terminal showed progress 20/20 and `ls depth_vis` listed 20 PNG files. Then I opened `evidence/output.png` and checked that the object outlines in the output image matched those in the input image. 
