# Depth Anything V2 — Installation and Reproduction Guide

## 1. Project Overview

**Repository:** [Depth Anything V2](https://github.com/DepthAnything/Depth-Anything-V2)  
**Inference task:** This project uses the pretrained AI model Depth Anything V2 to estimate the relative depth of objects and their surroundings from a single image. The ViT-L model processes sample images and generates depth maps.

### Tested configuration

| Component | Version / Configuration |
|---|---|
| Repository | Depth Anything V2 |
| Git commit | `a561b849ebae10a6f5ef49e26c83cbbcd36c71bf` |
| Python | 3.10.20 |
| PyTorch | 2.14.1+cu130 |
| Torchvision | 0.29.2+cu132 |
| Model | ViT-L |
| Checkpoint | `depth_anything_v2_vitl.pth` |
| Input images | 20 JPG files |
| Expected outputs | 20 PNG files |
| Tested output size | Approximately 86 MB |

**Important:** Use the specified Git commit and Python version. Do not use the system-default Python interpreter.

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

## 9. Reproducibility information

The following configuration was successfully tested:

- Fresh Git clone.
- Exact commit checkout.
- Python 3.10.20 virtual environment.
- Dependencies installed from the lockfile.
- ViT-L pretrained checkpoint downloaded and verified.
- All 20 example images processed successfully.
- 20 PNG depth visualisations generated.
- Total output size: approximately 86 MB.

The Git working tree may show untracked `checkpoints/`, `depth_vis/`, and `__pycache__/` directories. These are generated files and do not indicate an inference failure.

---
