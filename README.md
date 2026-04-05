# PillCount — Pharmacy Pill Counter

A browser-based pill & capsule counter powered by YOLOv8 + ONNX Runtime Web.  
No installation. Works on any phone via a shared link.

## How to deploy on GitHub Pages

1. Create a new GitHub repository (e.g. `pillcount`)
2. Upload `index.html` to the root of the repo
3. Also upload your `model.onnx` file to the root (or a `/models/` folder)
4. Go to **Settings → Pages → Source → main branch / root**
5. GitHub gives you a URL like `https://yourusername.github.io/pillcount`
6. Share that URL with your colleagues — done!

## How to use

1. Open the URL on any phone (Chrome on Android, Safari on iPhone)
2. Tap the model file upload area and select your `.onnx` model
3. Tap **Start Camera** — allow camera permission
4. Point the camera at the pills on a counting tray
5. Tap **Capture & Count**
6. Bounding boxes appear on detected units
7. Optionally enter the drug name and tap **Save to history**

## Adjustable settings

| Setting | Description |
|---|---|
| CONF threshold | Minimum confidence to count a detection (default 50%) |
| IOU threshold | Overlap threshold for NMS deduplication (default 45%) |

## Model requirements

- Format: YOLOv8 nano exported to `.onnx`
- Input: `[1, 3, 640, 640]` float32
- Output: `[1, 4+nc, 8400]` or `[1, 8400, 4+nc]`
- Class 0 = tablet, Class 1 = capsule (adjust in code if different)

## Export your model to ONNX

```python
from ultralytics import YOLO
model = YOLO('your_model.pt')
model.export(format='onnx', imgsz=640, opset=12)
```
