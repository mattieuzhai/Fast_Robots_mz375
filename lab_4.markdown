---
layout: default
title: Lab 4
permalink: /lab4/
---
## Lab 4
The purpose of this lab was to get the IMU working, set up our batteries so we can power our Artemis without having it connected to our laptop, and also start playing with our robot car and make sure that we can get some sensor readings while our robot is driving around. 

### Setting Up the IMU 
In order to be able to use the IMU (SparkFun SEN-15335), we had to install the relevant Arduino library, SparkFun 9 DOF IMU Breakout - ICM20948. To use the example code, I connected the IMU to the QWIIC port on the actual Artemis board so that there wouldn't be any other I2C devices on the peripheral. 

Photo of my setup originally:
![1](/Lab4/IMU.jpg)

After I connected it, I ran the Arduino example given to us with the Arduino library to make sure that the IMU was working (Example1_Basics.ino)

Serial Output:
![2](/Lab4/IMU_example.png)

In the data that the example prints, it seems that a lot of the values are scaled down by the example code. Other than that, the values were printed as predicted. The axes are defined on the IMU breakout board so it was easy to alter the values I wanted. The only thing that was of note is that the IMU could pick up on the Earth's graviational acceleration in the z-axis. 

Video of SerialPlot plotting accelerometer data:
<iframe width="560" height="315" src="https://youtube.com/embed/kauaCYTzCTM?feature=share" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

In the example, an AD0_val is defined. This value determines the least significant bit (LSB) of the IMU's I2C address. This means that we can use two IMUs in parallel if we have the logic value of that pin different on both IMUs. By default, the value is 0 when the ADR jumper is closed, which it should be by default. However, this doesn't really matter for us because we're only using one IMU. 


### Accelerometer Data for Pitch and Roll
In order to use the accelerometer data for pitch and roll, we used the equations that were given to us: 

$ \theta = tan^(-1)(a_x/a_z) = atan2(a_x/a_z) $