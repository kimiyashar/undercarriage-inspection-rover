# Camera-to-controller interface

The current simulator separates perception from control. A Python/OpenCV process will convert real camera images and AprilTag detections into the same structured observations that the C++ controller already consumes.

## `CameraFrame`

Each frame contains:

| Field | Meaning |
|---|---|
| `frameNumber` | Monotonic frame identifier |
| `valid` | Whether the frame is current and usable |
| `pathVisible` | Whether the planned inspection route is visible |
| `pathOffset` | Horizontal route offset; negative is left, positive is right |
| `obstacleDetected` | Camera-level obstacle signal |
| `bumperPressed` | Physical safety signal, currently simulated |
| `tags` | Zero or more AprilTag detections |

## `TagDetection`

| Field | Meaning |
|---|---|
| `id` | AprilTag identifier |
| `angleDeg` | Horizontal target angle; negative is left |
| `distanceM` | Estimated camera-to-tag distance in meters |
| `confidence` | Detection confidence from 0 to 1 |

## Planned real-camera adapter

The planned Python/OpenCV component will:

1. Capture a camera frame on the embedded computer.
2. Detect AprilTags and estimate their relative poses.
3. Estimate route position and detect obstacles.
4. Serialize a `CameraFrame` observation.
5. Send the observation to the C++ controller.
6. Receive a high-level action such as `TURN_LEFT` or `STOP_AT_TARGET`.
7. Capture localized undercarriage imagery at inspection waypoints.
8. Run or feed an on-device defect detector for corrosion, fluid leaks, damaged components, or other visible anomalies.

The Python-to-C++ transport has not been selected or implemented yet. Candidate options include newline-delimited JSON over a local process pipe, a lightweight socket, or direct C++ camera integration.

## Safety note

AprilTags are localization landmarks, not physical protection. A rover operating beneath a vehicle requires independent bumpers or proximity sensing, a hardware-level stop path, verified ground clearance, and supervised testing before autonomous operation.