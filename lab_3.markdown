---
layout: default
title: Lab 3
permalink: /lab3/
---
## Lab 3
The purpose of this lab was to get the time-of-flight (ToF) sensors working and reading distances from them. We also used Bluetooth to send back data from the time-of-flight sensors. 

#### Communicating with the ToF sensors
Each ToF sensor communicates utilizing the I2C protocol, which means that each sensor should have a unique I2C address. Unfortunately for us, each sensor comes hard-coded with the same I2C address (0x52), that means that we cannot communicate with both sensors as they're given and have to make some modifications in order to get readable data. 

THe way I chose to approach it was to turn off one of the sensors using the XSHUT pin and then re-write the I2C address of the other sensor. This would allow us to easily communicate with and receive data from both sensors. While this method only requires one of the sensors to have a wire soldered to their XSHUT pin, I have both sensors' XSHUT pins soldered to a pin on the Artemis. The main disadvantage to this strategy is that each sensor cannot remember when you rewrite their I2C address so you have to rewrite it everytime you power the sensor on. 

#### Sensor Placement 
I chose to place two ToF sensors on my robot, with one facing forwards on the front of the robot and one facing left 90 degrees on the side of the robot. I chose these two positions because it makes calculating coordinates in relation to my robot's reference frame easier and it's easy to visualize what the robot should be seeing. However, this creates some pretty large blindspots, namely on the entire right-side of the robot, behind my robot, as well as a small area inbetween the two sensors. Therefore, my robot won't be able to detect obstacles on those sides of the robot and will just crash. For instance, my robot could drift left, exposing its right side and crash that way. It could also back up into obstacles. 

#### Wiring Diagram
![wd](/Lab3/wireDiagram.png)

#### Wiring Photos 
![wp](/Lab3/wirePhoto.png)

#### Scanning for I2C Address 
The Sparkfun people fortunately made many examples for the Artemis Nano, including one that utilizes the Arduino Wire.h library to communicate with I2C devices. We used this example (Example5_WireI2C) in order to scan the peripheral for I2C devices and return the I2C address of the device (in this case, the ToF sensor)
![scan](/Lab3/scanSS.png)
As we can see, the program scans an extra port (that doesn't seem to exist most likely just a Windows 10 thing) but it still can detect a device with an I2C address of 0x29. Now, this is the ToF sensor even though it doesn't match the hard-coded 0x52. I've written both I2C addresses below in binary:

0x52 = 0b1010010
0x29 = 0b101001

It 