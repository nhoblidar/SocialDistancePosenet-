# 🚶‍♂️🚶‍♀️ Social Distancing Monitor using PoseNet (Jetson Nano)

This project uses **NVIDIA Jetson Nano** and **PoseNet** to detect people in video frames and check if they follow social distancing rules. It calculates the **distance between people in meters**, draws **green (safe)** and **red (danger)** lines, and shows bounding boxes with **SAFE / DANGER** labels.

The code also includes **smart center-point fallback logic**:
- Hip midpoint (best)
- Shoulder midpoint (fallback)
- Bounding box midpoint (last fallback)

So detection always works even when some keypoints are missing.

---

# Features

-  Detects multiple people in real time  
-  Computes actual distance (meters), not just pixels  
-  Hip midpoint → Shoulder midpoint → Bounding box fallback system  
-  Red lines for people too close (<2m)  
-  Green lines for safe distance  
-  SAFE / DANGER labels on boxes  
-  Fully works on Jetson Nano (dusty-nv jetson-inference)  
-  Supports camera, image, and video input  

---

# ⚙️ Installation (Jetson Nano)

Install Jetson Inference:


```bash
git clone --recursive https://github.com/dusty-nv/jetson-inference
cd jetson-inference
cd build/aarch64/bin
```

# How to Run

a) Webcam
```bash
python3 posenet_social_distance.py /dev/video0 display://0
```
b) Video file
```bash
python3 posenet_social_distance.py sample.mp4 output.mp4
```
c) Image
```bash
python3 posenet_social_distance.py input.jpg output.jpg
```

# How Distance Estimation Works
Center point detection logic

We try centers in this order:

(A) Hip midpoint → Most stable

If both hip keypoints exist:
```bash
center = midpoint(left_hip, right_hip)
```

(B) Shoulder midpoint → If hips missing
If hips not detected but shoulders exist:

```bash
center = midpoint(left_shoulder, right_shoulder)
```

(C) Bounding box midpoint → Final fallback

If NO keypoints exist:

```bash
center = midpoint(bbox)
```

This ensures:

a) No person is ever "lost"

b) No crash due to NoneType keypoints

# How Distance Estimation Works

Distance between two people cannot be directly calculated from pixels, so an estimation process is used.

## Pixel Distance
The Euclidean distance in pixel space is computed:
```python
pixel_dist = math.dist((x1, y1), (x2, y2))
```

## Estimate Person Height in Pixels
The height of the bounding box gives a person’s height in pixels:
```python
height_px = bottom - top
```
To avoid division errors, the minimum height is clamped.

## Real Height Assumption
An average adult height is assumed:
```python
person_height_meters = 1.7
```

## Convert Pixels to Meters
Meters per pixel:
```python
meters_per_pixel = 1.7 / height_px
```

Final distance:
```python
distance_meters = pixel_dist * meters_per_pixel
```

## Violation Rule
If:
```python
distance_meters < 2.0
```
Then it is marked as DANGER.  
Otherwise SAFE.

This method is approximate but effective for social-distance analysis on Jetson Nano.

---

# Bounding Box Logic

PoseNet outputs bounding box coordinates for each person:

Left  
Top  
Right  
Bottom  

The program uses these to:

- Draw green or red rectangles around people  
- Label the person with ID and SAFE or DANGER  
- Compute fallback midpoint when keypoints are missing  
- Estimate height for pixel-to-meter conversion  

Bounding boxes ensure that distance measurement always works, even when keypoints fail.

---

# Line Drawing Logic

For every pair of people:

- If they violate distance rule, draw a red line  
- If they do not violate, draw a green line  

Each person maintains their status until the next frame.

This makes it easy to identify violations visually.

---

# Performance Notes

Running on Jetson Nano:

- FPS depends on model and resolution  
- resnet18-body is lighter and faster than larger body models  
- USB cameras often perform better than CSI cameras for this script  
- Keeping resolution around 640x480 improves real-time performance  

---

# Limitations

- Distance estimation is approximate and depends on camera angle  
- Overhead or extreme camera angles reduce accuracy  
- Heavy occlusion may cause incorrect center point detection  
- Depth estimation is not used; project uses 2D estimation only  
- Assumes average human height; may vary slightly per person  

---

# Future Improvements

Possible enhancements:

- Real depth sensors support  
- YOLO-based person detection combined with PoseNet keypoints  
- Person tracking across frames using SORT or DeepSORT  
- Improved distance calibration using known objects  
- Heatmaps of movement patterns  

---

# Full Code Reference

Place your full Python script in this section or maintain it in the repository separately.  
The README explains the logic behind the script in detail.

---

# Reference Projects

https://github.com/dusty-nv/jetson-inference  

---

# Summary

This project uses PoseNet to detect people and compute approximate distances between them.  
The center-point fallback system ensures that missing keypoints do not break the program.  
Bounding boxes, midpoints, pixel-to-meter conversion, and real-time violation detection together create a working social-distance monitoring system that runs efficiently on the Jetson Nano.

