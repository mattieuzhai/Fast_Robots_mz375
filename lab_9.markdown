---
layout: default
title: Lab 9
permalink: /lab9/
usemathjax: true
---

# Lab 9
The purpose of this lab was to map out a map and learn how to convert points from a robot frame to the global frame. 

## Building an orientation PID controller
For this lab, I had to build a PID controller to either control orientation or angular speed. I decided to do orientation because I needed it for Lab 8 anyways and it would make taking points at certain angles easier. Because we're only doing small turns and don't really need to worry about speed, I essentially made it so that the robot would turn at a fixed speed (that I originally added to overcome deadband), stop at 15 degree increments, and then take a distance measurement. I did have to test my PID controller first. I tested it for a 180 degree turn and took some error values as well as the inputs to the motors. The Kp was 1.4, Ki was 0.1, and Kd was 1. Note that if there is a negative motor input, it means that the motor is turning backwards and I have a function to change the logic for this. 

<iframe width="800" height="420" src="https://youtube.com/embed/_2hJBe6mLdE?feature=share" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe> 

![s1](/Lab9/PID_plot1.png)
![s2](/Lab9/pwm_plot.png)

## Collecting data
I made the robot stop every 15 degrees for a full 360 degree turn so that 