# Autonomous Vehicle Navigation Simulation

This project is a Python-based simulation of an autonomous vehicle's perception and path-planning system. It processes a forward-facing dashcam video to detect and track other vehicles, transform the scene into a top-down Bird's-Eye View (BEV), and plan a safe path that avoids collisions.

![Example Output](https://github.com/user-attachments/assets/93521a1a-a6ba-4b96-9efb-2f5d4e285db9)
*(Note: The GIF above is an example of the expected output.)*

---

## Table of Contents
- [Key Features](#key-features)
- [How It Works](#how-it-works)
- [Project Structure](#project-structure)
- [Technologies Used](#technologies-used)
- [Setup and Installation](#setup-and-installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [Future Work](#future-work)

---

## Key Features

- **Vehicle Detection**: Utilizes the YOLOv8 object detection model to identify cars, trucks, and buses in video frames.
- **Object Tracking**: Implements an Extended Kalman Filter (EKF) to track detected vehicles, assigning persistent IDs and smoothing their positions over time.
- **Bird's-Eye View (BEV) Transformation**: Converts the driver's perspective into a top-down view for simplified path planning and spatial reasoning.
- **Path Planning**: Generates a primary (global) path for the vehicle to follow.
- **Collision Avoidance**: Implements a reactive local planner that detects potential collisions with tracked vehicles and generates a simple lateral avoidance maneuver.
- **Comprehensive Visualization**: Renders the vehicle's perception and decisions, including bounding boxes, the BEV, and the projected path on the road.

## How It Works

The system operates as a modular pipeline that processes video frame-by-frame:

1.  **Perception**:
    - The `Detector` uses YOLOv8 to find vehicles in the current frame.
    - The `EKFTracker` takes these detections, associates them with existing tracks, and updates their state (position, velocity) using a Kalman Filter. This provides a stable understanding of the environment.

2.  **World Modeling**:
    - The `PerspectiveTransformer` takes the tracked vehicle positions and remaps them from the 2D camera view to a 2D top-down Bird's-Eye View. This geometric transformation is crucial for planning in a simplified, metrically-accurate space.

3.  **Planning**:
    - The `PathPlanner` defines the ideal, long-term path for the ego-vehicle (a straight line in this simulation).
    - The `CollisionAvoider` checks if this ideal path is obstructed by any tracked vehicles in the BEV. If a collision is likely, it generates a new, temporary avoidance path by shifting laterally.

4.  **Visualization**:
    - The `PathRenderer` draws all the visual information: the region of interest (ROI), tracked vehicle bounding boxes, the BEV representation, and the final projected path onto the road.

## Project Structure

```
autonomous_navigation_project/
├── navigation/
│   ├── collision_avoider.py   # Local planner for reactive obstacle avoidance
│   └── path_planner.py        # Global path definition
├── object_processing/
│   ├── detector.py            # YOLOv8 object detection
│   └── tracker.py             # EKF-based object tracking
├── visualization/
│   ├── perspective.py         # Bird's-Eye-View transformation logic
│   └── renderer.py            # Renders all visual elements
├── config.py                  # Central configuration file for all parameters
├── main.py                    # Main script to run the simulation
├── requirements.txt           # Project dependencies
├── download_assets.py         # Helper script to download the sample video
└── calibrate_perspective.py   # Helper script to find perspective points for a new video
```

## Technologies Used

- **Python 3.9+**
- **OpenCV**: For video processing and drawing operations.
- **Ultralytics YOLOv8**: For real-time object detection.
- **NumPy**: For numerical operations and array manipulation.
- **SciPy**: Used for the Hungarian algorithm in the tracker.
- **FilterPy**: Provides the Kalman Filter implementation.

---

## Setup and Installation

Follow these steps to get the project running on your local machine.

**1. Clone the Repository**
```bash
git clone <your-repository-url>
cd autonomous_navigation_project
```

**2. Create and Activate a Virtual Environment**
```bash
# Windows
python -m venv .venv
.venv\Scripts\activate

# macOS / Linux
python3 -m venv .venv
source .venv/bin/activate
```

**3. Install Dependencies**
Install all the required Python packages using the `requirements.txt` file.
```bash
pip install -r requirements.txt
```

**4. Download Assets**
The project requires a sample video file. Run the provided helper script to download it automatically.
```bash
python download_assets.py
```
This will download `sample_video.mp4` into the project's root directory.

---

## Usage

The project has two main entry points: one for calibrating the perspective on a new video and one for running the main simulation.

### 1. (Optional) Calibrate for a New Video

If you want to use your own video, you must first find the correct perspective transformation points.

1.  Replace `sample_video.mp4` with your video file (ensure it's also named `sample_video.mp4` or update `config.py`).
2.  Run the calibration script:
    ```bash
    python calibrate_perspective.py
    ```
3.  A window will appear with the first frame of your video. Click on the four corners of the lane in front of the car in the following order: **Top-Left, Top-Right, Bottom-Right, Bottom-Left**.

```bash
          |                   |
          |    (Horizon)      |
          |                   |
          '  1-------2  '      <-- The top edge of your rectangle on the road
         /    (Top-L) (Top-R)   \
        /                       \
       /                         \
      /                           \
     /                             \
    '4---------------------------3'  <-- The bottom edge of your rectangle
   (Bot-L)                     (Bot-R)

[ Your Car's Position is Here at the Bottom ]
```

4.  The script will print a `SRC_POINTS` array to the console.
5.  Copy this array and paste it into `config.py`, replacing the existing `SRC_POINTS` value.

### 2. Run the Main Simulation

To run the full perception and planning pipeline, execute the `main.py` script.

```bash
python main.py
```

The script will process `sample_video.mp4`, display the annotated video in real-time, and save the final output to `output_video.avi`. Press `q` to stop the simulation at any time.

---

## Configuration

All key parameters for the simulation are centralized in `config.py`. You can easily modify this file to tune the system's behavior:

- **YOLO Settings**: Adjust the confidence threshold or target classes.
- **Perspective Transformation**: Change `SRC_POINTS` and `DST_POINTS` to alter the BEV.
- **Path Planning**: Modify vehicle dimensions, safe distances, and the avoidance maneuver distance.

---

## Future Work

This project provides a solid foundation that can be extended in many ways:

- **Advanced Path Planning**: Replace the simple straight-line path with a more dynamic planner that uses lane detection.
- **Behavioral Logic**: Implement more complex driving behaviors, such as slowing down behind vehicles in addition to lane changes.
- **Sensor Fusion**: Incorporate other sensor data, like LiDAR or RADAR, for more robust perception.
- **Control System**: Add a simulated vehicle controller (e.g., a PID controller) to execute the planned path.