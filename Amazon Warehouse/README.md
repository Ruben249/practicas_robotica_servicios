# Service_Robotics_Practices
## Amazon Warehouse 

![Portada](https://github.com/user-attachments/assets/1df065f8-d6d0-4e04-a78c-35a273cc4861)



### Index

-[Start](#start)

-[Development](#development)

-[Errors](#errors)

-[Videos](#videos)

### Start


In this project, we need to convert between **pixel coordinates** on a 2D image (representing the warehouse map) and **world coordinates** (measured in meters) to accurately navigate the warehouse. The center of the map corresponds to the origin `(0, 0)` in world coordinates, with the positive y-axis pointing upward and the positive x-axis pointing to the left.

The problem statement provides the **real-world dimensions** of the warehouse:
- **Warehouse Width**: 20.62 meters
- **Warehouse Height**: 13.6 meters

These values are stored as constants and allow us to calculate the **scaling factors** to translate between pixels and meters.

##### Step-by-Step Conversion Process

1. **Map Dimensions in Pixels**:
   We load the map as an image, where each pixel represents a small area of the warehouse. From the image data, we retrieve the map’s width and height in pixels (e.g., `image_width`, `image_height`).

2. **Calculate Scale Factors**:
   To determine how much each pixel represents in meters, we divide the warehouse's real-world dimensions by the map dimensions in pixels:
   - `scale_x = MAP_WIDTH / image_width`: This factor tells us how many meters each horizontal pixel represents.
   - `scale_y = MAP_HEIGHT / image_height`: This factor tells us how many meters each vertical pixel represents.

3. **Determine the Center of the Map**:
   The origin `(0, 0)` of the world coordinates is set at the center of the map. In pixel coordinates, this center is located at:
   - `center_x = image_width / 2`
   - `center_y = image_height / 2`

4. **Convert Pixel Coordinates to World Coordinates**:
   To convert from pixel coordinates `(y_pixel, x_pixel)` to world coordinates `(world_y, world_x)`, we calculate the offset of each pixel from the center and then multiply by the scale factors:
   - `world_y = (center_y - y_pixel) * scale_y`: Calculates the vertical world position. The `center_y - y_pixel` part gives the distance from the center, and multiplying by `scale_y` converts it to meters.
   - `world_x = (center_x - x_pixel) * scale_x`: Calculates the horizontal world position. The `center_x - x_pixel` part gives the horizontal distance from the center, and multiplying by `scale_x` converts it to meters.

### Interpreting the Conversion Results

Below are the results for the four corners of the map, with their pixel coordinates converted to world coordinates. These values help us understand the physical layout of the map in the warehouse:

| Pixel Coordinates | World Coordinates (Meters) | Explanation |
|-------------------|----------------------------|-------------|
| `(0, 0)`          | `(6.8, 10.31)`            | **Top-left corner** in pixels corresponds to the **top-right** in the world, as both `y` and `x` are positive. The `y` position is high (6.8 meters from the center), and `x` is 10.31 meters from the center, indicating the right side of the map. |
| `(278, 0)`        | `(-6.75, 10.31)`          | **Bottom-left corner** in pixels converts to **top-left** in the world. The negative `y` value (`-6.75`) shows it’s below the center vertically, while `x = 10.31` meters shows it’s still on the right side horizontally. |
| `(0, 414)`        | `(6.8, -10.26)`           | **Top-right corner** in pixels corresponds to the **bottom-right** in world coordinates. The positive `y` value (`6.8`) shows it’s above the center, and the negative `x` value (`-10.26`) indicates it’s on the left side of the map horizontally. |
| `(278, 414)`      | `(-6.75, -10.26)`         | **Bottom-right corner** in pixels converts to **bottom-left** in the world. Both `y` and `x` are negative, indicating it is below the center vertically and to the left horizontally. |

These world coordinates give us the precise real-world locations of the map's corners relative to the warehouse center, allowing for accurate positioning and navigation within the simulated environment.

With these results, we see that the map changes x and y:
![grafica](https://github.com/user-attachments/assets/aad85ac5-1e56-4572-9ffc-aee6607c895f)



### Development
The practice provides us with the coordinates of the shelves, which are as follows
Shelves coordinates = {
    1: (3.728, 0.579),
    2: (3.728, -1.242),
    3: (3.728, -3.039),
    4: (3.728, -4.827),
    5: (3.728, -6.781),
    6: (3.728, -8.665),
}
##### Expand the obstacles
As I explained before, the map is rotated, so the x-values are the y-values and vice versa.

To ensure safety, I had to create a black and white map, where the black pixels are obstacles. To make it difficult to crash, I have expanded the obstacles.

##### Calculate the path
Once we have the map ready and the coordinate conversion functions in place, it is necessary to calculate the path. The practice provides us with OMPL functions, of which the one that needs to be changed the most isStateValid.
In this function it is necessary to make that it can only pass through the white spaces where there is enough space for the robot to pass, but it must also take into account if the robot carries or not the shelf, because if it carries the shelf it increases its dimensions.

##### Navigate
Now that we have calculated the path, we have to navigate. There are several aspects to take into consideration:

-Before moving the robot, we first obtain its current position and orientation using `HAL.getPose3d()`. This function provides the robot's current coordinates (`x`, `y`) and its orientation (`yaw`).
```python
robot_pose = HAL.getPose3d()  # Get the current pose of the robot
current_x = robot_pose.x
current_y = robot_pose.y
current_yaw = robot_pose.yaw
```

-We compute the angle to the target using the atan2 function, which returns the angle between the robot's current position and the target. Then, we calculate the difference between this angle and the robot's current orientation (`current_yaw`).

-Next, we calculate the displacement (`delta_x`, `delta_y`) between the current position and the target position (`target_x`, `target_y`). The distance to the target is then calculated using the Euclidean distance formula.

-We compute the angle to the target using the atan2 function, which returns the angle between the robot's current position and the target. Then, we calculate the difference between this angle and the robot's current orientation (`current_yaw`).

-If the angle difference (`angle_diff`) is greater than a threshold (e.g., 0.1 radians), the robot will rotate towards the target. Otherwise, it will move forward. The robot adjusts its rotation speed (HAL.setW()) to align with the target angle.
```python
if abs(angle_diff) > 0.1:
    HAL.setV(0)  # Stop moving forward
    HAL.setW(angle_diff * 0.5)  # Rotate towards the target
else:
    HAL.setW(0)  # Stop rotating
    HAL.setV(min(max_speed, distance * acceleration))  # Move towards the target
```

-Finally, we check if the robot has reached the target by comparing the distance to the target (`distance`). If the distance is below a certain threshold (e.g., 0.1), the robot stops.

##### Return
Once we have navigated to the shelf, the isStateValid function needs to be adjusted, as the dimensions of the robot change and it cannot pass through the same places. To do this, remove the legs from the shelf, painting everything white. Subsequently the drone will recalculate the route. 

### Errors

-One of the errors I have had is that sometimes, when calculating the path, it would calculate an imprecise and very short one, so I have made it calculate 3 paths and do the one that is longer.


![Intentos](https://github.com/user-attachments/assets/8d60a86c-1c22-4427-8d3c-3aa506bb6541)


-Another mistake I had was that when I got to the furthest shelf, when I came back I calculated the route by bypassing the other shelves. For this, I made the shelf bigger so that I couldn't fit through the spaces under the shelves.
To improve this part, I made the robot and the shelf move forward having more or less the same orientation. For that I made that when the robot reached the position of the shelf, it would turn until its orientation was 0, and from then on I do what I explained before.

-Another bug I had in the navigation part was that it sometimes crashed, and I realised it was because it was following the path, but a bit off on the x-axis of the map, so to correct it I simply had to adjust it by hand.
 I also did other tests, like changing the widening factor of the obstacles and changing the dimensions of the robot and the shelf until it was safe to navigate without the risk of crashing.


#### Videos
[https://youtu.be/gsgs8BnNizI](https://youtu.be/hy7GcNt-Bvw)

In the video we can see how he makes 3 attempts to calculate the path and gets the best one. It then navigates to the coordinate of the implemented path.
The next step it does is to orientate itself correctly so that it doesn't have to turn around after having taken the shelf. It then picks it up and makes another 3 attempts to calculate the path. Once it does this, it removes the shelf from the map and navigates to the position to leave the shelf, orients itself and leaves the shelf.
