# Window Guard AI

**Edge-AI child detection on Raspberry Pi that automatically lowers window blinds and alerts parents to help prevent toddler window falls.**

*McMaster University · ELEC ENG 4OI6 Capstone · 2026–27*

<!-- Add a demo GIF here once you have one: ![Demo](docs/demo.gif) -->

---

## The Problem

Falls from windows and balconies are a leading cause of serious injury for young children in Canada. Between 2010 and 2023, 48 fall-related deaths were reported among children aged 0–9, and nearly half involved falling out of or through a building structure. It only takes a few seconds of inattention for a toddler to climb onto furniture and reach an open window.

## Our Solution

Window Guard AI watches a "hazard zone" around a window. When a **young child (ages 1–5)** stays in the zone, the system:

1. **Lowers a motorized blind** as a physical barrier
2. **Sounds a local alarm** (buzzer/LED) within 2 seconds
3. **Sends a notification** to the parent's phone within 5 seconds

Adults walking past or using the window do **not** trigger the barrier. All video is processed **on-device**. No images are stored or uploaded.

## How It Works

```
 Pi Camera Module 3 ──► Raspberry Pi 4B ──────────────┐
                        • YOLO11n (child / adult)     │
                        • Hazard-zone + tracking      │  trigger
                        • Dwell-time logic            ▼
 mmWave sensor ────────────────────────────────► ESP32 ──► Blind motor
                                                   │   ──► Buzzer / LED
                                                   │   ──► Manual override
                                                   ▼
                                           Parent notification (Wi-Fi)
```

| Component | Role |
|---|---|
| **Raspberry Pi 4B + Camera Module 3** | Runs the custom-trained YOLO model to detect children vs. adults |
| **mmWave sensor** | Confirms presence near the window regardless of lighting |
| **ESP32** | Controls the blind motor, alarm, and fail-safe manual override |
| **Motorized zebra blind** | Physical barrier between child and window |

### Detection logic
- A person is detected and **tracked** across frames (ByteTrack)
- If a **child** is inside the hazard zone for longer than the dwell threshold → trigger
- If the model is **uncertain** → treat the person as a child (safety first)
- An adult **carrying** a child does not trigger the barrier

## The Model

- **Base model:** Ultralytics YOLO11n, fine-tuned on 2 classes: `child`, `adult`
- **Data:** public datasets (Open Images V7, Roboflow Universe) + our own footage of adults from the deployment camera. No images of real children were collected without consent.
- **Deployment:** exported to NCNN for fast CPU inference on the Raspberry Pi

### Results

| Metric | Result |
|---|---|
| Child recall | _TBD_ |
| Child precision | _TBD_ |
| mAP@50 | _TBD_ |
| Inference speed (Pi 4B, 320 px) | _TBD_ FPS |
| Detection → barrier trigger latency | _TBD_ s |
| False alarms per hour | _TBD_ |
| Improvement over height-threshold baseline | _TBD_ |

## Repository Structure

```
window-guard-ai/
├── ml/            # Dataset prep, training configs, evaluation
├── pi/            # Raspberry Pi detection + tracking script
├── esp32/         # Firmware for blind motor, alarm, override
├── app/           # Parent notification interface
├── hardware/      # Schematics, wiring, mechanical drawings
└── docs/          # Reports, diagrams, test results
```

## Getting Started

### Train the model (laptop)
```bash
pip install ultralytics
yolo train model=yolo11n.pt data=ml/data.yaml epochs=100 imgsz=320
```

### Export for the Pi
```bash
yolo export model=runs/detect/train/weights/best.pt format=ncnn imgsz=320
```

### Run on the Raspberry Pi
```bash
pip install ultralytics picamera2
python pi/detect.py --model best_ncnn_model
```

## Privacy & Safety

- **On-device only:** video is never streamed or uploaded; frames are discarded after processing
- **No child data collected without consent:** training uses public datasets and consenting adults
- **Fail-safe:** the blind can always be raised manually without a key, network, or tool, consistent with Ontario O. Reg. 517/06 requirements for window safety devices
- **Anti-entrapment:** controlled motor force and speed to protect fingers

## Team

| Name | Program | Module |
|---|---|---|
| Sarah Ahmed | _TBD_ | _TBD_ |
| Faiqa Anjum | _TBD_ | _TBD_ |
| Jeniishaa Bhagudeva | _TBD_ | _TBD_ |
| Aliyah Qayum | _TBD_ | _TBD_ |

**Supervisor:** Dr. Shahram Shirani

## License

This project uses [Ultralytics YOLO](https://github.com/ultralytics/ultralytics), which is licensed under AGPL-3.0, so this repository is released under the **AGPL-3.0** license as well.

## Acknowledgements

- Public Health Agency of Canada, *Children's Falls*, Health Infobase
- Ultralytics YOLO
- Open Images Dataset V7 (Google)
