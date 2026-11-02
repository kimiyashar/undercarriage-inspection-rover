# Physical Inspection Rover Plan

This staged plan converts the software-only controller into a compact vision-guided rover for scanning vehicle undercarriages.

> **First physical milestone:** drive along a controlled test route, localize one inspection waypoint with an AprilTag, stop safely, capture a repeatable image, and return to the home marker.

## Recommended architecture

```text
Embedded Linux computer
  Python + OpenCV + AprilTag pose estimation + defect detection
                              |
                        serial / CAN
                              v
Low-level motor controller
  C++ motor control + encoders + hardware safety inputs
                              |
                     motors / bumpers
```

The embedded computer handles images, localization, logging, and high-level perception. A microcontroller should handle timing-sensitive motor control and remain able to stop the rover if vision or host communication fails.

## Major subsystems

### Mobility

- Low-profile differential-drive chassis sized for expected vehicle clearance
- Geared motors and wheel encoders
- Motor controller with an independent emergency-stop path
- Protected battery and accessible master power switch

### Perception

- Forward navigation camera
- Upward-facing inspection camera with controlled lighting
- AprilTags for repeatable localization and inspection waypoints
- Python/OpenCV for marker detection and pose estimation
- On-device model for candidate defect detection

### Safety

- Front and side bumpers
- Short-range proximity or time-of-flight sensors
- Tilt or lift detection
- Hardware motor kill switch
- Supervised operation and a controlled test fixture before use beneath a real vehicle

### Inspection data

Each captured image should be associated with:

- Timestamp
- Vehicle or test identifier
- Inspection waypoint/tag ID
- Estimated camera pose
- Image-quality status
- Defect candidates and confidence

## Staged build

1. **Simulation:** validate controller decisions with CSV camera and marker observations.
2. **Bench perception:** detect real AprilTags and estimate pose from recorded and live camera frames.
3. **Mobile base:** validate encoder-controlled driving and independent emergency stops.
4. **Integrated localization:** navigate to one physical inspection waypoint and stop repeatably.
5. **Inspection capture:** collect illuminated undercarriage images tied to waypoint and pose.
6. **Defect detection:** evaluate an on-device model against labeled examples and preserve source images for human review.

## Safety limitation

The current repository does not yet control physical hardware. A real undercarriage rover must be tested with the vehicle secured, engine off, parking brake engaged, wheels appropriately chocked, and no person beneath the vehicle. Computer vision must not be the only collision or stop mechanism.