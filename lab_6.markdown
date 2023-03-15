---
layout: default
title: Lab 6
permalink: /lab6/
usemathjax: true
---
## Lab 6
The objective of this lab was to get a PI/PID controller working on our robot as well as set up our Bluetooth system to be better suited to the labs. For this lab, I chose to do Task A, the distance-control task

### Prelab
For this lab, I changed my ble_arduino.ino file to set up flags that would start/stop the PID control from running. I also set it up so that it would send data asynchronously, since sending data as soon as I got it would slow down the system. I have a PID case for this lab, where depending on the value I send to the Artemis ("1" for turning it on, "0" for turning it off), I will either start running the PID control or stop running the PID control. I also used the old command SET_VEL that seems to be left over from previous years to send new Kp/Ki values to the robot so that I didn't have to reupload code every time I wanted to change them when testing the controller. In this command, I also reset all the PID control values so that I could start a new run. I decided that I was going to store data for every run of the PID to make things simpler, but would only send the data back through the GET_DATA command. Similar to how I structured the PID command, I set it so that to send back PID data, I would have to write a "1" to the Artemis. Thus, for future labs, I can use the same command and just send back different values as another flag from Python. I stored 500 values for every run of the loop. 

The code is super-long, so I will attach the relevant code.

Arduino code (mainly the case statements and the void loop() function):
<style type="text/css">
  .gist {width:1000px !important;}
  .gist-file
  .gist-data {max-height: 500px;max-width: 1000px;}
</style>
<script src="https://gist.github.com/mattieuzhai/a18b0d4e5db97f4858ffa63bd7c960b7.js"></script>

And the Python code (mainly notification handler):
