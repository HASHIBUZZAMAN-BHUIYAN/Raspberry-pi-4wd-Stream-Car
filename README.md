# Raspberry Pi 4WD Stream Car

A 4WD robot car built on a Raspberry Pi 4B with a Pi Camera V2 — a Flask web app streams live video and drives the motors via GPIO, controllable from any browser on the same network.

## Board used

Raspberry Pi 4B, Pi Camera V2

## Components

| Component | Qty |
|---|---|
| Raspberry Pi 4B | 1 |
| Pi Camera V2 (CSI ribbon) | 1 |
| L298N (or similar) dual motor driver | 1 |
| DC gear motors | 4 |
| Chassis, wheels, battery pack | 1 set |

## Wiring (BCM numbering)

| Raspberry Pi GPIO (BCM) | Motor Driver Pin | Role |
|---|---|---|
| GPIO 27 | IN1 | Left motor |
| GPIO 22 | IN2 | Left motor |
| GPIO 24 | IN3 | Right motor |
| GPIO 23 | IN4 | Right motor |

Pi Camera V2 connects via the CSI ribbon connector — no GPIO pins used for video.

## How it works

`robo.py` runs a small Flask server that:
- Opens `/dev/video0` (the Pi Camera) with OpenCV and streams it as MJPEG at `/video_feed`
- Serves a control page (`index.html`) at `/`
- Accepts `POST /move` with a `command` field (`w`/`a`/`s`/`d`/`q` for forward/left/backward/right/stop) and drives the corresponding GPIO pins via `RPi.GPIO`
- Starts/stops the video feed via `/start_stream` and `/stop_stream`

The video frame is rotated 180° and mirrored before encoding — adjust `cv2.rotate`/`cv2.flip` in `robo.py` if your camera is mounted differently.

## File structure

The app expects this layout (Flask's default template lookup):

```
main/
└── rb/
    ├── robo.py           ← this repo's CODE/robo.py, run from here
    └── templates/
        └── index.html    ← this repo's INDEX/index.html
```

## Setup

1. Wire the motor driver to the GPIO pins above and connect the Pi Camera V2 to the CSI port.
2. On the Pi: `sudo raspi-config` → enable the Camera interface, then reboot.
3. Install dependencies:
   ```
   pip install flask opencv-python RPi.GPIO
   ```
4. Create the `rb/templates/` folder structure shown above, copy `CODE/robo.py` into `rb/` and `INDEX/index.html` into `rb/templates/`.
5. From the `rb/` folder, run:
   ```
   python robo.py
   ```
6. Browse to `http://<raspberry-pi-ip>:5000` from any device on the same network to view the stream and drive.

## Extensions

- Add PWM speed control instead of full-on/off GPIO driving
- Add an ultrasonic sensor for auto-braking (see the Arduino "Smart Braking" car in this account's other repos for the same idea)
- Switch the video pipeline to `picamera2` for lower latency than OpenCV's V4L2 capture
