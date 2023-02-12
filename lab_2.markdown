---
layout: default
title: Lab 2
permalink: /lab2/
---
# Lab 2
The main objective of this lab was to get us familiar with the Bluetooth commands and make sure it worked with our Artemis boards

### Code Setup
First, we needed to install Python (I already had Python and pip3 installed)

Next, we created and activated a virtual environment by running the following commands in a terminal (Command Prompt for me, a Windows user):
```
python3 -m pip install --user virtualenv
python3 -m venv FastRobots_ble
.\FastRobots_ble\Scripts\activate
```

After this, we installed the necessary packages with the following command (though I later installed matplotlib as well using pip3)
```
 pip install numpy pyyaml colorama nest_asyncio bleak jupyterlab
 pip3 install matplotlib
```
In order to run the Python side of things, we used Jupyter which we could activate within our virtual environment using the command
```
jupyter lab
```
I then copied the given codebase into the project directory, which contained the base .ino files needed for the Artemis board. The codebase appears to be made up of a Python and Arduino half. The Python half allows us to communicate with the board through Bluetooth and send commands to the Artemis board, while the Arduino half is where we code in the actual commands that we want executed. 

Arduino comes with a library that supports Bluetooth Low Energy (BLE) which is used with Bluetooth 4.0 (and better). We installed this in the Arduino IDE and burned the given ble_arduino.ino sketch into the Artemis board. After uploading this, the Artemis board printed its MAC address to the Serial monitor:
![MAC](/Lab2/mac_address.png)

I also had to update the MAC address in the connection.yaml file, as well as create a new UUID for our board so that we wouldn't accidentally connect to any other boards. In order to generate these new UUIDs, we ran the following code, which was given to us in the demo.ipynb file:
```
from uuid import uuid4
uuid4()
```
I updated the ble_arduino.ino and connections.yaml files with this new uuid. 


### Demo
We were given a demo.ipynb file to run to ensure that everything worked. 
<style type="text/css">
  .gist {width:500px !important;}
  .gist-file
  .gist-data {max-height: 500px;max-width: 500px;}
</style>
<script src="https://gist.github.com/mattieuzhai/34eb5ffc9ddc850f809850cdd12c09dd.js"></script>

If everything worked, we should have been able to run every cell. Here are the outputs from the cells when I ran through the file: 
![c](/Lab2/connect_ss.png)
![d](/Lab2/demo_ss1.png)
![e](/Lab2/ping_ss.png)
And the Arduino Serial monitor when I ran PING and SEND_TWO_INTS:
![pong](/Lab2/pong_sm.png)
Interestingly, running the ble.disconnect() wouldn't always work cleanly and often made my notebook run that cell forever, so I simply pushed the reset button on the Artemsis to disconnect it for a lot of the tasks.
After proving everything worked, I started on the tasks. 
### Task 1
For this task, we simply had to make an ECHO command where the robot would return the string that we wrote to the Artemis board.  (modified a little bit). The processing of the string was made much easier by the EnhancedString class given to us. The Arduino code is below:
```C
char char_arr[MAX_MSG_SIZE];

// Extract the next value from the command string as a character array
 success = robot_cmd.get_next_value(char_arr);
if (!success)
    return;

/* /*
* Your code goes here.
*/
//Making an enhanced string
tx_estring_value.clear();
tx_estring_value.append("Robot says: ");
tx_estring_value.append(char_arr);
tx_estring_value.append("!");

//This is sending the string back to my computer
tx_characteristic_string.writeValue(tx_estring_value.c_str()); 
```
And the Python code + output:
![t1](/Lab2/task1.png)

### Task 2
Our next task was to make a command that returned the time. For this task, we also had to edit the cmd_types.py file and the enum CommandTypes function in ble_arduino.ino in order to add a new command. This allows us to assign an integer value to each command String, which is necessary for the switch function as the cases of the switch function cannot be a string. The millis() function in Arduino was also needed. The Arduino code: 
```C
//Building the string I want
tx_estring_value.clear();
tx_estring_value.append("T: ");
time_d = millis();
tx_estring_value.append(time_d);

//This is sending the string back to my computer
tx_characteristic_string.writeValue(tx_estring_value.c_str());
```
And the Python code + output:
![t2](/Lab2/task2.png)

### Task 3
The next task was setting up a notification handler to process any incoming data. Essentially, a notification handler executes a function anytime data with a specific UUID is received from the BLE device. Thus, I set up a function that would extract the time from the string and converted it back to an integer. One thing to note is that if you ran start_notify twice without running stop_notify in-between, then it would execute the function twice. The Python code + output (note that I've been running this for a very long time):
![t3](/Lab2/task3.png)

### Task 4 
The next task was making a function that would take the temperature at 1 second intervals over 5 seconds and then returning a time-stamped string of temperatures. I accomplished this using the built-in getTempDegC() function, which converted the temperature to Celsius for me. I used "|" as a delimiter for future processing.
Arduino code:
```C++
//Building the string I want
tx_estring_value.clear();
for(int i = 0; i < 5; i++){
    tx_estring_value.append("T:");
    tx_estring_value.append((float)(millis()/1000));
    tx_estring_value.append("|C:");
    tx_estring_value.append(getTempDegC());
    if(i != 4){
        tx_estring_value.append("|");
    }
    delay(1000);
}
tx_characteristic_string.writeValue(tx_estring_value.c_str());
```
And the Python code + output:
![t4](/Lab2/task4.png)

### Task 5

