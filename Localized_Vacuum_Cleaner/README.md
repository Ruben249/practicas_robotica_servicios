# Service_Robotics_Practices
## Localized Vacuum Cleaner

![Portada](https://github.com/user-attachments/assets/48d01426-4d08-47a6-8321-2a3bc9dcf54e)

### Index

-[Start](#start)


-[Development](#development)


-[Errors](#errors)

-[Video](#video)

#### Start

To start with this practice, it is necessary to transform the coordinates of the world with those of the robot.
To do this, I have moved the robot in Gazebo to the top left corner to get the coordinates of the robot in the first white pixel, and I have done the same in the bottom right corner, to get the last white pixel.
To get the first white pixel coordinate, I use this function:
```python
def first_white(map):
    """ first_white is a function that iterates over the image to find the first white pixel (255, 255, 255). """
    for y in range(map.shape[0]):
        for x in range(map.shape[1]):
            # Check if the pixel is white
            if np.array_equal(map[y, x], [255, 255, 255]):
                return x, y
    return None
```
For the last white pixel, I use one similar function.
I did the same with some other points on the map in order to get a correlation. To do this calculation, I used the transformation matrix provided:
![image](https://github.com/user-attachments/assets/71623fa5-b5c0-4308-824c-194167d08119)


#### Development

In order to debug, I have created the following function that paints a red rectangle where it calculates that the robot is in the map
![representacion](https://github.com/user-attachments/assets/1e9a31ee-1967-4f17-8917-7043a1d3a387)

```python
# Function to draw the robot's position on the image
def draw_robot():
    # Get the 2D position of the robot
    img_x, img_y = convert_position()

    # Ensure the coordinates are within the bounds of the image
    if 0 <= img_x < resized_map.shape[1] and 0 <= img_y < resized_map.shape[0]:
        # Draw a square
        cv2.rectangle(resized_map, (img_x, img_y), (img_x + 10, img_y + 10), 128, -1)
```

Once we got from the robot coordinates to the map coordinates, we had to divide the map into cells and calculate the best route according to the BSA algorithm(Backtracking Spiral Algorithm).
Based on this algorithm, at each position we have 4 neighbours, north, east, south and west. We have to take one of those neighbours to move forward and prioritise them in that order. We have to go north, if not we have to go east, if not we have to go south and if not we have to go west. 


#### Errors

-When doing the transformation to find the correlation between the coordinates on the map and those of the robot, one problem I had was that when I moved the robot, it was represented inversely on the map, like a mirror image. I solved it by inverting the x-axis.

#### Video 
Here is a video where we briefly see how the car behaves.
[FollowLine_Simple.webm](https://github.com/Ruben249/practicas_robotica_movil/assets/102288264/2c9a9c66-92af-472b-bf55-7fa6d0fcbe95)

