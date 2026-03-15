# Suspicious Activity Detection in Recorded and Live Surveillance

> Published and presented at **2023 IEEE Region 10 Conference (TENCON)** | [View on IEEE Xplore](https://ieeexplore.ieee.org/document/10322454)

A dual-module real-time surveillance system that detects suspicious activity in both recorded videos and live camera feeds using deep learning and computer vision. The system dispatches automated SMS alerts via Twilio upon detection and is served through a Flask web interface.

---

## How It Works

The pipeline uses a distance-based decision rule to route each frame to the appropriate detection module:

- **Face distance ≤ 200** → Head movement module (MediaPipe face mesh + PnP)
- **Face distance > 200** → Body gesture module (Spatiotemporal Autoencoder)

---

## Module 1: Body Gesture Anomaly Detection

A spatiotemporal autoencoder trained on the [Avenue Dataset](http://www.cse.cuhk.edu.hk/leojia/projects/detectabnormal/dataset.html) to reconstruct normal video sequences and flag anomalies via reconstruction error.

### Architecture

| Layer | Type | Details |
|---|---|---|
| 1 | Conv3D | 128 filters, 11×11 kernel, stride 4, tanh |
| 2 | Conv3D | 64 filters, 5×5 kernel, stride 2, tanh |
| 3 | ConvLSTM2D | 64 filters, 3×3 kernel, dropout 0.4/0.3 |
| 4 | ConvLSTM2D | 32 filters, 3×3 kernel, dropout 0.3 |
| 5 | ConvLSTM2D | 64 filters, 3×3 kernel, dropout 0.5 |
| 6 | Conv3DTranspose | 128 filters, 5×5 kernel, stride 2, tanh |
| 7 | Conv3DTranspose | 1 filter, 11×11 kernel, stride 4, tanh |

- **Input shape:** (None, 227, 227, 10, 1)
- **Loss:** Mean Squared Error
- **Optimizer:** Adam (lr=0.001, β1=0.9, β2=0.999)
- **Anomaly threshold:** MSE > 0.0068 → flagged as suspicious
- **Framework:** TensorFlow 2.3.1

---

## Module 2: Head Movement Detection

Uses **MediaPipe Face Mesh** to extract 6 key landmarks from 468 detected facial points and applies **PnP (Perspective-n-Point)** problem-solving to estimate head pose in 3D space. Suspicious directional head patterns trigger an alert.

---

## Alert System

Both modules feed into an automated **Twilio SMS alert pipeline**. When either module flags a suspicious event, a real-time SMS notification is dispatched to the configured recipient — designed for use cases including ATM security monitoring and exam supervision.

---

## Web Interface

A **Flask** application serves the inference pipeline with a simple UI (HTML/CSS frontend in `templates/` and `static/`) for uploading recorded videos or connecting a live feed.

---

## Project Structure
```
Suspicious_Activity_Detection/
├── data/
│   └── Avenue_Dataset/       # Training data
├── models/                   # Saved model weights
├── src/                      # Detection modules
├── static/                   # CSS and assets
├── templates/                # Flask HTML templates
├── .env.example              # Environment variable template
├── requirements.txt
└── README.md
```
---

## Setup
```bashgit clone https://github.com/lavender2412/Suspicious_Activity_Detection.git
cd Suspicious_Activity_Detection
pip install -r requirements.txt
cp .env.example .env
Add your Twilio credentials to .env
python app.py

---

## Environment VariablesTWILIO_ACCOUNT_SID=your_account_sid
TWILIO_AUTH_TOKEN=your_auth_token
TWILIO_FROM=+1xxxxxxxxxx
TWILIO_TO=+1xxxxxxxxxx

---

## Publication

**Suspicious Activity Detection in Recorded and Live Surveillance**
2023 IEEE Region 10 Conference (TENCON) | [View on IEEE Xplore](https://ieeexplore.ieee.org/document/10322454)

---

## Tech Stack

Python, TensorFlow, OpenCV, MediaPipe, Flask, Twilio, HTML/CSS
