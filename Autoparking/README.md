# Service_Robotics_Practices
## Autoparking

![Portada](https://github.com/user-attachments/assets/599d01b8-2952-401d-b8ba-48d58bdce6a7)


### Index

-[Start](#start)

-[Development](#development)

-[Errors](#errors)

-[Videos](#videos)

#### Start

To start with this practice, it is necessary to identify on which side the parking lot is located. To do this, you need to collect the data from the lasers and identify on which side the parked cars are parked.
Here is the API on how to use the laser:

```python
HAL.getFrontLaserData() # to obtain the front laser sensor data It is composed of 180 pairs of values: (0-180º distance in millimeters)
HAL.getRightLaserData() # to obtain the right laser sensor data It is composed of 180 pairs of values: (0-180º distance in millimeters)
HAL.getBackLaserData() # to obtain the back laser sensor data It is composed of 180 pairs of values: (0-180º distance in millimeters)
```

To tackle the exercise, I have created a state machine, each state being a phase of the car park.
```python
class States:
    CHECK_SIDE = 0
    CHECK_DISTANCE = 1
    APPROACH = 2
    ALIGN = 3
    FIND_SPOT = 4
    ADVANCE_TURN = 5
    REVERSE_TURN_1 = 6
    REVERSE_TURN_2 = 7
    FINAL_FORWARD = 8
    STOP = 9
```

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

