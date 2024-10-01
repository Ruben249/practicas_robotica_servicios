# Service_Robotics_Practices
## Localized Vacuum Cleaner

![Portada](https://github.com/user-attachments/assets/48d01426-4d08-47a6-8321-2a3bc9dcf54e)

### Index

-[Start](#start)


-[Development](#development)


-[Errors](#errors)

#### Start

To start with this practice, it is necessary to transform the coordinates of the world with those of the robot.
To do this, I have moved the robot in Gazebo to the top left corner to get the coordinates of the robot in the first white pixel, and I have done the same in the bottom right corner, to get the last white pixel.
To get the first white pixel coordinate, I use this function:
```python
def first_white(map):
    """ first_white is a function that iterates over the image to find the first white pixel (255, 255, 255). """
    for y in range(map.shape[0]):
        for x in range(map.shape[1]):
            # Chexk if the pixel is white
            if np.array_equal(map[y, x], [255, 255, 255]):
                return x, y
    return None
```
For the last white pixel, I use one similar function.
I did the same with some other points on the map in order to get a correlation. To do this calculation, I used the transformation matrix provided:
![image](https://github.com/user-attachments/assets/71623fa5-b5c0-4308-824c-194167d08119)


#### Development


Here is a video where we briefly see how the car behaves.

#### Errors
[FollowLine_Simple.webm](https://github.com/Ruben249/practicas_robotica_movil/assets/102288264/2c9a9c66-92af-472b-bf55-7fa6d0fcbe95)

