# Dev Log

Last updated: 2026-05-02

## Current Status

PillCount is currently a static GitHub Pages web app for mobile pill counting. The app uses the phone camera or selected images, runs detection in the browser with OpenCV.js, shows a detected count, and allows manual correction with add/remove marks and +/- controls.

The current development focus is moving from a basic working counting flow toward a repeatable test-and-tune workflow using real PillEye images.

## Completed Work

### PillEye Frame Extraction Tool

Added a Python utility:

```text
tools/extract_pilleye_frames.py
```

Purpose:

- Convert PillEye screen recordings into JPG test images.
- Sample approximately one frame per second.
- Skip near-duplicate frames.
- Skip blurry frames, especially during scrolling or motion.
- Output images as sequential JPG files.
- Output metadata CSV with:
  - `filename`
  - `time_sec`
  - `frame_no`
  - `blur_score`

Default input video:

```text
C:\Users\liaow\Downloads\Phone Link\Record_2026-05-01-23-04-23_5c855348d3281d1dae77e38e40ebf34c.mp4
```

Default output folder:

```text
C:\Users\liaow\Downloads\PillEye_frames
```

The tool supports:

```text
--video
--out
--prefix
--interval
--blur-threshold
--diff-threshold
```

The `--prefix` option was added so multiple videos can be exported into the same folder without overwriting previous image sets.

Example:

```powershell
python .\tools\extract_pilleye_frames.py --video "C:\Users\liaow\Downloads\Phone Link\Record_2026-05-02-00-04-15_5c855348d3281d1dae77e38e40ebf34c.mp4" --prefix pilleye_000415
```

This produces files such as:

```text
pilleye_000415_0001.jpg
pilleye_000415_0002.jpg
metadata_pilleye_000415.csv
```

### Requirements

Added:

```text
requirements.txt
```

Current content:

```text
opencv-python
```

### README Update

Updated `README.md` with usage instructions for the PillEye frame extraction tool, including Windows path examples and parameter explanations.

### Test Image Dataset Preparation

Generated test images from multiple PillEye screen recordings into:

```text
C:\Users\liaow\Downloads\PillEye_frames
```

Current known groups include:

```text
pilleye_default
pilleye_232040
pilleye_000415
```

For the `pilleye_000415` set:

- Original extracted JPG count: 234
- Filtered valid images kept: 99
- Deleted invalid images: 135
- Updated metadata file:

```text
metadata_pilleye_000415.csv
```

The filtering rule used for this cleanup:

- Keep images where the bottom of the screen contains the green `Scan barcode` bar.
- Delete non-counting screens such as history lists or screens without a full pill/tablet image.

## Current Testing Workflow

The current suggested manual testing workflow is:

1. Start the local static server:

```powershell
cd "C:\Users\liaow\Documents\Codex\2026-05-01\python-1-c-users-liaow-downloads\pillcount"
python -m http.server 8000
```

2. Open:

```text
http://localhost:8000
```

3. Use the app's PillEye image upload flow to select images from:

```text
C:\Users\liaow\Downloads\PillEye_frames
```

4. Record test results manually:

```csv
filename,expected_count,actual_count,error,note
pilleye_000415_0003.jpg,24,24,0,OK
pilleye_000415_0014.jpg,48,46,-2,missed edge tablets
```

Recommended workflow:

- Start with about 20 images.
- Identify common failure types.
- Tune detection parameters or logic.
- Re-test the same 20 images.
- Then run through the full image set.

## Known Limitations

### Shape Detection

The current counting approach is strongest for round tablets. It may be less accurate for:

- Oval tablets
- Capsules
- Long tablets
- Half-round or irregular tablets
- Mixed-shape images

This is because a detection strategy based mainly on circular shapes does not generalize well to non-round pills.

### Touching or Overlapping Pills

If pills touch each other or overlap, contour-based detection may merge multiple pills into a single object, causing under-counting.

### Packaging, Text, and Background Noise

Some PillEye images include:

- Plastic bag reflections
- Printed text
- Barcode UI
- Memo input areas
- History/list UI areas

These can create false positives unless the detection area is constrained or the segmentation logic is improved.

## Unfinished Discussion / Next Development Tasks

### 1. Support Non-Round Pills

Need to add a detection mode that can count non-circular pills.

Possible approach:

- Keep current round-tablet detection.
- Add contour/object detection for non-round tablets.
- Use features such as:
  - contour area
  - bounding box size
  - aspect ratio
  - ellipse fit
  - solidity
  - circularity

Potential UI modes:

```text
Round
Oval / Capsule
Mixed
```

The mixed mode should detect pill-like independent objects without requiring them to be circular.

### 2. Improve Separation of Touching Pills

Possible approaches:

- Distance transform
- Watershed segmentation
- Local maxima detection
- Better contour splitting for clustered pills

This is important for dense images where pills are close together.

### 3. Add a Batch Testing / Labeling Page

A helpful next tool would be a browser-based test page that can:

- Load a folder or list of test images.
- Show one image at a time.
- Run the current detector.
- Let the user enter the correct count.
- Save/export a CSV result table.

Suggested output:

```csv
filename,expected_count,actual_count,error,note
```

This would make optimization much faster than manually writing results in Excel.

### 4. Build a Ground Truth Dataset

Need to manually label a reliable subset of the PillEye images.

Suggested starting point:

- Label 20 easy images.
- Label 20 medium images.
- Label 20 difficult images.
- Include round, oval, capsule, and mixed-shape examples.

This dataset can be used to compare detection improvements.

### 5. Tune Detection Against Real PillEye Images

Use the images in:

```text
C:\Users\liaow\Downloads\PillEye_frames
```

Tune the detector based on error type:

- If under-counting:
  - lower minimum pill size
  - increase sensitivity
  - improve splitting of touching pills

- If over-counting:
  - raise minimum pill size
  - reduce sensitivity
  - improve text/background filtering
  - constrain detection to the pill photo region

### 6. Avoid Detecting UI Outside the Pill Photo

PillEye screenshots include app UI. Detection should ideally focus on the actual photo region, not the whole screenshot.

Possible approaches:

- Crop to the main image/photo area before detection.
- Ignore bottom UI areas such as memo input and `Scan barcode`.
- Detect the green `Scan barcode` bar to estimate where non-photo UI begins.

## Notes

The current image extraction and cleanup pipeline is good enough to start real detector testing. The next major product improvement is not more screenshot extraction, but building a feedback loop:

```text
test image -> detector result -> expected count -> error analysis -> detector tuning
```

Once this loop exists, PillCount can be improved much more systematically.
