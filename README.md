# AI-Based Cricket Ball Detection, Tracking & Simplified LBW DRS Prototype

A computer vision prototype demonstrating automated cricket ball detection, multi-frame trajectory tracking, physics-based trajectory extrapolation, and geometric stump-intersection assessment for educational Decision Review System (DRS) exploration.

---

## 1. Project Overview & Motivation

In modern international cricket, the **Decision Review System (DRS)** utilizes multi-camera high-speed tracking systems (such as Hawk-Eye and Virtual Eye) to assist on-field umpires in Leg Before Wicket (LBW) decisions. 

Building a broadcast-grade system requires synchronized multi-view 3D stereo cameras calibrated across the ground. The objective of this prototype is to construct a **reproducible, 2D monocular computer vision pipeline** that simulates the core computational stages of ball tracking and trajectory projection using accessible tools (OpenCV, Roboflow serverless inference, and NumPy numerical modeling).

> **Disclaimer**: This prototype implements an **experimental geometric trajectory approximation**. It does NOT constitute an official LBW decision. Official cricket laws (Law 36) dictate pitch zone assessment, impact point relative to stumps, and ball tracking margin of error ("Umpire's Call"), which require calibrated 3D field coordinate spaces.

---

## 2. System Architecture & Workflow

```
       +------------------------------------+
       |       Input Cricket Video          |
       |  (e.g., cricket.mp4 or broadcast)  |
       +-----------------+------------------+
                         |
                         v
       +------------------------------------+
       |  OpenCV Frame Extraction (cv2)     |
       |  Resolution: W x H, FPS: 25.0      |
       +-----------------+------------------+
                         |
                         v
       +------------------------------------+
       |  Roboflow Serverless Inference     |
       |  Model: 'cricket-drs-system/4'     |
       |  Detection Class: 'ball'           |
       +-----------------+------------------+
                         |
                         v
       +------------------------------------+
       |   Outlier Rejection & Tracking     |
       |   Euclidean velocity filtering     |
       |   Historical coordinate trail      |
       +-----------------+------------------+
                         |
                         v
       +------------------------------------+
       |  Trajectory Curve Extrapolation    |
       |  Bounce detection & quadratic fit  |
       |  y = a*x^2 + b*x + c projection    |
       +-----------------+------------------+
                         |
                         v
       +------------------------------------+
       |   Stump Region Intersection Test   |
       |   Geometric bounding box check     |
       +-----------------+------------------+
                         |
                         v
       +------------------------------------+
       |   Annotated Video & Telemetry HUD  |
       |   Output: outputs/drs_demo.mp4     |
       +------------------------------------+
```

---

## 3. Core Features Implemented

1. **Ball Detection (Feature A)**:
   - Queries Roboflow's serverless hosted model `cricket-drs-system/4` with header-based authentication.
   - Extracts bounding box center coordinates `(x, y)`, box dimensions `(w, h)`, and confidence score.
2. **Coordinate Tracking & Outlier Rejection (Feature B)**:
   - Maintains an ordered historical FIFO queue of coordinates.
   - Filters out anomalous leaps (false positives) using dynamic velocity distance thresholds.
   - Handles missed detections gracefully without resetting the tracked path.
3. **Bounce-Aware Trajectory Fitting (Feature C)**:
   - Identifies the pitch contact / rebound point (local maximum in image $Y$ coordinates).
   - Fits a polynomial curve ($y = ax^2 + bx + c$) to the post-bounce flight to project future motion under gravitational acceleration.
4. **Stump-Zone Geometric Intersection (Feature D)**:
   - Configurable 2D Region of Interest (ROI) around the striker's stumps.
   - Checks if the projected trajectory vector intersects the ROI coordinates.
   - Displays real-time collision marker and clear status banner (`INTERSECTS STUMPS` vs `PROJECTED CLEAR / MISS`).
5. **Video Export & Live Preview (Feature E & F)**:
   - Exports the fully annotated delivery to `outputs/drs_demo.mp4`.
   - Supports live OpenCV playback with an interactive heads-up telemetry display.

---

## 4. Setup & Installation

### Prerequisites
- Python 3.10+ (Tested on Python 3.12)
- Windows PowerShell / Command Prompt

### Step 1: Create and Activate Virtual Environment
```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

### Step 2: Install Dependencies
```powershell
pip install -r requirements.txt
```

### Step 3: Configure Environment Variables
Ensure `.env` contains your Roboflow credentials:
```env
ROBOFLOW_API_KEY=your_roboflow_api_key_here
ROBOFLOW_MODEL_ID=cricket-drs-system/4
```

---

## 5. Running the Demonstration

### Option 1: Run with Default Video & Local Preview
```powershell
.\venv\Scripts\python main.py --input cricket.mp4
```

### Option 2: Run with Your Custom Cricket Video
Place your cricket video file in the project folder (e.g. `my_match.mp4`) and run:
```powershell
.\venv\Scripts\python main.py --input my_match.mp4 --output outputs/drs_demo.mp4
```

### Option 3: Adjust Sampling Step (Save API Calls & Run Faster)
To process every 2nd or 3rd frame:
```powershell
.\venv\Scripts\python main.py --input cricket.mp4 --step 2
```

### Option 4: Calibrate Stump Target Box for Different Angles
```powershell
.\venv\Scripts\python main.py --input cricket.mp4 --stump-box 910 280 970 470
```

---

## 6. What I Built vs. What the Pretrained Model Provides

| Component | Pretrained Model (`cricket-drs-system/4`) | Custom Implemented CV Logic |
| :--- | :--- | :--- |
| **Object Detection** | Detects raw candidate bounding boxes for `ball` class | - |
| **Confidence Filtering** | Provides raw float confidence score | Configurable thresholding to eliminate low-certainty noise |
| **Coordinate Tracking** | None (stateless single-frame detection) | Multi-frame temporal queue, frame-to-frame velocity validation |
| **Physics Trajectory** | None | Pitch contact bounce detection & quadratic polynomial extrapolation |
| **Stump Intersection** | None | 2D bounding ROI collision detection & target crosshair visualization |
| **Video Rendering** | None | Dynamic gradient trail, OpenCV HUD telemetry overlay & video export |

---

## 7. Limitations & Real-World Cricket DRS Complexity

1. **Monocular 2D vs. Calibrated 3D**:
   - Broadcast DRS uses 4 to 6 synchronized cameras calibrated with stereo epipolar geometry to reconstruct true metric $(X, Y, Z)$ coordinates in meters. A single 2D camera view suffers from projective perspective distortion.
2. **Spin and Swing Dynamics**:
   - A standard parabola models gravity. Real cricket balls experience Magnus effect (spin dip/drift) and aerodynamic boundary layer separation (conventional/reverse swing), requiring differential equation solving ($m\ddot{\mathbf{r}} = \mathbf{F}_g + \mathbf{F}_d + \mathbf{F}_L$).
3. **Cricket Law 36 (LBW Requirements)**:
   - A real LBW verdict requires checking:
     1. Did the ball pitch outside leg stump? (Pitching)
     2. Did the ball hit the pad in line with the stumps? (Impact)
     3. Would the ball have gone on to hit the wicket? (Wicket hitting)
   - These cannot be certified from a 2D geometric intersection alone.

---

## 8. Viva Voce & Lab Examination Q&A

**Q1: What computer vision algorithm is used for detection in this project?**  
*Answer*: The model utilizes a YOLO-based convolutional/anchor-free object detection architecture hosted via Roboflow Serverless API (`cricket-drs-system/4`), trained to localize cricket balls with bounding box regression and classification heads.

**Q2: Why is frame-to-frame tracking necessary instead of just running detection?**  
*Answer*: Detection on isolated frames can suffer from occlusions, motion blur, and false positives (e.g. helmet logos, white pads). Tracking associates temporal detections across time, filters out physically impossible jumps via Euclidean speed thresholds, and enables mathematical curve fitting.

**Q3: Why is a quadratic polynomial ($deg=2$) chosen for trajectory extrapolation?**  
*Answer*: Under standard Newtonian kinematics with constant gravitational acceleration $g$, the vertical position equation is $y(t) = y_0 + v_{0y}t + \frac{1}{2}gt^2$. A second-order polynomial provides the canonical approximation for ballistic flight.

**Q4: How does the system handle ball bounce on the pitch?**  
*Answer*: In image coordinates (where $Y=0$ is top), as the ball falls towards the pitch, $Y$ increases. When it strikes the pitch and rebounds upward, $Y$ decreases. The algorithm detects this local extrema and fits the trajectory model exclusively to post-bounce coordinates to prevent distorted inverted projections.

**Q5: What is "Umpire's Call" in official DRS?**  
*Answer*: It is a margin-of-error buffer (typically 50% of the ball width) applied to tracking and projection because camera resolution, frame rates (even at 340 fps), and calibration have inherent statistical uncertainty.
