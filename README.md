# GateKeeper AI 🛡️
Context-Aware Smart Virtual Fencing & Physical Access Control using YOLOv8, OpenCV DNN, and Streamlit.

GateKeeper AI is a real-time edge computer vision pipeline designed for high-security physical access control. It moves beyond standard static image processing by combining dynamic spatial tracking with lightweight biometric verification. The system establishes virtual perimeters and conditionally triggers facial recognition only when an unauthorized spatial breach occurs.

## 🚀 Key Features

* **Interactive Virtual Fencing:** Draw custom polygonal restricted zones directly on the camera feed.
* **Spatial Geometry Tracking:** Utilizes YOLOv8 person detection mapped to foot-point coordinates to prevent false triggers from head/shoulder perimeter overlaps.
* **Conditional Biometric Verification:** Conserves compute resources by running facial recognition (OpenCV SFace) *only* when a valid person geometry breaches the drawn zone.
* **Real-time Edge Performance:** Heterogeneous workload distribution routes heavy spatial tracking (YOLOv8) to the GPU while handling lightweight embedding extraction (YuNet/SFace) on the CPU.
* **Streamlit Dashboard:** Centralized interface for face enrollment, zone configuration, and live monitoring.

## 🧠 System Architecture

GateKeeper AI recently migrated from a legacy `dlib`/ResNet pipeline to a highly optimized OpenCV DNN architecture to maximize frames-per-second (FPS) on local hardware (e.g., GTX 1650 / RTX 3050 setups).

1. **Detection Gate (YOLOv8):** Filters for `class 0` (person) at > 0.5 confidence.
2. **Spatial Math (`cv2.pointPolygonTest`):** Derives the bottom-center coordinate of the bounding box and checks intersection with the user-defined polygon.
3. **Face Detection (YuNet):** If a breach occurs, the head region is cropped and passed to YuNet for ultra-fast face localization.
4. **Biometric Embedding (SFace):** Extracts 128-dimensional facial features and compares them against enrolled user embeddings using Cosine Distance.

## 🛠️ Installation & Setup

Ensure you are running **Python 3.11**.

**1. Clone the repository:**
```bash
git clone [https://github.com/vikassinngh123/gatekeeper-ai.git](https://github.com/vikassinngh123/gatekeeper-ai.git)
cd gatekeeper-ai
```

**2. Install stabilized dependencies:**
```bash
pip install -r requirements_2.txt
```

**3. Download OpenCV DNN Models:**
The biometric pipeline requires two lightweight ONNX models from the OpenCV Zoo. Download and place them directly in the root directory next to `demo_app.py`:
* **YuNet (Face Detection):** [face_detection_yunet_2023mar.onnx](https://github.com/opencv/opencv_zoo/tree/main/models/face_detection_yunet)
* **SFace (Face Recognition):** [face_recognition_sface_2021dec.onnx](https://github.com/opencv/opencv_zoo/tree/main/models/face_recognition_sface)

**4. Download YOLOv8 Weights:**
Ensure you have the YOLOv8 nano weights downloaded to the root directory:
* **YOLOv8n:** `yolov8n.pt`

## 🖥️ Usage

Launch the Streamlit web application:

```bash
streamlit run demo_app.py
```

### Application Workflow
1. **Tab 1: Face Enrollment:** Stand in front of the camera, enter an ID/Name, and capture an image to register authorized personnel into the session state.
2. **Tab 2: Zone Configuration:** Use the interactive sliders/inputs to draw a restricted zone polygon over your camera's field of view. 
3. **Tab 3: Live Monitoring:** Activate the feed. The system will draw bounding boxes: Blue for individuals outside the zone, Green for authorized personnel inside the zone, and Red for unknown intruders breaching the perimeter.
