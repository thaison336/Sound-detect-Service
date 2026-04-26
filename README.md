# Smart Audio Detection Service

## Project Summary
This project is a real-time acoustic intelligence pipeline that combines embedded microphone-array sensing with software-based signal processing and deep learning classification.

It solves a practical edge AI problem: converting raw noisy audio streams into actionable environmental events (what sound happened, how confident the model is, and where it came from in 360 degrees). This matters for surveillance, smart devices, and HMI systems where low-latency context awareness is required without cloud dependency.

## Why This Project Is Technically Strong
- Integrates hardware-level sensing and software inference in one system:
  - ReSpeaker XVF-3000 USB controls for VAD/speech/DOA telemetry.
  - DSP pre-processing before ML inference.
  - Real-time visualization (console and GUI).
- Uses a hybrid detection strategy:
  - Fast rule-based classification for immediate feedback.
  - CNN-based environment classification for richer semantics.
- Designs for streaming stability, not just single-shot inference:
  - Sliding audio buffer.
  - Prediction hop scheduling.
  - Probability smoothing and controlled label switching.
  - Silence-aware state reset.

## Key Features
- Real-time pipeline at 16 kHz with 1024-sample streaming chunks for low-latency updates.
- Hardware-assisted perception from ReSpeaker:
  - Voice Activity Detection.
  - Speech flag.
  - Direction of Arrival (0-359 degrees).
- DSP chain in audio_processor.py:
  - 4th-order band-pass filtering (100-7500 Hz).
  - FFT-based spectral gating.
  - Adaptive Gain Control with noise gate and gain smoothing.
- Hybrid classifier in audio_classifier.py:
  - Rule-based tags: silence, speech, music, noise.
  - CNN model (audio_cnn_best.h5) over log-mel features for environmental events.
- Temporal robustness mechanisms:
  - 2.0 s inference window with 0.5 s prediction hop.
  - Rolling probability history (K=3) for smoothing.
  - Confidence threshold fallback to unknown.
  - Fast-switch streak logic to reduce class flapping.
- Recruiter-friendly product surfaces:
  - Interactive dashboard (gui_app.py).
  - Live terminal demo (smart_audio_pipeline.py).
  - Service/API/CLI scaffolding for integration workflows.

## System Architecture

### High-Level View
1. Audio is captured from microphone stream (PyAudio).
2. Signal is cleaned by DSP module (band-pass, spectral gate, AGC).
3. Rule-based classifier computes immediate sound type.
4. CNN classifier runs on buffered windows for environment labels.
5. ReSpeaker USB telemetry provides VAD/speech/DOA in parallel.
6. Results are rendered to GUI/console and can be served via service layer.

### Main Components
- sound_detector.py
  - USB control transfer interface to ReSpeaker tuning parameters.
  - Reads VAD, speech-detected flag, AGC gain, and sound direction.
- audio_processor.py
  - Real-time DSP front-end to improve SNR before classification.
- audio_classifier.py
  - Streaming input, feature extraction, rule-based tagging, CNN inference.
  - Implements smoothing/reset/switch logic for stable online predictions.
- smart_audio_pipeline.py
  - Orchestrates DSP + classifier and outputs live analytics table.
- gui_app.py
  - Operational dashboard showing signal levels, confidence, and DOA radar.
- sound_service.py, api.py, cli.py
  - Integration interfaces for monitoring and service-style deployment.

### Data Flow
```text
Mic Stream -> Chunk (1024) -> DSP Clean -> Rule-Based Tag
                                                             |
                                                             -> Buffer (2s window, 0.5s hop) -> Log-Mel -> CNN -> Smoothed Label

ReSpeaker USB -> VAD/Speech/DOA -------------------------------> Merge -> UI / Service Output
```

## Tech Stack (and Why)
- Python
  - Fast iteration for audio + ML prototyping and hardware integration.
- PyAudio
  - Direct low-latency microphone stream handling.
- NumPy + SciPy
  - Efficient numeric operations and DSP primitives.
- Librosa
  - Reliable spectral and log-mel feature extraction for audio ML.
- TensorFlow/Keras
  - Model loading/inference for CNN-based environment recognition.
- PyUSB
  - Device-level communication with ReSpeaker control interface.
- Rich + Tkinter
  - Two UX layers: terminal observability and desktop dashboard.
- Flask (+ CORS)
  - REST integration surface for external systems.

## Setup

### Prerequisites
- Python 3.9+ recommended.
- ReSpeaker Mic Array v2.0 (for hardware VAD/DOA features).
- Windows users: libusb driver for SEEED Control (Interface 3).

### Install
```bash
git clone https://github.com/thanhtoan23/Sound-detect-Service.git
cd Sound-detect-Service

python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

### Windows Driver Note (ReSpeaker control interface)
Use Zadig to install libusb-win32 for SEEED Control (Interface 3) only.
Do not replace the regular USB audio interface driver.

## Usage (Minimal and Reproducible)

### 1) Run the end-to-end console pipeline
```bash
python smart_audio_pipeline.py
```
What you get:
- DSP status (input/output RMS, AGC gain).
- Rule-based sound type.
- Smoothed CNN label + confidence.

### 2) Run the real-time dashboard
```bash
python gui_app.py
```
What you get:
- Live signal meters.
- Confidence bar and predicted class.
- Direction radar (from ReSpeaker telemetry).

### 3) Quick hardware check (VAD/DOA)
```bash
python sound_detector.py
```

## Engineering Challenges and Solutions
- Challenge: Streaming predictions are noisy and can flicker between classes.
  - Solution: Added hop-based inference, rolling probability smoothing, and streak-based fast-switch control.
- Challenge: Low-energy background noise can be amplified by AGC.
  - Solution: Added gate threshold, gate ratio, gain cap, and smoothing in AudioProcessor to prevent noise blow-up.
- Challenge: Real-time UX must remain responsive while processing continuously.
  - Solution: Separated processing loop from UI loop using thread + queue patterns in the GUI and service layers.
- Challenge: Embedded hardware telemetry and audio inference operate at different cadences.
  - Solution: Merged asynchronous ReSpeaker status with chunk-based software inference into a unified output state.

## Future Improvements
- Harden the service/API path into a production-ready deployment target with integration tests.
- Add automated benchmark scripts for latency, throughput, and per-class confidence drift.
- Move configurable thresholds/hops/windowing into a single typed config schema.
- Add model versioning and experiment tracking for reproducible retraining.
- Export structured telemetry (Prometheus/OpenTelemetry) for observability in edge deployments.
- Extend dataset and evaluate with confusion matrices and domain-shift scenarios.

## Repository Structure
```text
audio_processor.py        # DSP front-end (filtering, gating, AGC)
audio_classifier.py       # Streaming features + rule-based + CNN inference
sound_detector.py         # ReSpeaker USB telemetry (VAD/Speech/DOA)
smart_audio_pipeline.py   # End-to-end console orchestrator
gui_app.py                # Real-time dashboard
sound_service.py          # Background service loop and state management
api.py                    # Flask API surface
cli.py                    # Terminal command interface
```

## Candidate Positioning
This codebase demonstrates practical strengths expected from a strong applied-ML/software candidate:
- Building full-stack sensing systems from hardware IO to inference UX.
- Designing real-time pipelines with explicit tradeoffs between latency and stability.
- Combining DSP fundamentals and deep learning in a deployable architecture.

## Contributors
- Truong Thien An
- Nguyen Thai Son
- Nguyen Thanh Toan
- Le Cong Vinh