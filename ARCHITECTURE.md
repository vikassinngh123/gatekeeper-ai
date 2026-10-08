## Technical Architecture Log: Biometric Pipeline Migration

### Legacy Pipeline: `dlib` / `face_recognition`
* **Model:** ResNet-34 face feature extractor (dlib).
* **Benchmark Accuracy:** ~99.38% on LFW.
* **Bottlenecks:**
  * High CPU inference latency (150–300 ms per crop), degrading real-time multi-person video feeds.
  * Rigid dependency pinning required (`numpy<2`, `opencv-python<5`, `setuptools<81`) due to legacy C++ Python bindings and deprecated `pkg_resources` references.
  * Large binary footprints and CMake build friction on Windows environments.

### Target Pipeline: OpenCV DNN (YuNet + SFace)
* **Detection:** YuNet (lightweight anchor-based detector targeting sub-10 ms latency).
* **Recognition:** SFace (ResNet-like deep feature extractor).
* **Benchmark Accuracy:** 99.60%–99.80% on LFW.
* **Architectural Advantages:**
  * Native OpenCV DNN module execution without external heavy C++ dependencies.
  * Native compatibility with modern Python 3.11+, NumPy 2.x, and OpenCV 5.x runtimes.
  * Heterogeneous workload distribution: YOLOv8 handles spatial tracking on the GPU, while SFace performs lightweight embedding extraction on CPU threads without resource contention.