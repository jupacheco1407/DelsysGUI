# EMG Sensor Monitoring GUI

A comprehensive graphical interface for real-time EMG sensor monitoring and muscle activation analysis, developed using the Delsys API and built with PySide6 (Qt). This GUI provides an intuitive and structured workflow from sensor connection to real-time force monitoring, tailored for biomechanical and physiological research.

---

## Features

- Modern Qt-based GUI with dark theme  
- Multi-stage workflow: Connect → Configure → Calibrate → Monitor  
- Real-time force visualization with color-coded progress bars  
- Muscle/sensor mapping for custom labeling  
- Automatic detection of incorrect activations and alerts  
- Personalized calibration system for accurate force measurement  

---

## Installation

### Prerequisites

- Python 3.8+
- Delsys Trigno Base System and EMG Sensors
- Delsys AeroPy Library

### Steps

1. Clone the repository
2. Install dependencies 
3. Run the application -> GUI.py
   Note: Make sure the Delsys API is installed and properly configured on your machine.

## Usage Guide

### 1. Connection Phase

- Click **"Connect"** to initialize the Trigno Base system.
- The system will automatically detect connected EMG sensors.
- A status message will confirm successful connection.

### 2. Configuration Phase

- Enter the movement name (e.g., `Bicep Curl`).
- For each sensor:
  - Assign a muscle name (e.g., `Biceps Brachii`).
  - Define activation expectation:
    - **Expected** for muscles that should be active.
    - **Not Expected** for muscles that should remain inactive.
- For "Not Expected" muscles, select a **reference sensor**.
- Click **"Continue"** to proceed.

### 3. Calibration Phase

- Follow the on-screen instructions:
  - Click **"Start Calibration"** for each sensor.
  - Apply maximum voluntary contraction for approximately **5 seconds**.
  - The system will capture the peak force and automatically proceed to the next sensor.
- A message will confirm when all sensors have been calibrated.

### 4. Monitoring Phase

- Click **"Start Monitoring"** to begin real-time feedback.
- The interface will display:
  - Force percentage bars (**0–100%**)
  - Color-coded activation levels
  - Warnings for incorrect activation patterns
  - A global alert bar for significant deviations
