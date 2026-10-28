# Workshop 02 — Depth Anything V2

## 1. Project Overview

**Repository:** [Depth Anything V2](https://github.com/DepthAnything/Depth-Anything-V2)  
**Inference task:** This project uses the pretrained AI model Depth Anything V2 to estimate the relative depth of objects and their surroundings from a single image. The ViT-L model processes sample images and generates depth maps.

## 2. Repository Map

| Component | Description |
|---|---|
| Repository | Depth Anything V2 |
| Environment | Python 3.10.20 and `.venv` |
| Dependencies | `requirements.txt` |
| Entry point | `run.py` |
| Model | Depth Anything V2 ViT-L pretrained model |
| Checkpoint | `checkpoints/depth_anything_v2_vitl.pth` |
| Input | `assets/examples/` |
| Output | `depth_vis/` |

The `run.py` script loads the selected pretrained model, processes the example images, and saves depth visualizations to the output directory. The ViT-L checkpoint must be available at the expected path before inference can run successfully.

## 3. Environment Setup

The following commands describe the environment setup used for this project:

```bash
git clone https://github.com/DepthAnything/Depth-Anything-V2.git
cd Depth-Anything-V2
uv python install 3.10
uv venv --python 3.10
source .venv/bin/activate
uv pip install -r requirements.txt
```

**Python version used:** Python 3.10.20  
**Virtual environment:** `.venv`  
**Dependency file:** `requirements.txt`

For the homework reproduction, use a fresh clone and create a new virtual environment instead of reusing the original `.venv`.

## 4. Model and Checkpoint

The pretrained model used in this experiment is Depth Anything V2 ViT-L.

**Checkpoint path:**

```text
checkpoints/depth_anything_v2_vitl.pth
```

During the initial setup, the original checkpoint was found to be corrupted or incomplete. I downloaded the official checkpoint again and replaced the faulty file. Inference succeeded after the checkpoint issue was resolved.

## 5. Inference

**Input directory:** `assets/examples/`  
**Output directory:** `depth_vis/`

Run the following command from the repository root:

```bash
python run.py --encoder vitl --img-path assets/examples --outdir depth_vis
```

The command runs the pretrained ViT-L model on the example images and saves the resulting depth visualizations in `depth_vis/`.

## 6. Verification and Evidence

The inference completed successfully for all 20 example images:

- 20 out of 20 images processed.
- 20 PNG output files generated.
- The output files had non-zero sizes.
- The total output size was approximately 86 MB.

Example output paths:

```text
depth_vis/demo01.png
depth_vis/demo02.png
...
depth_vis/demo20.png
```

These files provide direct evidence that pretrained inference completed successfully.

### Verification commands

```bash
ls -lh depth_vis
find depth_vis -maxdepth 1 -type f -name '*.png' | wc -l
find depth_vis -maxdepth 1 -type f -name '*.png' | sort
```

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

## 11. Submission Evidence

The final workshop submission should be organized under:

```text
MLMPworkshop/
└── workshop02_AI_Project_Survival/
    ├── README.md
    └── evidence/
        ├── input/
        └── output/
```

### Recommended evidence

- **Input evidence:** Representative example input image(s), or a reference to the repository's existing example images.
- **Output evidence:** Representative generated depth visualization(s).
- **Execution evidence:** Terminal output showing the inference command and successful processing.
- **Verification evidence:** An output file listing showing the generated PNG files.
- **Failure evidence:** The original checkpoint error, if available.
- **Fix evidence:** A note showing that the official checkpoint was downloaded again.
- **Reproducibility evidence:** Fresh-clone test results and peer feedback, once available.

The generated outputs total approximately 86 MB. Unless the instructor requires every output file, a few representative images and a clear verification record may be sufficient instead of copying the entire output directory.

## 12. Current Status

**Core inference task: COMPLETED**

| Requirement | Status |
|---|---|
| Inspect the repository | Completed |
| Configure the Python environment | Completed |
| Install dependencies | Completed |
| Resolve the checkpoint failure | Completed |
| Run pretrained inference | Completed |
| Process 20 example images | Completed |
| Generate 20 PNG outputs | Completed |
| Document the workflow | Completed |
| Fresh-clone reproduction | Pending |
| Peer reproduction | Pending |
| Revise README based on feedback | Pending |
| Finalize submission evidence | Pending |

## 13. Conclusion

The pretrained inference task for Workshop 02 has been completed successfully in the original working environment. The model processed all 20 example images and generated 20 PNG output files.

The next objective is to reproduce the result from a fresh clone, obtain peer feedback, and finalize the submission evidence.
