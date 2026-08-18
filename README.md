# 🎬 Theatre Piracy Detection System

A computer vision-based **theatre piracy detection and watermark verification system** that embeds a hidden visual payload into a movie and detects the payload from a recorded/camera-captured version of the video.

The system is designed around the idea of placing small, short-duration visual symbols at predefined positions in a movie. These symbols act as a **forensic watermark** that can later be detected from a pirated recording.

---

## 🚀 Overview

Traditional visible watermarks can be easily noticed and removed. This project explores a more subtle approach where a sequence of geometric symbols is embedded into different regions of a movie for a short duration.

Each number in the encoded message is mapped to a visual operator:

| Code | Symbol |
| ---- | ------ |
| `1`  | `+`    |
| `2`  | `-`    |
| `3`  | `x`    |
| `4`  | `\|`   |
| `5`  | `--`   |
| `6`  | `\|\|` |

For example:

```text
Message:
[1, 5, 4, 5, 2, 6]

Encoded symbols:
+  --  |  --  -  ||
```

The encoded video can then be processed again to determine whether the watermark is present.

---

# ✨ Key Features

* 🎥 **Video watermark embedding**
* 🔐 Encodes a custom numeric payload into the video
* 🔢 Six unique geometric symbols for payload encoding
* ⏱️ Periodic watermark appearance
* 📍 Multiple watermark positions
* 🎨 Supports different watermark colors
* ↔️ Left and right side watermark placement
* ⬆️ Vertical watermark stacks
* 📏 Configurable watermark size
* 🎯 OpenCV-based watermark detection
* 🔍 HSV color segmentation
* 🖼️ Contour-based candidate detection
* 📐 Hough Line Transform for symbol classification
* 🧹 Morphological image processing
* 🎞️ Video frame-by-frame analysis
* 🔊 Preservation/restoration of original audio
* ✅ Automatic watermark verification

---

# 🧠 System Pipeline

```text
                    ORIGINAL VIDEO
                          │
                          ▼
                 ┌─────────────────┐
                 │ Payload Message │
                 │ [1,5,4,5,2,6...]│
                 └────────┬────────┘
                          │
                          ▼
                  Symbol Conversion
                          │
                          ▼
              ┌───────────────────────┐
              │ + -- | x - || symbols │
              └───────────┬───────────┘
                          │
                          ▼
                 Watermark Encoder
                          │
                          ▼
                 ENCODED MOVIE
                          │
                          ▼
                 Theatre Recording
                          │
                          ▼
                Recorded/Pirated Video
                          │
                          ▼
                  Watermark Detector
                          │
             ┌────────────┴────────────┐
             ▼                         ▼
       Color Segmentation        Edge Detection
             │                         │
             └────────────┬────────────┘
                          ▼
                   Hough Transform
                          │
                          ▼
                  Symbol Classification
                          │
                          ▼
                  Payload Verification
                          │
                    ┌─────┴─────┐
                    ▼           ▼
                  FOUND       NOT FOUND
```

---

# 🔐 Watermark Encoding

The encoder accepts a numeric message:

```python
MESSAGE = [1, 5, 4, 5, 2, 6]
```

The message is converted into symbols using:

```python
DIGIT_TO_SYMBOL = {
    1: "+",
    2: "-",
    3: "x",
    4: "|",
    5: "--",
    6: "||"
}
```

Each symbol is rendered using OpenCV drawing primitives such as:

* `cv2.line()`

The symbols are inserted into the video for:

```text
Interval = 5 seconds
Visible duration = 1 second
```

Therefore, a new payload symbol is displayed periodically rather than continuously.

---

# 📍 Watermark Placement

The project experiments with several watermark locations.

### Left Vertical Stack

Symbols are positioned vertically near the left side of the frame.

```text
┌──────────────────────────────┐
│                              │
│  [+]                         │
│                              │
│  [--]                        │
│                              │
│  [|]                         │
│                              │
│  [x]                         │
│                              │
└──────────────────────────────┘
```

### Right Vertical Stack

The later implementation focuses on a **white watermark on the right side**.

```text
┌──────────────────────────────┐
│                         [+]  │
│                              │
│                         [--] │
│                              │
│                         [|]  │
│                              │
│                         [x]  │
└──────────────────────────────┘
```

The right-side implementation also reduces the watermark size to make it less intrusive.

---

# 🎨 Watermark Variants

The notebooks contain experiments with different watermark configurations.

### Blue Watermark

```text
Blue symbols
```

Detected using an HSV blue range.

### Black Watermark

```text
Black symbols on light background
```

Detected using a black HSV range.

### White Watermark

```text
White symbols on dark background
```

Detected using a white HSV range.

The later experiments concentrate on:

> **Small white operators positioned on the right side of the video.**

---

# 📏 Small Watermark Version

The project reduces the watermark dimensions to make it less noticeable.

The smaller implementation uses:

```python
ROI = 100
THICK = 9
```

Compared with the earlier implementation:

```python
ROI = 200
THICK = 18
```

This reduces the visual footprint of the watermark while keeping the same symbol encoding mechanism.

---

# 🔍 Watermark Detection

The decoder processes the video frame by frame.

## 1. Read Video

```python
cap = cv2.VideoCapture(VIDEO)
```

Each frame is processed independently.

---

## 2. Convert to HSV

```python
hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
```

HSV makes it easier to isolate the watermark based on its color.

For example, white watermark detection uses:

```python
LOW_WHITE  = np.array([0, 0, 200])
HIGH_WHITE = np.array([180, 40, 255])
```

---

## 3. Color Segmentation

```python
mask = cv2.inRange(
    hsv,
    LOW_WHITE,
    HIGH_WHITE
)
```

This creates a binary mask containing candidate watermark regions.

---

## 4. Morphological Filtering

Morphological opening is applied to remove small noise:

```python
mask = cv2.morphologyEx(
    mask,
    cv2.MORPH_OPEN,
    np.ones((3,3), np.uint8)
)
```

---

## 5. Contour Detection

Candidate regions are extracted using:

```python
contours, _ = cv2.findContours(
    mask,
    cv2.RETR_EXTERNAL,
    cv2.CHAIN_APPROX_SIMPLE
)
```

Very small regions are discarded using an area threshold.

---

# 📐 Symbol Classification

After a candidate watermark region is found, the system converts it to grayscale and performs edge detection.

```python
gray = cv2.cvtColor(
    roi,
    cv2.COLOR_BGR2GRAY
)

edges = cv2.Canny(
    gray,
    40,
    120
)
```

The system then applies the **Hough Line Transform**:

```python
lines = cv2.HoughLines(
    edges,
    1,
    np.pi / 180,
    50
)
```

The detected line orientations are used to classify the symbol.

### Classification Logic

```text
Vertical + Horizontal
        │
        ▼
       +

Multiple Vertical Lines
        │
        ▼
       ||

Multiple Horizontal Lines
        │
        ▼
       --

Diagonal Lines
        │
        ▼
       x

Single Vertical Line
        │
        ▼
       |

Single Horizontal Line
        │
        ▼
       -
```

---

# ✅ Watermark Verification

The detector maintains a list of detected operators.

For example:

```text
Detected:
+, --, |, --, -, ||
```

The expected payload is converted into its corresponding operators.

The detector then checks whether the expected operator set is present.

Example:

```text
Expected operators:
{+, -, x, |, --, ||}

Detected operators:
{+, -, x, |, --, ||}

Result:
✅ WATERMARK VERIFIED
```

If one or more expected symbols are absent:

```text
❌ WATERMARK NOT VERIFIED
```

---

# ⚠️ Verification Logic

The current implementation verifies the **presence of the expected symbol types**, rather than reconstructing the exact chronological order of every symbol in the payload.

For example, it checks whether all required operators were detected:

```python
missing = EXPECTED_OPS - set(counts.keys())
```

Therefore, the current detector should be described as a **watermark presence/verification system**, rather than a complete ordered payload decoder.

---

# 🔊 Audio Preservation

Some encoder implementations create a temporary video without audio and then restore the original audio using FFmpeg.

The process is:

```text
Original Video
      │
      ├──────────────► Original Audio
      │
      ▼
Watermark Encoding
      │
      ▼
Encoded Video Without Audio
      │
      └──────────────┐
                     ▼
              FFmpeg Muxing
                     │
                     ▼
          Encoded Video + Original Audio
```

FFmpeg is used to copy the original audio stream without re-encoding it.

---

# 📁 Project Structure

```text
TheatrePiracyDetectionSystem/
│
├── Working_Pipeline1.ipynb
│
├── Working_pipeline2.ipynb
│
└── README.md
```

### `Working_Pipeline1.ipynb`

Contains the initial watermarking and detection experiments, including:

* Blue watermark encoding
* Multiple symbol configurations
* Video watermark detection
* HSV segmentation
* Hough-based symbol classification
* Watermark verification

### `Working_pipeline2.ipynb`

Contains the later experiments and refined pipeline, including:

* Black watermark experiments
* White watermark experiments
* Left-side watermark placement
* Right-side watermark placement
* Audio restoration
* Reduced watermark size
* Improved detection thresholds

---

# 🛠️ Technologies Used

| Technology       | Purpose                              |
| ---------------- | ------------------------------------ |
| Python           | Main programming language            |
| OpenCV           | Video processing and computer vision |
| NumPy            | Numerical/image operations           |
| FFmpeg           | Audio/video stream handling          |
| Jupyter Notebook | Experimentation and development      |

---

# 📦 Installation

Clone the repository:

```bash
git clone <repository-url>
cd TheatrePiracyDetectionSystem
```

Install the required Python packages:

```bash
pip install opencv-python numpy jupyter
```

FFmpeg is also required for the implementations that restore the original audio.

Check installation:

```bash
ffmpeg -version
```

---

# ▶️ Running the Project

Start Jupyter Notebook:

```bash
jupyter notebook
```

Open:

```text
Working_Pipeline1.ipynb
```

or:

```text
Working_pipeline2.ipynb
```

Update the input video path:

```python
INPUT_VIDEO = "/path/to/input_video.mp4"
```

Set the payload:

```python
MESSAGE = [1, 5, 4, 5, 2, 6]
```

Run the encoder cells to generate the watermarked video.

Then provide the resulting/recorded video to the corresponding decoder and set:

```python
EXPECTED_MESSAGE = [1, 5, 4, 5, 2, 6]
```

Run the decoder to check whether the watermark is detected.

---

# ⚙️ Configuration

The following parameters can be modified.

### Payload

```python
MESSAGE = [1, 5, 4, 5, 2, 6]
```

### Watermark Size

```python
ROI = 100
```

### Line Thickness

```python
THICK = 9
```

### Appearance Interval

```python
INTERVAL_SEC = 5
```

### Display Duration

```python
ON_SEC = 1
```

### Watermark Position

The position can be adjusted through functions such as:

```python
get_right_vertical_pos()
```

and:

```python
get_vertical_left_pos()
```

---

# 🧪 Experimental Workflow

The project progressively improves the watermarking approach:

```text
Initial Watermark
       │
       ▼
Blue Operators
       │
       ▼
Color Detection
       │
       ▼
Black / White Experiments
       │
       ▼
Left / Right Placement
       │
       ▼
Audio Preservation
       │
       ▼
Smaller Operators
       │
       ▼
White Right-Side Watermark
```

The final experiments focus on making the watermark **smaller and less visually intrusive** while maintaining detectability.

---

# 🎯 Applications

The system can be used as a prototype for:

* Theatre piracy detection
* Forensic video watermarking
* Movie leak investigation
* Camera-recording identification
* Digital content protection
* Hidden visual payload detection
* Cinema anti-piracy research

---

# 🔮 Future Improvements

* Decode the **exact ordered payload** instead of only checking symbol presence.
* Use temporal tracking to associate symbols with their correct timestamps.
* Add perspective and geometric distortion correction for camera recordings.
* Improve detection under different theatre lighting conditions.
* Handle motion blur and compression artifacts.
* Add robustness against cropping and resizing.
* Use feature-based or learned symbol recognition instead of only Hough Lines.
* Automatically estimate the watermark location.
* Generate a unique watermark for each theatre, screen, or playback session.
* Build an automated report containing detection confidence and timestamps.
* Evaluate detection accuracy across different camera angles, resolutions, and recording conditions.

---

# 👨‍💻 Author

**Pranav Redij**

---

## 📌 Project Summary

**Theatre Piracy Detection System** is a computer-vision prototype for embedding subtle geometric watermarks into movie frames and detecting them from recorded video. It uses **OpenCV, HSV segmentation, morphological processing, contour detection, Canny edge detection, and Hough Line Transform** to identify hidden symbols and verify the presence of an encoded payload.
