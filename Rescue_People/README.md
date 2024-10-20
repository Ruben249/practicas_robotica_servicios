# Service_Robotics_Practices
## Rescue People

![Portada](https://github.com/user-attachments/assets/4aeba2e1-0fd1-4afe-a973-f48dc1b2ca09)


### Index

-[Start](#start)


-[Development](#development)


-[Errors](#errors)

-[Video](#video)

#### Start

To start with this practice, the first thing to do is to get the drone off the ground. Afterwards, we send it to the centre of the accident and make it do spirals so that it detects faces and tells us its position.

The drone uses OpenCV to convert the image to grayscale, detects faces using a pre-trained Haar Cascade classifier, and logs the coordinates of the detection.

#### Development

This practice is divided into two major components: navigation and face detection.

##### Navigation: 
The drone navigates using a spiral pattern around a defined center. The following function generates waypoints for the spiral:

```python

def generate_spiral_waypoints(center_x, center_y, initial_radius, waypoint_count):
    waypoints = []
    for turn in range(waypoint_count):
        angle = 2 * math.pi * turn / 8
        radius = initial_radius - (initial_radius / waypoint_count) * turn
        waypoint_x = center_x + radius * math.cos(angle)
        waypoint_y = center_y + radius * math.sin(angle)
        waypoints.append((waypoint_x, waypoint_y))
    return waypoints
```
The drone follows these waypoints to cover the search area effectively. Once all waypoints are visited, it returns to the base.

##### Face Detection:
While following the waypoints, the drone constantly analyzes images from its ventral camera. If it detects non-blue pixels (anything other than the sky), it rotates to search for human faces:

```python

def rotate_until_face_detected():
    print("Rotating to search for a face...")
    while True:
        HAL.set_cmd_yaw_rate(30)
        camera_image = HAL.get_ventral_image()
        drone_position = HAL.get_position()
        if detect_human_faces(camera_image, drone_position):
            HAL.set_cmd_yaw_rate(0)
            print("Face detected, stopping rotation.")
            return
```
When a face is detected, the drone stops rotating and logs the face detection along with its coordinates.

##### Battery Management: 
The drone monitors its battery level and logs a 50% warning after 4 minutes. After 8 minutes, the drone returns to the base automatically.

#### Errors

-The main problem I had was that even if the drone was above a person, it didn't detect the face, because to detect the face it needs to see it at a certain angle. So my solution has been that, using opencv, when it detects a colour other than blue, the drone starts to rotate on itself to find that angle. 


#### Video 
Here is a video where we briefly see how the drone behaves.

[FollowLine_Simple.webm](https://github.com/Ruben249/practicas_robotica_movil/assets/102288264/2c9a9c66-92af-472b-bf55-7fa6d0fcbe95)

