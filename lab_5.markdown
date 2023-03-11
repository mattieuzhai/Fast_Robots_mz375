---
layout: default
title: Lab 5
permalink: /lab5/
usemathjax: true
---

## Lab 5
The objective of this lab was to connect new motor drivers to our robot, assemble all of our sensors onto our robot, and demonstrate open-loop control of the robot from the Artemis. 

#### Diagram
Here is a diagram of how I wired up my motor drivers to the Artemis, batteries, and motors. I connected the 850 mAh battery to the motors instead of the given 650 mAh battery. I did this because the motors use a lot more power than the Artemis/sensors ever will, so they get the 650 mAh battery. Separating the power sources also allows for less noise, as the motor drivers generate a lot of noise. 

![diagram](/Lab5/motordriverdiag.png)


#### Testing the motor driver
After I soldered the wires to my motor driver and soldered the motor driver to my Artemis, I tested my motor driver output using a power supply and an oscillioscope. I forgot to take a photo of the setup (and now everything is connected to my robot), so here's a verbal description instead. I attached the power supply to the Vin and GND wires. I measured the output from the A/BOUT1 pins and the A/BOUT2 wire with the oscilloscope probes (I attached the probe to the A/BOUT1 wire and the ground clip to the A/BOUT2 wire)

For the power supply, I used an output of 3.7 V to match our battery. I also set the current output to 1.2 A to match the current output that was suggested on the datasheet. The code I used to test the output is below:

```C++
void setup(){
  pinMode(6, OUTPUT); //1
  pinMode(7, OUTPUT); //2
}
void loop(){
    analogWrite(6, 128);
    analogWrite(7,0);

    delay(5000);
    
    analogWrite(6, 0);
    analogWrite(7,128);
}
```

I used this in order to test the forward and backwards functionality of the motor drivers. I should get two square waves that are the inverse of each other. Here is a video showing that it works both ways:

<iframe width="800" height="420" src="https://youtube.com/embed/PWDE3oPmT50" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Note that the sawtooth wave is most likely due to some capacitance with the power supply or oscilloscope. When I tested it with a battery, I was able to see both square waves as expected. 

Next, I connected the motor driver to a motor to test it's spinning. I used the same code. Here is a video of the wheels spinning:

<iframe width="800" height="420" src="https://youtube.com/embed/hzpEp-xkcv4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Afterwards, I connected the motor driver to a battery and ran the same code again. Here is the output with both square waves:

<iframe width="800" height="420" src="https://youtube.com/embed/m0unCb0mC-k" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

When I ensured that one motor was working, I connected the other and ran through the same process (it looked the same as when I did the first motor driver) After I had everything soldered and in a closed circuit with the battery, I attached my Artemis and sensors to the robot. I placed a ToF sensor at the front and one to the left (follow the blue wires to find them). The IMU is near the front of the car as well (to be as far away from the motor drivers as possible) Here is a photo of the setup:

![photo](/Lab5/IMG-4291.jpg)

After this, I started trying to make my robot move. I first explored the lower PWM values (which set the duty cycle) which would start my car from rest. I found that each motor had a different minimum PWM value. One motor driver only needed a PWM value of 41, while the other one could have a PWM value of 30. The motor that controls my left wheels needs a stronger signal than the one to my right. Here is the test code:

```C++
void loop() {
  //Go forward

  analogWrite(6, 41);
  analogWrite(13,30);
  
  analogWrite(11,0);
  analogWrite(7, 0);

  delay(4000);
}
```
And here is the video of the robot chugging along:

<iframe width="800" height="420" src="https://youtube.com/embed/RCpaKFsu5hQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>    

It took a lot more power to start it turning from rest. I assume this is due to the robot not being a perfect differential drive robot and having 4 wheels instead of 2. It took a value of 150 for the left wheel and 130 for the right. Here is the code:

```C++
void loop() {
   //Go forward

  analogWrite(6, 150); 
  analogWrite(13, 0);

  analogWrite(11, 130);  
  analogWrite(7, 0);

  delay(4000);
}
```
And here is the video:
<iframe width="800" height="420" src="https://youtube.com/embed/a-58Z6i6Cww" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>   

After I did this, I tried to find the calibration factor to make my robot drive straight. I found that a good calibration factor was 0.57, where the "speed" for the right motor would be 0.57 times the speed of the left motor. I will probably change it to be 1.75 (1/0.57), where the speed for the left motor will be 1.75 times the speed for the right motor. That way, I can avoid any deadband issues. Here is the code:

```C++
void loop() {
  // put your main code here, to run repeatedly:
  speed = 80;

  //Go forward
  analogWrite(6, speed); 
  analogWrite(13, 0.57 * speed);

  analogWrite(11, 0);  
  analogWrite(7, 0);

  delay(2500);
}
```
And here is the video of the robot going straight(ish) for about 6 feet. It hits a bump in my floor at the end, which is why it does that weird