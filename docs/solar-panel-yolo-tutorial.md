# Solar Panel Detection using YOLOv11 (Instance Segmentation)

This tutorial shows how to **train and use YOLOv11 for detecting and segmenting solar panels** in aerial/drone/satellite images using **Ultralytics YOLO** + **Roboflow** dataset management.

Project goal: Build a model that can find solar panel arrays and create pixel-level masks (instance segmentation), useful for:

- Solar potential estimation
- Roof-top PV inventory
- Maintenance inspection (hotspots, dirt, cracks — if extended)
- Urban energy planning

## 1. Why YOLOv11?

- Very fast inference (real-time possible even on edge devices)
- Excellent small-object detection improvements (important for aerial views)
- Native instance segmentation support (`yolo11s-seg.pt`, `yolo11m-seg.pt`, etc.)
- Active development & community (Ultralytics)
- Easy training on custom datasets

## 2. Prerequisites

- Google Colab (free T4 GPU) or local machine with NVIDIA GPU + CUDA
- Basic Python knowledge

Install required packages:

```bash
!pip install -q ultralytics roboflow

```
## 3. Get Your Dataset from Roboflow

Replace the workspace, project name and version number with your own values.

```bash
from roboflow import Roboflow

# Use your private key (never commit it to GitHub!)
rf = Roboflow(api_key="YOUR_ROBOFLOW_API_KEY_HERE")

project = rf.workspace("blaqdiamond").project("id-gwois")   # ← change
version = project.version(1)                                # ← change

dataset = version.download("yolov11")
```
After running, you should see a folder similar to /content/id-gwois-1/ with this structure:

├── data.yaml
├── README.roboflow.txt
├── train/
│   ├── images/
│   └── labels/
├── valid/
│   ├── images/
│   └── labels/
└── test/ (optional)

## 4. Quick Dataset Sanity Check
Verify that images and labels were downloaded correctly:

```bash
from pathlib import Path

root = Path(dataset.location)
print("Dataset location:", root)

for split in ["train", "valid", "test"]:
    img_dir = root / split / "images"
    if img_dir.exists():
        images = list(img_dir.glob("*.[jJ][pP][gG]*")) + \
                 list(img_dir.glob("*.[pP][nN][gG]"))
        print(f"{split:6s} → {len(images):4d} images")
```
## 5. Training the Model
We recommend instance segmentation for solar panels (you get both boxes + masks).

```bash

from ultralytics import YOLO
import os

# Recommended starting point: small segmentation model
model = YOLO("yolo11s-seg.pt")
# Alternatives: "yolo11n-seg.pt" (faster, lighter), "yolo11m-seg.pt" (more accurate)

results = model.train(
    data     = os.path.join(dataset.location, "data.yaml"),
    epochs   = 60,                  # 40–120 depending on dataset size
    imgsz    = 640,                 # 640 is good balance for aerial images
    batch    = 16,                  # lower to 8 if you get out-of-memory error
    device   = 0,                   # GPU
    workers  = 8,
    patience = 25,                  # early stopping
    name     = "solar-panel-y11s-seg",
    project  = "runs/solar_detection",
    
    # Augmentations useful for aerial/satellite data
    degrees  = 15.0,
    translate = 0.2,
    scale    = 0.5,
    shear    = 5.0,
    flipud   = 0.4,
    fliplr   = 0.5,
    mosaic   = 1.0,
    mixup    = 0.15,
)

```

## 6. Validate the Trained Model
After training finishes, you can re-run validation anytime:

```
# Load your best weights
model = YOLO("runs/solar_detection/solar-panel-y11s-seg/weights/best.pt")

metrics = model.val()
print(metrics)

```
## 7. Run Inference (Prediction)
On a single image

```

results = model.predict(
    source = "path/to/your/satellite_or_drone_image.jpg",
    save   = True,
    conf   = 0.35,
    iou    = 0.6,
    imgsz  = 640
)

# Display result (in Colab/Jupyter)
results[0].show()

```

On a folder 

```

model.predict(
    source = "folder/with/aerial_images/",
    save   = True,
    save_txt = True,       # also save YOLO-format labels
)

```

## 8. Export for Deployment

```

# Choose format depending on your target platform
model.export(format="onnx")       # most universal
# model.export(format="tflite")   # mobile / edge
# model.export(format="engine")   # TensorRT — fastest on NVIDIA GPUs

```

## 9. Next Steps & Improvements
Train longer (80–150 epochs) if your dataset is large
Experiment with higher resolution (imgsz=768 or 896) for small/dense panels
Add class balancing or focal loss if one class dominates
Post-process: calculate total panel area from masks, remove small false positives
Build a demo app (Gradio / Streamlit) to upload images and get predictions
Extend to defect detection (cracks, dirt, shading) with a second model
Try sliding-window inference for very large orthophotos



