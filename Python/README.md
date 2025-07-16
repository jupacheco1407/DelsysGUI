# EMG Force Monitoring System - GUI Interface

A comprehensive graphical interface for real-time EMG sensor monitoring with muscle activation analysis.

## Features

- 🖥️ Modern Qt-based GUI with dark theme
- 🔄 Multi-stage workflow: Connect → Configure → Calibrate → Monitor
- 📊 Real-time force visualization with color-coded progress bars
- 💪 Muscle/sensor mapping configuration
- 🚨 Incorrect activation detection and alerts
- 📈 Calibration system for personalized force measurement

## Installation

### Prerequisites
- Python 3.8+
- Trigno Base system and sensors
- AeroPy library

### Steps
1. Clone the repository:
2. 
Install dependencies:

bash
pip install PySide6 AeroPy
Run the application:

bash
python GUI.Final.py
### Usage Guide
## 1. Connection Phase

Click "Connect" to initialize Trigno Base connection

System will automatically detect available EMG sensors

Status message will confirm successful connection

## 2. Configuration Phase

Enter movement name (e.g., "Bicep Curl")

For each sensor:

Map to muscle name (e.g., "Biceps Brachii")

Set activation expectation:

✅ "Expected" for primary movers

❌ "Not Expected" for muscles that should remain inactive

For "Not Expected" muscles, select reference sensor

Click "Continue" to proceed

## 3. Calibration Phase
Follow on-screen instructions for each sensor:

Click "Start Calibration"

Apply maximum force for 5 seconds

System captures peak values

Progress automatically advances to next sensor

All sensors calibrated message appears when complete

## 4. Monitoring Phase

Click "Start Monitoring" to begin

Real-time feedback includes:

Force percentage bars (0-100%)

Color-coded activation levels

Warning messages for incorrect patterns

Global alert bar shows significant deviations

Understanding the Visual Feedback
Color	Meaning
🟢 Dark Green	High expected activation (>70%)
🟢 Medium Green	Moderate expected activation (>40%)
🟢 Light Green	Low expected activation
🟡 Yellow	Moderate unexpected activation
🔴 Red	Significant unexpected activation

PySide6 >= 6.4.0

AeroPy >= 1.2.0
