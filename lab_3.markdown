---
layout: default
title: Lab 3
permalink: /lab3/
---
## Lab 3
The purpose of this lab was to get the time-of-flight (ToF) sensors working and reading distances from them. We also used Bluetooth to send back data from the time-of-flight sensors. 

#### Communicating with the ToF sensors
Each ToF sensor communicates utilizing the I2C protocol, which means that each sensor should have a unique I2C address. Unfortunately for us, each sensor comes hard-coded with the same I2C address (0x52), that means that we cannot communicate with both sensors as they're given. 

The two ways to get around this are 1) power off one sensor and change the I2C address of the other and 2) power one off and read the other, alternating so that you can read each sensor one at a time.

The way I chose to approach it was to turn off one of the sensors using the XSHUT pin and then re-write the I2C address of the other sensor. While this method only requires one of the sensors to have a wire soldered to their XSHUT pin, I have both sensors' XSHUT pins soldered to a pin on the Artemis (mainly for troubleshooting). The main advantage to this strategy is that it is simpler to do, as you don't have to code in additional logic to turn off one sensor at a time. You are also able to read data quicker, as each sensor has a boot-up sequence when you power it on. The main disadvantage to this strategy is that each sensor cannot remember when you rewrite their I2C address so you have to rewrite it everytime you power the sensor on. Another disadvantage is that you would always have both sensors on to avoid this issue, thus drawing more power. 

The other method's advantage is that you could potentially save power by only having the necessary sensors on when you need them. The disadvantage is that it's more logically challenging to code and wouldn't be able to collect data as quick as you have to wait for each sensor to power on, which could take time. 

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

It appears that the program bitshifts the I2C address it reads to the right one in order to remove the R/W bit of the I2C address so it displays 0x52 >> 1, which is 0x29. 

#### Testing 1 ToF Sensor
The ToF sensor has 3 distance modes: Short, Medium and Long. The advantages of using a shorter distance mode is that it is more accurate regardless of the lighting (since it limits the range to around 1.2 m), while the big disadvantage obviously being that it cannot see as far. For my robot, I decided that I would use the Long mode because I wasn't sure what environments I would eventually be testing my robot in and might have to detect longer distances, especially if mapping eventually. However, if I'm just calibrating some other part of the robot and need the ToF data, then Short mode could be okay too.  

Sparkfun provides a library for the VL53L1X distance sensor. In order to test the functionality of a single sensor, I used the provided example (Example1_ReadDistance), with the sensor set to long mode. I tested the range by walking around with the distance sensor and found that the range was ~4 m, which is honestly pretty long. In order to test accuracy, reliability, and ranging time, I took 64 measurements from 10 cm to 30 cm in 5 cm increments and then calculated the mean and standard deviation in order to compare to the true distance. I also measured the ranging time using the micros() function in Arduino. 

Here is a plot showing that below (the error bars are there, just very teeny which is a good thing I suppose):
[g1](/Lab3/graph1.png)

And the Python scripting code:
```python
td = [100, 150, 200, 250, 300]
md = [100, 149, 199, 246, 307]
std = [1.25, 1.19, 1.34, 1.42, 1.55]
x = [1, 2, 3, 4, 5]
plt.plot(x,td,'r-o', label = 'True distance')
plt.errorbar(x, md, yerr = std, capsize = 5, label = 'Measured distance')
plt.legend()
plt.ylabel('Distance (mm)')
plt.xlabel('Measurement')
plt.title('Measured vs True Distances (with error bars)')
```
Ranging Time Table:
        | Ranging Time |
--------| -------------|
With everything | 2.369 ms |
W/o stopRanging() | 1.71 ms|
Just reading | 1.056 ms| 


And the Arduino code I used to measure it (the loop after all initialization steps):
```C++
void loop(void) {
  int times_sum = 0; int distance_sum = 0; 
  int distances[64];

  for(int i = 0; i < 64; i++){
    distanceSensor1.setDistanceModeLong();
    distanceSensor1.startRanging();  //Write configuration bytes to initiate measurement
    while (!distanceSensor1.checkForDataReady()) {
      delay(1);
    }
    long t1 = micros();
    int distance = distanceSensor1.getDistance();  //Get the result of the measurement from the sensor
    //distanceSensor1.clearInterrupt();
    //distanceSensor1.stopRanging();

    long t2 = micros();
    distances[i] = distance;
    Serial.print("Distance(mm): ");
    Serial.print(distance);
    Serial.printf("   Time for ranging: %d", (t2-t1));
    times_sum = times_sum + (t2 - t1);
    distance_sum = distance_sum + distance;

    Serial.println();
  }
  long time_avg = times_sum/64;
  int distance_avg = distance_sum >> 6;
  int std = 0;
  for(int i = 0; i < 64; i++){
    int diff = pow(distances[i] - distance_avg, 2);
    std = std + diff;
    Serial.printf("Difference: %d \n", diff);
  }
  

  Serial.printf("Average ranging time: %d\n", time_avg);
  Serial.printf("Average distance: %d\n", distance_avg);
  Serial.printf("StDev value: %d\n", std);
  delay(5000);
}
```
Note that I had to calculate the standard deviation off board, since I couldn't figure out sqrt() in Arduino, so the value I read isn't actually the true standard deviation, just part of the calculation 

#### Using 2 ToF Sensors
I used the rewrite I2C address method in order to read both sensors at the same time. Note that I had to connect to each sensor individually when using .begin() to initalize each sensor. 

A photo of my setup:
![s](/Lab3/twoToF.png)

Arduino code:
```C++
#include <Wire.h>
#include "SparkFun_VL53L1X.h"  //Click here to get the library: http://librarymanager/All#SparkFun_VL53L1X
#include <math.h>

#define XSHUT_PIN 8
#define XSHUT_PIN1 4
SFEVL53L1X distanceSensor1;
SFEVL53L1X distanceSensor2;


void setup(void) {
  Wire.begin();

  Serial.begin(115200);
  pinMode(XSHUT_PIN, OUTPUT);
  pinMode(XSHUT_PIN1, OUTPUT);

  digitalWrite(XSHUT_PIN, LOW);
  digitalWrite(XSHUT_PIN1, HIGH);

  if (distanceSensor1.begin() != 0)  //Begin returns 0 on a good init
  {
    Serial.println("Sensor1 failed to begin. Please check wiring. Freezing...");
    while (1)
      ;
  }
  distanceSensor1.setI2CAddress(0x25);
  digitalWrite(XSHUT_PIN, HIGH);

  if (distanceSensor2.begin() != 0)  //Begin returns 0 on a good init
  {
    Serial.println("Sensor2 failed to begin. Please check wiring. Freezing...");
    while (1)
      ;
  }
  
  Serial.println("Sensors online!");
  Serial.println(distanceSensor1.getI2CAddress());
  Serial.println(distanceSensor2.getI2CAddress());

  distanceSensor1.setDistanceModeLong();
  distanceSensor2.setDistanceModeLong();
}

void loop(void) {
  distanceSensor1.startRanging();  //Write configuration bytes to initiate measurement
  distanceSensor2.startRanging();
  
  while (!distanceSensor1.checkForDataReady() || !distanceSensor2.checkForDataReady()) {
    delay(1);
  }
 long t1 = micros();
 int distance1 = distanceSensor1.getDistance();  //Get the result of the measurement from the sensor
 //distanceSensor1.clearInterrupt();
 //distanceSensor1.stopRanging();
  int distance2 = distanceSensor2.getDistance();
  //distanceSensor2.clearInterrupt();
  //distanceSensor2.stopRanging();

  long t2 = micros();
 
  Serial.printf("Distance 1 (mm): %d, Distance 2(mm): %d", distance1, distance2);
  Serial.printf("   Time for ranging: %d", (t2-t1));
  
  Serial.println();
}
```

Serial output of my readings:
![xD](/Lab3/xD.jpg)

#### ToF Limiting Factor
In order to find out what the limiting factor was in 