---
layout: default
title: Lab 7
permalink: /lab7/
usemathjax: true
---
# Lab 7 
The purpose of this lab was to figure out a way to estimate the location of the robot in between ToF sensor readings. This is because our ToF sensor readings are too slow and our system won't be able to drive as fast. The best way to do this is with a Kalman filter, which we calculated and tested. However, I ended up implementing a linear extrapolator on the robot because it is simpler and more versatile with different battery levels. 

## Estimating drag and momentum
In order to do the Kalman filter, we need to estimate the dynamics of our system/state. Our state is defined as the distance from the wall and the velocity of our robot. Thus, we can estimate both using drag and momentum equations. Our state is thus defined as:

$$\begin{bmatrix} \dot{x} \\\ \ddot{x} \end{bmatrix} = \begin{bmatrix} 0  & 1 \\\ 0 & -d/m \end{bmatrix} * \begin{bmatrix} x \\\ \dot{x} \end{bmatrix} + \begin{bmatrix} 0 \\\ 1/m \end{bmatrix} * u $$

In order to find the constants d and m, we needed to apply a step response to our robot and measure both the steady-state velocity and the 90% rise time (which is 0.9 times the time it took to reach that steady-state velocity)

In my code, I kept the car still for 3 seconds, then drove it at a wall until it reached a steady velocity, all while measuring the ToF values. 
