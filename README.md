# GateKeeper AI 🛡️
Context-Aware Smart Virtual Fencing & Physical Access Control using YOLOv8, OpenCV DNN, and Streamlit.

GateKeeper AI is a real-time edge computer vision pipeline designed for physical access control. It combines dynamic spatial tracking with lightweight biometric verification. The system monitors virtual perimeters and triggers biometric verification and non-blocking email alerts only when an unauthorized spatial breach occurs.

## 🚀 Key Features

* **Interactive Virtual Fencing:** Draw custom polygonal or rectangular restricted zones directly over the live camera feed.
* **Spatial Geometry Tracking:** Uses YOLOv8 person detection mapped to foot-anchor coordinates (`(x1 + x2)/2, y2`) to prevent false alarms from upper-body boundary overlaps.
* **Conditional Biometric Verification:** Conserves compute resources by running facial recognition (OpenCV SFace) *only* when a valid person geometry breaches the drawn zone.
* **Real-Time Edge Performance:** Runs YOLOv8 spatial tracking on the GPU while offloading lightweight facial detection (YuNet) and feature extraction (SFace) to CPU threads.
* **Threaded Email Alerts:** Sends automated intrusion alerts via SMTP in a background worker thread with a cooldown timer to prevent video stream freezing.
* **Streamlit Dashboard:** Centralized interface for biometric enrollment, zone configuration, and live monitoring.

## 🧠 System Architecture

GateKeeper AI uses an optimized OpenCV DNN pipeline for real-time edge performance:

1. **Spatial Detection (YOLOv8):** Detects `class 0` (person) at > 0.5 confidence.
2. **Perimeter Math (`cv2.pointPolygonTest`):** Computes whether the bottom-center foot coordinate falls within the restricted polygon.
3. **Face Detection (YuNet):** If a breach occurs, the head region is localized using the lightweight YuNet ONNX model.
4. **Biometric Extraction & Matching (SFace):** Extracts 128-dimensional facial embeddings and calculates cosine similarity against enrolled authorized personnel.
5. **Alert Dispatcher (`smtplib`):** If an unrecognized individual breaches the perimeter, a daemon thread dispatches an alert email without blocking the video pipeline.

## 🛠️ Installation & Setup

Ensure you are running **Python 3.11**.

**1. Clone the repository:**
```bash
git clone [https://github.com/vikassinngh123/gatekeeper-ai.git](https://github.com/vikassinngh123/gatekeeper-ai.git)
cd gatekeeper-ai
```

**2. Install dependencies:**
```bash
pip install -r requirements.txt
```

**3. Download Model Weights:**
Ensure the following three model weight files are placed in the repository root directory next to `demo_app.py`:
* **YOLOv8 Nano:** `yolov8n.pt`
* **YuNet Face Detector:** [face_detection_yunet_2023mar.onnx](https://raw.githubusercontent.com/opencv/opencv_zoo/main/models/face_detection_yunet/face_detection_yunet_2023mar.onnx)
* **SFace Recognizer:** [face_recognition_sface_2021dec.onnx](https://raw.githubusercontent.com/opencv/opencv_zoo/main/models/face_recognition_sface/face_recognition_sface_2021dec.onnx)

You can download the ONNX models directly via terminal:
```powershell
curl.exe -L -o face_detection_yunet_2023mar.onnx "[https://raw.githubusercontent.com/opencv/opencv_zoo/main/models/face_detection_yunet/face_detection_yunet_2023mar.onnx](https://raw.githubusercontent.com/opencv/opencv_zoo/main/models/face_detection_yunet/face_detection_yunet_2023mar.onnx)"
curl.exe -L -o face_recognition_sface_2021dec.onnx "[https://raw.githubusercontent.com/opencv/opencv_zoo/main/models/face_recognition_sface/face_recognition_sface_2021dec.onnx](https://raw.githubusercontent.com/opencv/opencv_zoo/main/models/face_recognition_sface/face_recognition_sface_2021dec.onnx)"
```

## 🖥️ Usage

Launch the Streamlit dashboard:

```bash
streamlit run demo_app.py
```

### Application Workflow
1. **Tab 1: Face Enrollment:** Capture or upload an image and input an employee name to register authorized personnel into session storage.
2. **Tab 2: Setup Zone:** Use the canvas tool to draw the restricted perimeter over the camera field of view.
3. **Tab 3: Live Monitor:** Start the live camera feed. The system displays:
   * 🟦 **Blue Box:** Person outside the perimeter (Safe).
   * 🟩 **Green Box:** Authorized personnel verified inside the perimeter.
   * 🟥 **Red Box:** Unauthorized intruder breach (triggers alert banner and email notification).
