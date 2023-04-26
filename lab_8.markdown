---
layout: default
title: Lab 8
permalink: /lab8/
usemathjax: true
---

# Lab 8
The purpose of this lab was to combine all of the cool things that we had done in previous labs and execute a stunt. I originally had decided that I was going to do the flip. However, after figuring out the code, I found that I couldn't really get my robot to do the stunt consistently and that bothered me so I decided to switch and do the drifting turn with orientation control. That meant I had to build a PID controller that controlled my orientation as well. 

## Building the orientation PID controller
I used a very similar controller to my distance one. However, I was now calculating yaw from the gyroscope and using that as my error and setpoint. I had to change how I thought about PID though. In order to make it be able to drive while turning, I would first have a base speed that my robot drives forward at. However, the way I would do it is have two functions that drive the robots right/left wheels separately. The control value I obtain from my PID value would be an offset factor that I add/subtract to the speed of the right and left wheels of my robot to make it turn and drive. Thus, if my error is 0, my robot would drive forward because the robot is supposed to be going forward, but if I change the setpoint to 180, it will turn and drift until it reaches a yaw of 180, because the right wheels speed will be greater than the left wheels speed. It also worked with a base speed of 0 to be able to do on-axis turns. 

## Stunts
The code I used for the stunt was pretty simple. I drove the robot at a base speed towards the wall while running the PID orientation to keep the yaw of the robot at 0 (so setpoint = 0). When my robot reached some distance from the wall, I would change the setpoint to 180 and make the robot turn around. Fortunately, this was more consistent than the flip and I was able to get consistent results, with the main issue being that I forgot to cap my PWM values for my motor speeds. The main debugging I had was choosing good Kp, Kd, and Ki values. I found that a Kp of 1.4, Ki of 0.3, and Kd of 0.5 worked when I had higher speeds. However, at lower base speeds, it wouldn't turn fast enough and so I had to have a higher Kp/Ki value. The videos are all shot with using a PWM value of 160 as the base speed for the motors. 

I used the linear extrapolator in order to predict the robots distance from the walls. From other people's labs, it appears that the linear extrapolator is better if your ToF sensors are able to give a reading decently quickly (mine seemed to give about every 100-200 ms), as the Kalman filter is more sensitive to battery levels since it has to predict the movement of the robot based of a PWM value and the same PWM value won't always cause the motors to drive at the same speed. I also developed my Kalman filter without tape on the wheels, so the dynamics and values I calculated wouldn't work. 

Here are the three stunt attempts I decided to upload. 
<iframe width="898" height="505" src="https://www.youtube.com/embed/EKoluVjh1UQ" title="Lab 8 Stunt" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<iframe width="898" height="505" src="https://www.youtube.com/embed/3jrfYmxNnCg" title="Lab 8 Stunt 2" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<iframe width="898" height="505" src="https://www.youtube.com/embed/KyTex5vwQtQ" title="Lab 8 Stunt 3" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

I also grabbed data from one of the turns. Here is a plot of just the PID control error over time:
![s](/Lab8/plot3.png)

And here is a plot of the distance as well as the PID error. 

![sd](/Lab8/plot1.png)

This shows that when I reached that threshold (which I had to manually tune because of the robot's momentum and the slipperiness of the taped wheels), it change the setpoint. I also stopped collecting distance values after I start executing the turn in order to speed up the loop and because I don't really need that data anymore, which is why it levels out as I no longer update the distance value. 

Here is a plot of my motor's PWM values. Note that I have a function to internally make sure that the PWM values never go above 255 when I input them, as that would just cause a lot of issues since it will spillover (so 257 would actually be 2). 

![sdf](/Lab8/plot2.png)

I also got some interesting bloopers. Occasionally, my robot's IMU would crap itself and stop working, leading to the robot doing donuts in the middle of a run. Here, you can see that I don't expect my robot to do that:

<iframe width="334" height="593" src="https://www.youtube.com/embed/BO-pYFzBHOk" title="Lab 8 Blooper" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Here is a blooper showing what tended to happen when my robot would randomly disconnect. This meant that it was no longer executing the control loop and because my robot has to constantly adjust itself to go straight and it suddenly wasn't, it just veered off course. 

<iframe width="898" height="505" src="https://www.youtube.com/embed/FoQjXXP8n_8" title="Lab 8 Blooper 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

And here is a video of a stunt that technically was not a success because my Kp value was too high, leading to overshoot, but the robot still did cool things and managed to correct even with some overshoot on the spin:

<iframe width="898" height="505" src="https://www.youtube.com/embed/3vSVNaLw-_A" title="Lab 8 Cool" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

