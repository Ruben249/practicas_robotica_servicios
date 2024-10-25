# Service_Robotics_Practices
## Rescue People

![Portada](https://github.com/user-attachments/assets/4aeba2e1-0fd1-4afe-a973-f48dc1b2ca09)


### Index

-[Start](#start)

-[Development](#development)

-[Errors](#errors)

-[Videos](#videos)

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
While following the waypoints, the drone constantly analyzes images from its ventral camera. IThe image you receive looks like this:

![mar](https://github.com/user-attachments/assets/bf41472b-e18a-4d59-ae46-b6adcd05264b)

In this case it would only detect the face that looks straight and in the normal orientation of a face. That is why I decided to rotate the image

```python

# Rotate an image by a given angle
def rotate_image(image, angle):
    (h, w) = image.shape[:2]
    center = (w // 2, h // 2)
    rotation_matrix = cv2.getRotationMatrix2D(center, angle, 1.0)
    rotated_image = cv2.warpAffine(image, rotation_matrix, (w, h))
    return rotated_image
```
When a face is detected, the drone give us his coordinates.

##### Battery Management: 
The drone monitors its battery level and logs a 50% warning after 4 minutes. After 8 minutes, the drone returns to the base automatically.

#### Coordinates tranformation
For this exercise it is necessary to show the location of people at sea. Then, the Gazebo transformation has to be made with respect to altitude and latitude.

```python
if is_new_detection(pos):
    detected_people_positions.append(pos)
    people_count += 1
    utm_coords = update_utm_with_robot_position(utm_boat)
    print(f"Person {people_count} detected at UTM position X: {utm_coords[0]}, Y: {utm_coords[1]}")
```

#### Errors

-The main problem I had was that even if the drone was above a person, it didn't detect the face, because to detect the face it needs to see it at a certain angle. So my initial solution was that, using opencv, when it detects a colour other than blue, the drone starts to rotate around itself to find that angle. That solution took a long time, so I decided that instead of rotating the drone, the most practical thing to do was to rotate the image, so I changed the system.

-A very important error I had was that due to the high demand of computer resources, the image was out of phase with respect to the drone, and there were even times when the drone passed over a person, and it did not appear in the image. This has delayed the development of the exercise a lot, added to the fact that practically after each execution, the docker had to be restarted.


#### Videos
Here is the link to the video of the drone's behaviour

https://youtu.be/BdDdAW7zDIQ


In this video we can observe the behaviour of the drone with a one-minute battery:

https://youtu.be/tA4a2U7SKSI

