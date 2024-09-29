# Service_Robotics_Practices
## Localized Vacuum Cleaner

![Portada](https://github.com/user-attachments/assets/48d01426-4d08-47a6-8321-2a3bc9dcf54e)

### Index

-[Start](#start)


-[Development](#development)

#### Start

To start with this practice, it is necessary to transform the coordinates of the world with those of the robot.
To do this, I moved the robot in Gazebo to the first white pixel on the map, which is the top left corner, and wrote down the coordinates it returned. I also moved the robot to the last white pixel on the map, the bottom right corner, and wrote down the coordinates. I did the same with some other points on the map in order to get a correlation. To do this calculation, I used the transformation matrix provided:
![image](https://github.com/user-attachments/assets/71623fa5-b5c0-4308-824c-194167d08119)


#### Development

To solve the exercise it is necessary to implement at least one PID, so I have searched for information on how it is implemented exactly, and I have reached the following conclusion:

prev_speed_error represents the speed error at the previous time instant.
The cumulative sum of errors (integral, I) is used in the controller to correct accumulated errors over time.
Speed error is the difference between the desired speed and the current speed of the system.
The derivative part is used to predict how the error will change in the future and therefore helps to stabilize the system and reduce overshoot.
i = accum_speed_error2: The accumulated sum of errors is stored in the variable i. The interal part (I) of the controller uses this value to correct errors accumulated over time.

I frequently encountered the problem that as soon as I started the simulation the car began to oscillate, until I realized that it was a small error in the camera as soon as it started, so I decided to set a time.sleep of 1 sec so that I had time to capture the image

```python
# Function for speed PID
def speed_PID(speed_error):
    prev_speed_error2 = prev_speed_error
    accum_speed_error2 = accum_speed_error
    p = speed_error
    d = speed_error - prev_speed_error2
    i = accum_speed_error2
    return kp * p + ki * i + kd * d

# With this time.sleep we make the car take a second to start and thus avoid errors with the camera
time.sleep(1)

while True:
    # Get the image from the camera
    image = HAL.getImage()
```

To solve this exercise it is necessary to use the PID controllers explained in class. I have decided to use one for speed and another to be able to obtain how much it should rotate.

![ControlSystems](https://github.com/Ruben249/practicas_robotica_movil/assets/102288264/8ad3a57e-eafe-42f5-a874-caeef1a48ee8)
![TypesofControlSystems](https://github.com/Ruben249/practicas_robotica_movil/assets/102288264/2df9e2e4-403e-4c13-be16-a66fde0dbfd5)

Another recurring problem was that I couldn't find the kp, ki, kd that would allow me to find the right and necessary speed for the curves, so I decided to put some standard ones and modify the speeds later.

```python
# Calculate the error for linear and angular control
        speed_error = x_c - weight / 2

        # To maintain turning based on linear error
        err_angular = speed_error  

        # Detect if it's a curve or a straight line
        if abs(speed_error) > 1:
            # PID control for curves
            speed_control = speed_PID(0)  # We put 0 to keep constant linear speed in curves
            angular_control = angular_PID(err_angular)
        else:
            # PID control for straight lines
            speed_control = speed_PID(speed_error)
            angular_control = angular_PID(err_angular)
```

Here is a video where we briefly see how the car behaves.

[FollowLine_Simple.webm](https://github.com/Ruben249/practicas_robotica_movil/assets/102288264/2c9a9c66-92af-472b-bf55-7fa6d0fcbe95)

