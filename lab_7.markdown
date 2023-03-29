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

I also looked at the graph to calculate the 90% rise time, which calculated by doing the equation: $$ 0.9t = (41009.5 - 39630.5) * 0.9 / 1000 $$ and got 1.2411 seconds. 

I then calculated the m term, which was $$(-d * t_{0.9})/(ln(0.1)) = 5.390 * 10^{-4}$$

With this, I finally had my A and B matrices. I set the C matrix to be $$\begin{bmatrix} -1 & 0\end{bmatrix}$$ because we are converting our measurement (which is a positive measurement) into a negative distance measurement, which directly correlates to our state. The C matrix essentially describes how our measurement correlates to our state. The 0 represents the fact that we can't measure the velocity of our robot. 

## Other Kalman filter parameters
The only other Kalman filter parameters that we need are the noise parameters. We have process noise and measurement noise.

$${\Sigma}_u =  \begin{bmatrix} ({\sigma}_1)^2 & 0 \\\ 0 & ({\sigma}_2)^2 \end{bmatrix} $$

$${\Sigma}_z = ({\sigma}_3)^2 $$

Our process noise describes the noise in our estimate of our state. As this can't really be measured due to a myriad of factors, I estimated both parameters to be 10. With the sensor noise, we can measure the noise. I took a bunch of still-sensor measurements, calculated the standard deviation, and found it to be ~ 30 mm. 

## Offboard Kalman Filter
After I got all the parameters for the Kalman Filter, I tested it on some data that I saved from running the PI controller. Note that I had to discretize the A and B matrices because both were time dependent. This was mainly just multiplying them by the time elapsed.

Code to do this:
<style type="text/css">
  .gist {width:1000px !important;}
  .gist-file
  .gist-data {max-height: 500px;max-width: 1000px;}
</style>
<script src="https://gist.github.com/mattieuzhai/770ddc85454973721771e0cb9bc5958b.js"></script>

Below is a prediction with the aforementioned sigmas:
[!sddfdf](/Lab7/KF1.png)

They might look different and the Kalman filter may look less accurate, but there is actually no telling which one is more accurate. We used to be measuring our distance from the wall with the ToF sensor and it is noisy and can be a little less accurate. Thus, it's possible that our Kalman filter is actually more accurate at predicting how far our robot is from the wall and we should trust that more than just our sensor readings. 

I also wanted to prove that I had everything working and it wasn't just me reading my sensor measurements (which would work in this case, as the sensor measurements also happen to measure our state). Thus, I adjusted my parameters so my measurement noise was 3000, which would make it so that our Kalman filter only trusted our model. 
[!lkj](/Lab7/KF2.png)

As we can see, the model is still okay, but not the greatest. This could be due to noise or just inconsistencies with the battery. 

Finally, I switched it so that we only based our prediction off our measurement (turned process noise high and measurement noise low). This would give us a perfect reading, as our sensors measure our state. However, this isn't really helpful to us because we want to use the Kalman filter in order to predict in-between these sensor readings. Thus, when running the Kalman filter on board, we would mostly be relying on the prediction steps and then updating that with our sensor values when we get one. 
