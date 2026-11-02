# Vision-Guided Undercarriage Inspection Rover

A work-in-progress autonomous rover designed to travel beneath parked vehicles, localize itself with AprilTags, and collect repeatable imagery for undercarriage inspection. The current milestone is a tested C++ perception-to-control simulator that converts processed camera and marker observations into safety-prioritized navigation actions.

> **Current scope:** C++ control and CSV-simulated perception are implemented. Real Python/OpenCV camera processing, AprilTag pose estimation, on-device defect detection, embedded deployment, and physical hardware integration are in development or planned.

## Current status

### Implemented

- C++17 perception-to-control system
- Inspection waypoint, dock, boundary, and unknown AprilTag classification
- Safety-prioritized navigation logic
- Multiple AprilTags in one camera frame
- CSV replay of simulated camera and marker observations
- Controller and CSV parser tests
- Makefile build commands
- GitHub Actions build and test workflow

### In development / planned

- Python/OpenCV camera pipeline
- Real AprilTag detection and pose estimation
- On-device undercarriage defect detection
- Embedded Linux deployment
- Motor, bumper, encoder, and lighting integration
- Repeatable inspection-path execution
- Image capture linked to vehicle position and inspection waypoint

## System architecture

```text
CURRENT
simulated_camera_data.csv
          |
          v
   C++ CSV frame loader
          |
          v
   C++ rover controller
          |
          v
 navigation action + reason

PLANNED
camera -> Python/OpenCV perception -> C++ controller -> motors
                       |
                       +-> localized inspection images + defect candidates
```

The perception interface is documented in [`docs/camera-frame-interface.md`](docs/camera-frame-interface.md). The staged physical build is described in [`docs/hardware-bom.md`](docs/hardware-bom.md).

## AprilTag scheme

| IDs | Meaning |
|---|---|
| `1-99` | Inspection waypoints beneath or around a vehicle |
| `500` | Home dock |
| `900-999` | Safety boundaries |
| Anything else | Unknown and ignored |

AprilTags are printed visual markers read by a camera. They provide stable IDs and, with pose estimation, relative position and orientation. They are not RFID tags.

## Controller decisions

The controller can return:

- `TURN_LEFT`
- `TURN_RIGHT`
- `DRIVE_STRAIGHT`
- `STOP_AT_TARGET`
- `HOLD_POSITION`
- `EMERGENCY_STOP`
- `SEARCH_FOR_ROUTE`

Safety always overrides navigation:

1. Invalid or stale camera frame
2. Bumper activation
3. Camera-detected obstacle
4. Confident boundary AprilTag
5. Correct inspection waypoint
6. Visible route
7. Lost route

If an inspection waypoint and boundary marker are visible simultaneously, the boundary wins.

## Build and run

### Requirements

- C++17 compiler
- `make`

```bash
make
./rover_sim simulated_camera_data.csv 2
```

The final argument is the target AprilTag ID. In this example, the rover navigates toward inspection waypoint 2.

Example output:

```text
Frame 4: TURN_LEFT | target AprilTag is left of center
Frame 7: STOP_AT_TARGET | correct target is centered and within stopping distance
Frame 8: EMERGENCY_STOP | boundary AprilTag visible
Frame 11: SEARCH_FOR_ROUTE | neither target nor route is visible
```

## Run tests

```bash
make test
```

Expected result:

```text
PASS: all controller decision situations
PASS: CSV camera simulation loader
```

## Simulated perception data

`simulated_camera_data.csv` contains one or more observations per frame:

```text
frame,valid,path_visible,path_offset,obstacle,bumper,tag_id,tag_angle_deg,tag_distance_m,tag_confidence
```

Multiple rows with the same frame number represent multiple AprilTags visible in one frame. A `tag_id` of `-1` means no marker was detected.

The sample sequence covers route following, correct and incorrect inspection waypoints, boundary priority, obstacles, bumper activation, invalid frames, lost routes, weak detections, and unknown tags. Dock behavior is covered by the controller tests.

## Physical safety limitation

AprilTag boundaries are semantic landmarks, not continuous barriers. A physical rover also needs independent proximity sensing, bumpers, a hardware stop path, and appropriate clearance checks before entering or moving beneath a vehicle.

## Next milestone

Build a Python/OpenCV adapter that captures real camera frames, estimates AprilTag poses, and produces the same `CameraFrame` fields currently supplied by the CSV simulator. Then connect localized imagery to an on-device defect-detection pipeline.