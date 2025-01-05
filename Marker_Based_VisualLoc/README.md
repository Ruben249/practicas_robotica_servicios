# Service_Robotics_Practices
## Marker Based Visual Loc

![Cover](https://github.com/user-attachments/assets/26251d5e-dfc8-4b22-a9da-81d06bafd0d2)

---

### Index

- [Start](#start)
- [Development](#development)
- [Errors](#errors)
- [Video](#video)

---

#### Start

In this project, we implemented **visual localization using markers** (AprilTags) to calculate the position and orientation of a mobile robot in a 2D space. The process begins with the definition of **essential matrices** and setting up the system to detect markers and navigate using pseudo-random movements.

##### Initial Setup

The camera’s intrinsic matrix and a predefined transformation from the camera to the robot are defined. Additionally, a YAML file provides the world positions of the AprilTags.

```python
# Camera intrinsic matrix
matrix_camera = np.array(
    [[1, 0, 0.069],
     [0, 1, -0.047],
     [0, 0, 1]],
    dtype="double",
)

# Transformation from camera to robot
matrix_camera_to_robot = np.array([
    [1, 0, 0, 0.069],
    [0, 1, 0, -0.047],
    [0, 0, 1, -0.107],
    [0, 0, 0, 1]
])
