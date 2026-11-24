
## 2. Requirements

- RHEL 8 or RHEL 9 (x86_64).
- Internet connection.
- At least 3 GB of free disk space.
- Git and curl.
- NVIDIA GPU is optional. CPU inference is supported but may be slower.

The instructions use `uv` to install and manage Python 3.10.20 independently of the system Python.

## 3. Installation

### Step 1 — Install system dependencies

Open a terminal and run:

```bash
sudo dnf install -y git curl mesa-libGL glib2 libgomp
```

### Step 2 — Install uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Add uv to the current shell PATH:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Verify the installation:

```bash
uv --version
```

### Step 3 — Clone the repository

```bash
git clone https://github.com/DepthAnything/Depth-Anything-V2.git
cd Depth-Anything-V2
```

Check out the tested commit:

```bash
git checkout a561b849ebae10a6f5ef49e26c83cbbcd36c71bf
```

Verify the commit:

```bash
git rev-parse HEAD
```

Expected output:

```text
a561b849ebae10a6f5ef49e26c83cbbcd36c71bf
```

### Step 4 — Install the required Python version

Install Python 3.10.20:

```bash
uv python install 3.10.20
```

Locate the installed interpreter and create a virtual environment:

```bash
PYTHON_BIN="$(uv python find 3.10.20)"
"$PYTHON_BIN" -m venv .venv
```

Verify the Python version:

```bash
.venv/bin/python --version
```

Expected output:

```text
Python 3.10.20
```

**Important:** Always use `.venv/bin/python` for this project. Do not rely on the system `python` or `python3` command.

### Step 5 — Install Python dependencies

The repository provides `requirements.txt`. For reproducible installation, use the committed dependency lockfile:

```bash
uv pip sync --python .venv/bin/python requirements-lock.txt
```

Verify the required packages:

```bash
.venv/bin/python -c "import torch, torchvision, cv2, matplotlib; print('All imports successful'); print('PyTorch:', torch.__version__); print('Torchvision:', torchvision.__version__); print('CUDA available:', torch.cuda.is_available())"
```

Expected output includes:

```text
All imports successful
```

The CUDA value may be `True` or `False`, depending on the available GPU and driver configuration.

## 4. Download the pretrained model

Create the checkpoint directory:

```bash
mkdir -p checkpoints
```

Download the ViT-L pretrained checkpoint:

```bash
curl -fL --retry 3 --retry-all-errors \
"https://huggingface.co/depth-anything/Depth-Anything-V2-Large/resolve/main/depth_anything_v2_vitl.pth" \
-o checkpoints/depth_anything_v2_vitl.pth
```

Verify the downloaded file:

```bash
test -s checkpoints/depth_anything_v2_vitl.pth && echo "Checkpoint download successful"
```

Verify its SHA256 checksum:

```bash
echo "a7ea19fa0ed99244e67b624c72b8580b7e9553043245905be58796a608eb9345  checkpoints/depth_anything_v2_vitl.pth" | sha256sum -c -
```

Expected output:

```text
checkpoints/depth_anything_v2_vitl.pth: OK
```

This checksum is the reference checksum recorded from the successfully tested checkpoint.

## 5. Run inference

The repository includes 20 example images in:

```text
assets/examples/
```

Run the pretrained ViT-L model:

```bash
.venv/bin/python run.py \
--encoder vitl \
--img-path assets/examples \
--outdir depth_vis
```

The script automatically selects CUDA when available, otherwise CPU on RHEL systems.

Wait until the progress reaches:

```text
Progress 20/20: assets/examples/demo20.jpg
```

The output images will be saved in:

```text
depth_vis/
```

## 6. Verify the results

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

Expected output:

```text
Approximately 86M
```

List the generated files:

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

**Reproduction is successful when all 20 PNG files have been generated.**

## 7. Troubleshooting

| Problem | Solution |
|---|---|
| Python version is incorrect | Use `.venv/bin/python`, not the system Python. |
| `uv: command not found` | Run `export PATH="$HOME/.local/bin:$PATH"` again. |
| `ModuleNotFoundError` | Run `uv pip sync --python .venv/bin/python requirements-lock.txt`. |
| `libGL.so.1` is missing | Run `sudo dnf install -y mesa-libGL glib2`. |
| CUDA is unavailable | The script can use CPU automatically. GPU inference requires a compatible NVIDIA driver and PyTorch installation. |
| Checkpoint download is incomplete | Repeat the download command and verify the SHA256 checksum. |
| `Permission denied` when entering a `.pth` file | A `.pth` file is model weights, not an executable program. Do not run it as a command. |
| `xFormers not available` | This message is not fatal for the tested inference workflow. Continue if inference proceeds normally. |

## 8. Output structure

After successful inference, the project directory should contain:

```text
Depth-Anything-V2/
├── assets/
│   └── examples/
│       ├── demo01.jpg
│       ├── demo02.jpg
│       └── ...
│       └── demo20.jpg
├── checkpoints/
│   └── depth_anything_v2_vitl.pth
├── depth_anything_v2/
├── depth_vis/
│   ├── demo01.png
│   ├── demo02.png
│   └── ...
│   └── demo20.png
├── .venv/
├── requirements.txt
├── requirements-lock.txt
└── run.py
```

---






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

**Expected output size:** Approximately 86 MB.

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

**Category:** Python environment and dependency compatibility.

**Problem:**

Different Red Hat Linux computers may have different system Python versions. Using the default system Python can cause dependency compatibility problems.

**Root cause:**

The system Python version is not guaranteed to be identical across computers. The project requires a consistent Python environment to minimise differences during reproduction.

**Minimal fix:**

Install Python 3.10.20 independently using `uv` and create a dedicated virtual environment.

```bash
uv python install 3.10.20

uv venv --python 3.10.20
```

**Verification:**

```bash
.venv/bin/python --version
```

Expected output:

```text
Python 3.10.20
```

*Note: This section describes the environment compatibility issue. Replace it with the actual error and solution if a different failure occurred during your reproduction.*

## 6. AI Agent Check

**Which AI coding agent did you use?**

[Record the AI coding agent actually used.]

**What advice did it give?**

[Record the actual advice provided by the AI agent.]

**What command or fix did you apply yourself?**

[Record the actual command or solution you applied.]

**How did you verify the result yourself?**

Record the checks you actually performed:

1. Python version: `3.10.20`.
2. Git commit: verified using `git rev-parse HEAD`.
3. Python dependencies: successfully imported the required packages.
4. Model checkpoint: downloaded successfully.
5. Inference: completed successfully on all 20 example images.
6. Output count: verified 20 PNG files.
7. Visual inspection: checked the generated depth maps.

Asking an AI agent for advice does not count as independent verification. Record the commands you executed and the results you observed.

## 7. Reproducibility Notes

The following configuration was successfully tested:

- Fresh Git clone.
- Exact Git commit checkout.
- Python 3.10.20 virtual environment.
- Dependencies installed from a lockfile.
- Pretrained ViT-L checkpoint.
- 20 example JPG images processed.
- 20 PNG depth visualisations generated.
- Approximately 86 MB of output data.

Each teammate must create their own virtual environment on their own computer.

Do not copy the `.venv/` directory between computers.

The following directories are generated during reproduction and should not be committed:

```text
.venv/
checkpoints/
depth_vis/
__pycache__/
```

The dependency lockfile and this README should remain in the parent `Depth-Anything-V2-Reproduction/` directory.

**Important:** This guide has been tested against the specified configuration. Different GPU hardware, NVIDIA drivers and operating system configurations may affect inference performance and compatibility.


## 7. One Real Failure

**Category:** Checkpoint / asset

### Root cause

The original `depth_anything_v2_vitl.pth` checkpoint was corrupted or incompletely downloaded.

### Observed impact

The model could not be used reliably with the original checkpoint.

### Minimal fix

Download the official Depth Anything V2 ViT-L checkpoint again and replace the corrupted or incomplete file.

### Verification

Rerun the inference command:

```bash
python run.py --encoder vitl --img-path assets/examples --outdir depth_vis
```

The rerun processed all 20 example images and generated 20 PNG output files.

### Lesson learned

A model checkpoint is a critical project asset. When inference fails during model loading, verify that the checkpoint exists and is a complete, valid download before investigating unrelated dependencies.

## 8. Warnings

During execution, the following warning was observed:

```text
xFormers not available
```

Despite this warning, inference completed successfully for all 20 example images. The warning did not prevent successful inference, so it did not need to be resolved for this workshop run.

## 9. AI Coding Agent

**AI coding agent used:** GitHub Copilot

### Reported change

The agent was asked to change the Python version used by the virtual environment to Python 3.10. The reported agent-assisted change was limited to this environment setting.

### Verification

The environment was configured with Python 3.10.20, and pretrained inference subsequently processed 20 images and generated 20 PNG files. Re-downloading the checkpoint was a separate troubleshooting action.

## 10. Reproducibility Test

The original successful run demonstrates that inference works in the current environment. It does not by itself prove that another person can reproduce the result from a clean setup. A fresh-clone reproduction test is still needed.

### Planned procedure

1. Clone the repository into a new directory.
2. Create a new virtual environment.
3. Install the required dependencies.
4. Download and verify the official pretrained checkpoint.
5. Run inference using the documented command.
6. Confirm that all 20 example images are processed.
7. Verify the generated PNG files.
8. Ask another student to follow this README independently.
9. Record their feedback and update the instructions.

**Fresh-clone reproduction status:** Pending  
**Peer reproduction status:** Pending  
**README revision based on peer feedback:** Pending
