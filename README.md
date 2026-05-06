# 🚜 RTO Safety Compliance Tracking System
### Automated Safety Reflector Detection for Agricultural Vehicles
 
> **Built during a 2-month internship with the Regional Transport Office (RTO)**
> 🏆 **Winner — RTO Hackathon** | Team: TradeWise
 
---
 
## 📋 Overview
 
Tractors carrying agricultural loads on state highways are required by law to have a **red cloth with a white cross** (safety reflector) attached at the rear — making them visible to oncoming traffic at night. Non-compliance is a leading cause of road accidents in rural Maharashtra.
 
This system automates compliance enforcement at **factory exit gates** using CCTV cameras. It detects tractors in real time, checks for the safety reflector, extracts the number plate of violating vehicles, notifies the owner, and logs the violation to the RTO database — all without manual intervention.
 
> 📰 The problem was documented in local Marathi press (*Hello Jalna*, Dec 2024) before this project was built, validating its real-world urgency.

<img width="1220" height="1181" alt="image" src="https://github.com/user-attachments/assets/da0ecefc-ea67-41ac-81fb-4e008a8ad6cc" />


---
 
## 🎯 Problem Statement
 
| Issue | Impact |
|---|---|
| Tractors exit factories without mandatory safety reflectors | Night-time road accidents on state highways |
| Manual inspection is impractical at scale | Violations go undetected |
| No digital record of non-compliance | No accountability or follow-up |
 
**Objective:** Automate the detection, recording, and notification of safety reflector violations at factory gates — removing the need for manual checks.
 
---
 
## ⚙️ How It Works
 
```
CCTV Feed
    │
    ▼
┌─────────────────────────────┐
│  YOLOv8n + OpenCV           │  ← Real-time tractor detection
│  Frame-by-frame processing  │
└────────────┬────────────────┘
             │ Tractor detected?
             ▼
┌─────────────────────────────┐
│  Fine-Tuned VGG16           │  ← Red reflector present?
│  (TensorFlow / Keras)       │
└────────────┬────────────────┘
             │
     ┌───────┴───────┐
     │               │
   ✅ YES           ❌ NO
  Log pass       Extract plate
                 (Tesseract-OCR)
                     │
                     ▼
              Notify owner
                     │
                     ▼
           Log to RTO database
                     │
                     ▼
           React dashboard
           (review & action)
```
 
---
 
## 🧠 System Architecture
 
### Pipeline Components
 
**1. Tractor Detection (`detection/`)**
- OpenCV reads the live CCTV stream frame by frame
- YOLOv8n model runs inference on each frame
- If a tractor is detected with confidence ≥ threshold, the bounding box region is cropped and forwarded
**2. Reflector Classification (`classifier/`)**
- Cropped tractor image is passed to a fine-tuned VGG16 model
- Binary classification: `compliant` (reflector present) or `non_compliant` (reflector absent)
- Multi-frame buffering: requires consistent non-compliant prediction across N frames before triggering a violation (reduces false positives)
**3. Number Plate Extraction (`ocr/`)**
- Tesseract-OCR applied to the tractor image
- Pre-processing: grayscale conversion, adaptive thresholding, upscaling
- Post-processing: regex validation against Indian number plate format (`MH-XX-XX-XXXX`)
**4. Notification + Database Logging (`backend/`)**
- Plate number used to look up vehicle owner in RTO database
- Automated notification dispatched to owner
- Violation record stored: plate number, timestamp, captured frame, factory location, violation type
**5. React Dashboard (`frontend/` — built by web team)**
- RTO officers can review flagged violations with captured images
- Filter by date, factory gate, or plate number
- Mark violations as reviewed, dismissed, or escalated
---
 
## 🛠️ Tech Stack
 
| Component | Technology | Why |
|---|---|---|
| Video processing | OpenCV | Industry standard for real-time frame capture and image pre-processing |
| Object detection | YOLOv8n | Single-pass detection — fast enough for live video; nano variant runs on edge hardware |
| Classification | Fine-Tuned VGG16 | Transfer learning on VGG16 pre-trained on ImageNet; fine-tuned on domain-specific reflector images for higher accuracy with limited training data |
| OCR | Tesseract-OCR | Open-source, configurable for Indian number plate character sets |
| Backend | Python | Unified language across entire ML pipeline |
| Database | SQL (RTO database) | Stores violation records for audit trail and dashboard queries |
| Dashboard | React | Built by web team; consumes REST API endpoints from the backend |
 
---
 
## 📊 Model Details
 
### YOLOv8n — Tractor Detection
 
- Base model: `yolov8n.pt` (Ultralytics)
- Fine-tuned on a custom dataset of tractor images from factory gate CCTV footage
- Input: video frame (640×640)
- Output: bounding boxes with class label and confidence score
### Fine-Tuned VGG16 — Reflector Classifier
 
- Architecture: VGG16 pre-trained on ImageNet, fine-tuned for binary classification
- Top layers replaced: Flatten → Dense(256, ReLU) → Dropout(0.5) → Dense(1, Sigmoid)
- Training data: ~2500+ labeled images (compliant / non-compliant), augmented with flips, rotations, brightness shifts
- Framework: TensorFlow / Keras
- Saved format: `.h5`
- Binary output: `0` = compliant, `1` = non-compliant
---
 
## 📸 Results
 
**Detection output** — YOLOv8n identifying tractors in live CCTV footage with bounding boxes
 
**Classifier output** — Streamlit interface showing model loading and Red Cloth Detector in action
 
**Violation log** — Database entries with plate number, timestamp, and captured frame
 
---
 
## 🔮 Future Scope
 
- [ ] Automatic call to vehicle owners via government API
- [ ] Cloud deployment (AWS/GCP) for multi-gate, multi-factory scalability
- [ ] Expanded compliance checks: overloading detection, headlight verification
- [ ] Analytics dashboard with weekly/monthly violation trends per factory
- [ ] Mobile app for field RTO officers
- [ ] Integration with VAHAN database for real-time owner lookup
---
 
## 👥 Team
 
| Name | Role |
|---|---|
| Raj Patil | ML pipeline — detection, classification, OCR, backend |
| Pranav P Patil | Team coordination, research |
| Ritesh Patil | Data collection, model training support |
| Pranav Patil | React dashboard (frontend) |
 
> **Internship context:** This project was built as part of a 2-month technical internship with the Regional Transport Office (RTO), with support from the RTO's deployment and IT teams.
 
---
 
## 🏆 Recognition
 
- 🥇 **Winner** — RTO Hackathon
- 📰 Problem validated by local press coverage of tractor road accidents (*Hello Jalna, Dec 2024*)
- ✅ Presented to and reviewed by RTO officials as a scalable enforcement solution
---
 
## 📄 License
 
This project was developed for and in collaboration with the Regional Transport Office. For usage or collaboration enquiries, please contact the team.
 
---
 
<p align="center">
  Built with ❤️ for road safety in rural Maharashtra
</p>

