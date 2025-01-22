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

![Grid](https://github.com/user-attachments/assets/d0b851bb-dcbb-41b1-abcb-e5d2a829178d)

I have made this grid by dividing the map into 15X15 cells. The function to see it is the following:
```python
# Function to draw grid on the resized map
def draw_grid(image, cell_size):
    # Iterate over the image dimensions and draw lines for the grid
    for x in range(0, image.shape[1], cell_size):
        cv2.line(image, (x, 0), (x, image.shape[0]), (0, 255, 0), 1)  # Vertical green lines
    for y in range(0, image.shape[0], cell_size):
        cv2.line(image, (0, y), (image.shape[1], y), (0, 255, 0), 1)  # Horizontal green lines
```

Based on this algorithm, at each position we have 4 neighbours, north, east, south and west. We have to take one of those neighbours to move forward and prioritise them in that order. We have to go north, if not we have to go east, if not we have to go south and if not we have to go west. 


#### Errors

-When doing the transformation to find the correlation between the coordinates on the map and those of the robot, one problem I had was that when I moved the robot, it was represented inversely on the map, like a mirror image. I solved it by inverting the x-axis.


-Once I had the cells and the BGS algorithm done, I forgot to take into account that I only had to go through the cells that were on white pixels, so I calculated the path over the whole map, generating this image:

![image](https://github.com/user-attachments/assets/198e25a9-15cd-4473-b2f1-256a09be7cef)

To solve this, I created an array with only the navigable cells, so that the BGS algorithm only does it on it. To refine it, I made all cells that had a black pixel non-navigable. Here is the result:

![Path](https://github.com/user-attachments/assets/670c01f9-24ad-4aca-8680-0b07bcfd7148)

As you can see in the photo, when I calculate the path there are some places that are not painted. These are the points where I do the backtracking. I solved this problem by adding this point to the array, because before I didn't add it, and I have made the red squares smaller to see the cells well, and this is how it looks:

![GOOD-PATH](https://github.com/user-attachments/assets/f28340e2-a560-49e3-9ab7-82db8ce18a67)


-Another problem I had was that when calculating the path, it kept overwriting itself when doing the backtrackin, so in the end it detected that I had already finished navigating. By changing this, every time I had to do the backtracking to another point, it started to navigate. To solve these problems, I created a ‘security’ array with which I saved the whole path every time I did bactracking.

-The last problem I had was that when navigating, the robot would crash into obstacles. To solve this problem, I decided to create a function that checks the points that are close to an obstacle and makes them unnavigable. After this I had to adjust the erode kernel so that it doesn't convert so many cells directly into obstacles.

![Screenshot from 2025-01-22 16-28-30](https://github.com/user-attachments/assets/281f6176-24d7-41e0-9795-16fe141d2a3c)


#### Video 
Here is a video where we briefly see how the robot behaves.
[Video1]([https://github.com/Ruben249/practicas_robotica_movil/assets/102288264/2c9a9c66-92af-472b-bf55-7fa6d0fcbe95](https://youtu.be/PlmgJyPVkKU))

