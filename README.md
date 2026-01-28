# SUAS 2024 – Autonomous Cargo Delivery UAV

Autonomous cargo delivery UAV system developed for the **AUVSI Student Unmanned Aerial Systems (SUAS) 2024** competition.  
The platform performs fully autonomous missions: it flies a pre-planned route, detects alphanumeric markers on the ground, executes payload drops using a motorized winch mechanism, and returns to home without manual intervention under normal conditions.

This repository documents the **system architecture, edge AI pipeline, and integration work** behind the project.

🎥 **Project Overview Video1:** [Watch on YouTube](https://youtu.be/MoKNS4MnHKc?si=5vaya1-DzLdw4j8K)

🎥 **Project Overview Video2:** [Watch on YouTube](https://youtu.be/0MTjb65SmeQ?si=WlFSHK1B2cfJrwUZ)

---

## 1. Competition Context

The **AUVSI Student Unmanned Aerial Systems (SUAS)** competition is an international collegiate drone competition that challenges teams to design and demonstrate a UAS capable of:

- Autonomous flight and navigation,
- Remote sensing using onboard payload sensors,
- Executing a set of mission tasks defined by the rulebook.

Our team participated in **SUAS 2024**, passed the required qualification stages, and submitted an official video report. We successfully demonstrated an autonomous cargo delivery mission and obtained a participation certificate for SUAS 2024.

---

## 2. Project Overview

The goal of this project was to build an **autonomous cargo delivery UAV** that can:

- Take off and fly a mission defined by uploaded waypoints,
- Detect letters and digits painted on the ground that represent drop locations,
- Stop or hold position over each detected target,
- Lower and release a payload using a motorized mechanism and fishing line (monofilament),
- Retract the line and continue to the next delivery,
- Return to the home location and land autonomously after all deliveries are completed.

All perception and decision-making for payload delivery runs **onboard** on an **NVIDIA Jetson Nano**, using a **Raspberry Pi High Quality Camera** and a **YOLOv8** model trained on custom data. The flight controller is a **Pixhawk Cube Orange**, integrated with a ground control station (Mission Planner) for mission planning, telemetry monitoring, and safety supervision.

---

## 3. System Architecture

At a high level, the system consists of the following components:

- **Airframe & Payload Mechanism**
  - Multirotor airframe configured for stable autonomous flight.
  - Cargo carried under the drone using a **fishing line** and a **motorized winch mechanism**.
  - The winch motor is driven by the flight controller, which can lower and raise the payload on command.

- **Flight Controller & Autopilot**
  - **Pixhawk Cube Orange** running a standard autopilot firmware (e.g., ArduPilot/PX4).
  - Responsible for low-level stabilization, waypoint following, and executing autonomous flight modes.
  - Provides telemetry data (position, attitude, velocity, status) to the Jetson and the ground control station.

- **Onboard Compute & Camera**
  - **NVIDIA Jetson Nano** as the onboard computer.
  - **Raspberry Pi High Quality Camera** mounted on the UAV, providing a downward-looking video stream.
  - The Jetson captures the camera feed, runs object detection in real time, and coordinates payload drop decisions with the flight controller.

- **Perception & Mission Logic**
  - A **YOLOv8** model trained to detect alphanumeric markers (letters and digits) on the ground.
  - A mission management layer running on the Jetson that:
    - Monitors detections,
    - Decides when the UAV should pause or hold over a target,
    - Triggers the cargo drop sequence via the flight controller.

- **Ground Control Station & Safety**
  - **Mission Planner** used for:
    - Uploading waypoints and defining missions,
    - Monitoring telemetry, battery, and flight status,
    - Supervising autonomous operation.
  - Manual RC override and standard failsafes (link loss, battery, geofence) are configured for safety.

---

## 4. Hardware & Software Stack

**Hardware**

- **Onboard Compute:** NVIDIA Jetson Nano (Linux-based edge AI platform).
- **Flight Controller:** Pixhawk Cube Orange (Cube-class autopilot).
- **Camera:** Raspberry Pi High Quality Camera.
- **Payload Mechanism:** Motorized winch with monofilament (fishing line) for lowering/retracting water bottle payloads.
- **Ground Segment:**
  - Ground control computer running Mission Planner.
  - RC transmitter/receiver for manual override and safety.

**Software**

- **Autopilot Firmware:** ArduPilot/PX4 (configured for autonomous multirotor flight and payload control).
- **Ground Control Station:** Mission Planner for mission upload, telemetry monitoring, and parameter configuration.
- **Computer Vision:** YOLOv8 (Ultralytics) for real-time object detection of ground markers.
- **Acceleration:** CUDA on Jetson Nano for GPU-accelerated inference.
- **Mapping:** OpenDroneMap (ODM) for generating maps/orthophotos from in-flight images.
- **Data Transfer:** FTP for transferring processed map outputs from the Jetson to the ground control station.
- **Languages & Tools:** Primarily Python (for perception and mission logic), shell scripts/systemd services for startup automation.

---

## 5. Perception & Edge AI Pipeline

The perception pipeline runs entirely on the Jetson Nano during flight:

1. **Data Collection & Model Training**
   - The team prepared a dataset of alphanumeric markers (letters and digits) painted on the ground.
   - Images were collected under varying viewpoints and lighting conditions to approximate the mission environment.
   - A **YOLOv8** model was trained on this custom dataset to detect the markers that correspond to valid drop locations.

2. **Onboard Inference**
   - The Jetson Nano captures frames from the Raspberry Pi HQ Camera.
   - The YOLOv8 model runs on the Jetson GPU using **CUDA**, providing real-time detections that can keep up with the UAV’s motion.
   - Detection outputs include bounding boxes, class labels (letter or digit), and confidence scores.

3. **Decision Logic Integration**
   - The perception node publishes detection results to the mission logic layer.
   - Mission logic evaluates:
     - Whether a detection corresponds to a valid drop marker,
     - The stability and persistence of the detection (e.g., ensuring it is not a transient false positive),
     - The current mission state (en route, drop pending, drop complete, returning home, etc.).
   - When conditions are met, the mission logic initiates the **cargo drop sequence** via the flight controller.

All of this runs **onboard**, without cloud connectivity or offboard inference. The system is a real deployment of edge AI on an embedded platform in a live UAV environment.

---

## 6. Autonomous Mission & Cargo Logic

The autonomous mission is structured as follows:

1. **Mission Start**
   - Operators upload a waypoint mission to the flight controller via Mission Planner.
   - After standard pre-flight checks, the UAV is armed and switched to an autonomous flight mode.
   - The Jetson has already booted and started its services; it is ready to monitor the camera feed and telemetry.

2. **En Route Flight**
   - The UAV follows the predefined waypoint route under the control of the autopilot.
   - The Jetson continuously processes camera frames with the YOLOv8 detector.

3. **Target Detection & Hold**
   - When a letter/digit corresponding to a drop location is detected:
     - The Jetson computes or requests the relevant position from the autopilot.
     - The Jetson commands the autopilot (e.g., through a MAVLink interface) to slow down or hold over the target area (e.g., entering a loiter or position-hold behavior).

4. **Cargo Drop Sequence**
   - Once the UAV is stabilized over the target:
     - The Jetson issues a command to the flight controller to activate the payload motor output.
     - The motor unwinds the fishing line, lowering the payload (water bottle) to the ground.
     - The mechanism releases the payload at the appropriate point.
     - The motor then reels the line back in, retracting the monofilament so that it is clear for further flight.

5. **Mission Continuation**
   - After a successful drop sequence, the Jetson tells the autopilot to resume the mission.
   - The UAV proceeds to the next targets and repeats the detect–hold–drop cycle as needed.

6. **Return to Home**
   - When all deliveries are completed (or the mission is otherwise finished), the UAV returns to the home location and lands autonomously under autopilot control.

From the operator’s perspective, after arming and starting the mission, the entire detection and cargo logic runs autonomously, with manual intervention reserved for safety or abnormal situations.

---

## 7. Safety, Failsafes & Ground Control

Although the system is highly autonomous, safety is treated as a first-class requirement:

- **Ground Control Station (GCS)**
  - Mission Planner is used to:
    - Upload missions,
    - Monitor real-time telemetry (position, battery, flight mode, status),
    - Observe mission progress and payload events.

- **Failsafes**
  - Standard failsafes are configured on the autopilot, such as:
    - Loss-of-signal / link-loss behavior,
    - Low-battery actions,
    - Geofencing to restrict the operational area.
  - These settings ensure that the UAV will attempt to enter a safe state if communication is lost or other critical conditions arise.

- **Manual Override**
  - A radio controller is available for manual takeover.
  - Operators can switch out of autonomous mode, disarm the system, or command a manual landing if required.

- **Jetson Services & Control**
  - The Jetson starts its services automatically on boot (camera capture, perception, mission logic, telemetry links).
  - For debugging or emergency situations, operators can still stop or reconfigure services as needed.

This arrangement ensures that the system can operate autonomously in normal conditions while maintaining clear safety backstops.

---

## 8. Data Logging, Mapping & FTP Workflow

Beyond executing the cargo mission, the system also implements an end-to-end data pipeline:

1. **Video Recording**
   - While the camera is active, the Jetson records video of the flight.
   - This footage is useful for:
     - Post-flight analysis,
     - Reviewing detection events and payload drops,
     - Building training or validation datasets.

2. **Still Image Capture**
   - In addition to continuous video, the system periodically captures still images during flight.
   - These images cover the cargo drop area and surrounding terrain.

3. **Mapping with OpenDroneMap (ODM)**
   - After the flight, the still images are processed with **OpenDroneMap (ODM)** to generate a map (e.g., an orthophoto) of the mission area and drop locations.
   - ODM converts the aerial imagery into georeferenced outputs that can be inspected and overlaid with other geographic data.

4. **FTP Transfer**
   - Once processing is complete and the UAV has landed, the Jetson transfers the generated map products to the ground control station using **FTP**.
   - This provides a clean workflow from in-flight data capture to ground-side analysis and visualization.

---

## 9. What This Project Demonstrates

This project demonstrates the ability to:

- Deploy a **custom-trained YOLOv8 model** on an **NVIDIA Jetson Nano** and run it in real time during flight.
- Integrate edge AI perception with a **Cube-class flight controller** (Pixhawk Cube Orange) for closed-loop decision making.
- Design and implement a **mission logic layer** that links high-level events (marker detection) to flight behaviors (hold/loiter, resume mission) and actuator control (payload motor).
- Work within the **constraints and safety requirements** of a UAV competition environment.
- Build an **end-to-end data pipeline**: capture > record > map with ODM > transfer results via FTP.

From a hiring perspective, this repository is evidence of hands-on experience in **edge AI, UAV integration, real-time systems, and field testing** at a serious student project level.

---

## 10. My Role & Contributions

I managed and captained the project.
My contributions to this project are summarized as follows:

- **Edge AI and Detection**
- I led the preparation of the custom dataset for alphanumeric markers.
- I managed the training and deployment of the YOLOv8 model on Jetson Nano.
- I developed the real-time inference pipeline from camera stream to detection output.

- **Edge AI performance gain and Model Optimization**
- CUDA activation, model optimization, GPU-CPU usage optimization.

- **Jetson–Autopilot Integration**
  - Implementing the communication between the Jetson and the Pixhawk Cube Orange (e.g., via MAVLink).
  - Designing and implementing parts of the mission logic that convert detections into flight and payload commands.
  - Developing startup scripts/services so that, when power is applied to the Jetson, all required components (telemetry links, camera, perception, mission logic) start automatically.

- **Cargo Drop Logic**
  - Integrating the payload motor control into the autonomy stack so that the flight controller can drive the motor for lowering and retracting the payload based on commands from the Jetson.

- **Data Pipeline & Mapping**
  - Setting up the workflow for periodic still image capture during flight.
  - Using OpenDroneMap (ODM) to generate maps of the cargo drop area from captured images.
  - Implementing the FTP transfer of mapping outputs from the Jetson to the ground control station after landing.

- **Testing & Documentation**
  - Supporting field tests, analyzing behavior, and iterating on detection thresholds and mission parameters.
  - Contributing to the SUAS 2024 video report and technical documentation.

---

### Additional Information

* **Developer**: [Fatih AYIBASAN] (Computer Engineering Student)
* **Email**: [fathaybasn@gmail.com]

---
