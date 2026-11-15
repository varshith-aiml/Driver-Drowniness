# Real-Time Driver Drowsiness & Yawn Detection System

A lightweight computer vision application built with Python, OpenCV, and `dlib` that monitors a driver's eye closure and yawning patterns in real time via webcam. When signs of fatigue are detected, the system triggers an immediate audio alarm and displays an on-screen visual warning to help prevent drowsy driving accidents.

---

## Table of Contents

- [Overview & Features](#overview--features)
- [How the Math Works](#how-the-math-works)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Prerequisites & Installation](#prerequisites--installation)
- [Usage](#usage)
- [Configuration & Tuning](#configuration--tuning)
- [Troubleshooting](#troubleshooting)
- [Known Limitations & Roadmap](#known-limitations--roadmap)

---

## Overview & Features

* **Real-Time Face & Landmark Tracking:** Detects faces using OpenCV's Haar Cascade classifier and maps 68 facial landmarks using `dlib` at live webcam frame rates.
* **Eye Closure Detection (EAR):** Continuously computes the Eye Aspect Ratio for both eyes and tracks consecutive closed-eye frames to distinguish normal blinking from micro-sleeps.
* **Yawn Detection (MAR / Lip Distance):** Calculates vertical lip separation to catch wide yawns early.
* **Non-Blocking Audio Alerts:** Uses Python's `threading` module alongside `winsound` so the alarm sounds without freezing or lagging the live video stream.
* **Live HUD Overlay:** Draws green contours around the eyes and mouth while displaying real-time `EAR` and `YAWN` metrics in the top-right corner of the frame.
* **Multi-Camera Support:** Easily switch between built-in laptop webcams and external USB cameras via CLI arguments.

---

## How the Math Works

### 1. Eye Aspect Ratio (EAR)
Each eye is represented by 6 landmark coordinates $(p_1, p_2, p_3, p_4, p_5, p_6)$ from the 68-point facial landmark map:

$$\text{EAR} = \frac{\Vert{}p_2 - p_6\Vert{} + \Vert{}p_3 - p_5\Vert{}}{2 \cdot \Vert{}p_1 - p_4\Vert{}}$$

* **When the eye is open:** The numerator (vertical distance between upper and lower eyelids) stays relatively constant, keeping the EAR around `0.30 – 0.35`.
* **When the eye closes:** The vertical distance drops close to zero, causing the EAR to plummet. If it stays below `0.30` for `30` consecutive frames (~1 to 1.5 seconds depending on FPS), the **Drowsiness Alert** fires.

### 2. Lip Distance (Yawn Detection)
The script isolates landmarks `50–52` & `61–63` (top lip) and `56–58` & `65–67` (bottom lip), computes the centroid of each group, and measures the absolute vertical pixel difference (`abs(top_mean[1] - low_mean[1])`). When the mouth opens wide enough to exceed `20` pixels, the **Yawn Alert** triggers.

---

## Tech Stack

* **Python 3.8+**
* **OpenCV (`cv2`)** – Video capture, grayscale preprocessing, Haar Cascade face detection, and HUD rendering.
* **dlib** – 68-point facial landmark prediction (`shape_predictor_68_face_landmarks.dat`).
* **imutils** – Frame resizing and facial landmark index helpers.
* **SciPy & NumPy** – Euclidean distance calculations and array operations.
* **winsound & threading** – Asynchronous audio alerts on Windows.

---

## Project Structure

Make sure both pre-trained model files are present in the project directory (note: if you move them into a `models/` folder to keep your GitHub repo tidy, update the file paths on lines 72–73 of the script):

```text
driver-drowsiness-detection/
│
├── drowniness_yawn.py                      # Main detection script
├── haarcascade_frontalface_default.xml     # OpenCV Haar Cascade face detector
├── shape_predictor_68_face_landmarks.dat   # dlib 68-point facial landmark model (~99.7 MB)
├── requirements.txt                        # Python dependencies
└── README.md                               # Project documentation
