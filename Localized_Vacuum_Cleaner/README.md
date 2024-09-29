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


Here is a video where we briefly see how the car behaves.

[FollowLine_Simple.webm](https://github.com/Ruben249/practicas_robotica_movil/assets/102288264/2c9a9c66-92af-472b-bf55-7fa6d0fcbe95)

