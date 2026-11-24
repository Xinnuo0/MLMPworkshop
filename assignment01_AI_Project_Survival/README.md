# Assignment 1: AI Project Survival

## Project

**Repository URL:** https://github.com/DepthAnything/Depth-Anything-V2.git

**Exact commit:** `a561b849ebae10a6f5ef49e26c83cbbcd36c71bf`

**Inference task:** Monocular depth estimation (one RGB image in, a depth map image out)

## Repo map

**Environment:** `requirements.txt` in the repository root (no `pyproject.toml`). The README and `requirements.txt` do not pin a Python version or package versions. I used Python 3.10.

**Entry point:** `run.py`

**Model:** Depth Anything V2 Small (`vits`), checkpoint `depth_anything_v2_vits.pth` (about 95 MB). It is not included in the repository and must be downloaded manually into `checkpoints/`.

**Input:** `assets/examples/demo01.jpg` (copied to `evidence/input.jpg`)

**Output:** `depth_vis/demo01.png` (copied to `evidence/output.png`)

## Environment setup

Work in a separate folder from this submission repository (`~/workshop02_projects/`). Replace nothing.

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
```bash
mkdir -p Depth-Anything-V2-Reproduction
cd Depth-Anything-V2-Reproduction
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

**Command:** `python run.py --encoder vitl --img-path assets/examples --outdir depth_vis`

**Error:** `FileNotFoundError: [Errno 2] No such file or directory: 'checkpoints/depth_anything_v2_vitl.pth'`

**Category:** CHECKPOINT / ASSET

**Root cause:** The repository does not include pretrained weights. `run.py` builds the path `checkpoints/depth_anything_v2_{encoder}.pth` and tries to load it, but the `checkpoints/` folder and file did not exist. The traceback reached line 36 of `run.py`, which showed that the Python version, dependencies and command were fine.

**Minimal fix:** Create `checkpoints/`, download the Small weights (`depth_anything_v2_vits.pth`, 95 MB) into it, and run with `--encoder vits` so the requested file name matches the downloaded file. No additional Python packages were installed.

**Side note:** My first download attempt returned `404 Not Found` because I had typed backslashes inside the quoted URL (they appeared as `%5C` in the request). It also left a 0-byte file, which I checked with `ls -lh checkpoints`, deleted with `rm`, and then downloaded again with the correct URL.

## AI agent check

**Which AI coding agent did you use?:** GitHub Copilot in VS Code (Codex was also tried for the first prompts). I also used Claude in a chat window to explain commands and the debugging steps.

**What advice did it give?:** Python 3.10 was recommended because neither the README nor `requirements.txt` specifies a version, and 3.10 is a conservative choice for PyTorch, torchvision, OpenCV and Gradio. For the inference command, Copilot suggested the Large model (`vitl`) with `depth_anything_v2_vitl.pth`. Codex suggested `python -m pip install -r requirements.txt` for installation.

**What command or fix did you apply yourself?:** I created the environment myself with `uv venv --python 3.10`, installed with `uv pip install -r requirements.txt` (as the course instructs, because a `uv venv` environment does not include pip by default), downloaded the Small checkpoint manually, and changed the command to `--encoder vits`.

**How did you verify the result yourself?:** I ran `python --version` (3.10.20) and `which python` (path inside the repository `.venv`). I checked `ls -lh checkpoints` (95M). After running the inference command, the terminal showed progress 20/20 and `ls depth_vis` listed 20 PNG files. I opened `evidence/output.png` next to `evidence/input.jpg` and [describe what you saw: e.g. nearer objects appear brighter and the object outlines match the input image].
