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
```
##### Pseudo-random Navigation

A simple navigation mechanism was implemented, allowing the robot to move forward and rotate randomly, exploring the environment:
```python
def random_navigation():
    HAL.setV(0.2)  # Constant linear speed
    random_angular_velocity = random.uniform(-1.0, 1.0)  # Random angular speed
    HAL.setW(random_angular_velocity)
```
#### Development
The primary development focused on improving the calculation of the robot's estimated pose using detected AprilTags.
##### 1. Detecting AprilTags

The pyapriltags library was used to detect markers in the image captured by the camera. Detection provides the marker's corners, its center, and its unique identifier.
```python
results = detector.detect(gray)  # AprilTags detection
```

##### 2. Calculating the Robot’s Position

Using the detected corners, we solved the Perspective-n-Point (PnP) problem to determine the transformation between the marker and the camera. This enabled calculating the robot's position in the world.
```python
success, rvec, tvec = cv2.solvePnP(
    object_points, image_points, matrix_camera, dist_coeffs,
    flags=cv2.SOLVEPNP_IPPE_SQUARE
)
```
By combining transformations from the world to the marker and from the marker to the camera, the robot's position and orientation in the space were calculated.
```python
world_to_robot = np.matmul(np.matmul(world_to_marker, marker_to_camera), matrix_camera_to_robot)
x = world_to_robot[0, 3]
y = world_to_robot[1, 3]
```
##### 3. Fusion of Multiple Markers

When multiple markers were visible, the system fused their individual estimations. A weight inversely proportional to the distance to the marker was assigned, prioritizing closer markers for more accurate calculations.
```python3
# Fusing multiple detections
total_weight = sum(p[3] for p in poses)
x_fused = sum(p[0] * p[3] for p in poses) / total_weight
y_fused = sum(p[1] * p[3] for p in poses) / total_weight
yaw_fused = math.atan2(
    sum(math.sin(p[2]) * p[3] for p in poses) / total_weight,
    sum(math.cos(p[2]) * p[3] for p in poses) / total_weight
)
```

#### Errors

-Initially, the intrinsic camera matrix and transformation matrices lacked proper dimensions and alignment. This caused issues in solving the **Perspective-n-Point (PnP)** problem, as the inputs were not consistent with the expected format. Debugging these definitions was crucial for accurately determining the pose.

- One problem to take into account is that when we do not detect any airtag, we have to use odometry to know where we are, making the difference between the point where we were before and where we are now.

-Another problem was that at the beginning it did not calculate the orientation properly, so I had to add pi/2 to it.

-One thing I have implemented is to add more weight to the nearest tag to be used to calculate the position, as when it detected more than one it got worse.

#### Video
