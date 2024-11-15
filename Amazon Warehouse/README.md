# Service_Robotics_Practices
## Amazon Warehouse 

![Portada](https://github.com/user-attachments/assets/1df065f8-d6d0-4e04-a78c-35a273cc4861)



### Index

-[Start](#start)

-[Development](#development)

-[Errors](#errors)

-[Videos](#videos)

#### Start


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



#### Development

Once we have identified the parking side, we have to centre the car on the road. To do this, we take the point furthest from the car and calculate the distance with:
```python
for i in range(180):
        dist = laser_data.values[i]
        angle = math.radians(i - 90)
        laser_polar.append((dist, angle))
        x = dist * math.cos(angle)
        y = dist * math.sin(angle)
        laser_xy.append((x, y))
```

##### Step1
By taking out the distance, we make the car turn and move forward slowly until the distance between the car and the previously chosen point is less than 2 metres.
![Paso1](https://github.com/user-attachments/assets/b779a6c8-d0f4-48bb-905d-99af1efcc6f6)

##### Step2
![Paso2](https://github.com/user-attachments/assets/8e1ac54f-239a-4557-a3c8-2073368fbe20)

##### Step3
When the distance is the desired one, we create a straight line between two points, those points will be the one after a measurement at infinity and the one before another measurement at infinity, we calculate the x of each one and we subtract it, that will be the deviation. We make the car move forward and turn in the other direction until this difference is close to 0.

![Paso3](https://github.com/user-attachments/assets/7182e81d-7a59-4ff1-9b32-148dfc24af9f)

Here is a picture of what the laser looks like with the cars in parallel and what it looks like when the car approaches them and has to straighten up:

Laser at step 1
![LStep1](https://github.com/user-attachments/assets/6d2e4f6c-feb3-48f6-8aeb-64abb3cf1d2f)

Laser at step 2
![LStep2](https://github.com/user-attachments/assets/1a956a5e-3706-4acc-b8ec-d06c7d732985)

Laser at step 3
![LStep3](https://github.com/user-attachments/assets/d6aceade-c152-4624-8d1d-319382244119)

We can see the above. We can see how at the beginning the car is further away from the parked cars, we can see how the inclination increases between the points where there are parked cars because the car is getting closer, and then we can see how the car corrects the direction and gets parallel and closer to the other cars. 

##### Step4
The next step is to find an available parking space. To do this, I followed the typical manoeuvres performed in normal driving. First I go over the gap, then I back up turning to the right, when I've got the back of the car in, we turn left, and finally we go straight ahead and turn to the centre. To do this, I have been checking how far the car and the yaw have moved forward.



#### Errors

-One error I encountered was that the vehicle failed to line up correctly with the line of cars. The incorrect orientation affected the accuracy in detecting gaps and moving in a straight line.
To solve this I implemented an alignment logic that adjusted the rotation of the vehicle until the difference in the x-coordinates of the detected points was close to zero, meaning that the vehicle was parallel to the cars.

-Another problem I had was finding the right value for the constants, because if I changed them slightly, the car would crash into parked cars or turn too much and drift away from the cars. 

-A recurring error I had was that when turning, the car stopped moving when it was close to another car, but it was not crashing, so I had to restart the computer and change the constants a little bit.

-Another problem that took up a lot of my time was figuring out the angles and speeds needed to perform the parking manoeuvre correctly.
#### Videos
In this first video we see how the car behaves when we have all the cars, just as the world is without touching anything:

[https://youtu.be/BdDdAW7zDIQ](https://youtu.be/1HeZRPQvYYM)

In the following video, we remove the cars in front of our car from the world, i.e. we remove the cars in front of us:

[https://youtu.be/tA4a2U7SKSI](https://youtu.be/_r4x2ddXYe4)

In this last video we make a bigger space to be able to park without a car behind. To do this, when looking for a parking space, we remove the car behind:

https://youtu.be/07b0AqKpK5s

