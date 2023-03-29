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

In my code, I kept the car still for 3 seconds, then drove it at a wall using a PWM value of 85 (which is around the ) until it reached a steady velocity, all while measuring the ToF values. Here is the data for that:

![df](/Lab7/distanceVsTime.png)

From this, I extracted the velocities using the differences between the distances divided by the time step. The graph looks a little funky because of the noise (a little different sensor measurements over a short time): 

![sdf](/Lab7/dirtyV.png)

Looking at the time when the robot starts to move, it looks like this:

![sdfsdf](/Lab7/cleanV.png)

As we can see, the steady state velocity evens out around 1000 mm/s. This means that is our steady-state velocity, and we can calculate our drag term to be u/velocity, or 1/1000 mm/s. This means that when we input a u (control) into our Kalman filter, we will need to divide it by 85 because that's the step response I used to calculate it. 

I also looked at the graph to calculate the 90% rise time, which calculated by doing the equation: $$ 0.9t = (41009.5 - 39630.5) * 0.9 / 1000 $$