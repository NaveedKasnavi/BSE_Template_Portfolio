# RC Obstacle Avoidance Car
My project is a RC car that doesn't hit obstacles. 

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Naveed K | Saratoga High School | Electrical Engineering | Incoming Sophomore

# Fourth Milestone
My fourth milestone was the modify the car chassis to have axle steering instead of tank steering and have the controller be able to drive the car around while not hitting obstacles. The regular arduino servo wasn't strong enough, so I got a 20kg servo for the steering. To turn the wheels, I 3d printed a ball joint, which would connect to the servo. I used a ball joint because the lever moves horizontally, so if i hadn't used a ball joint it would've bent and either would cause a lot of friction or would break. 

# Third Milestone
My third milestone was to make a custom controller to control the car. I used Kevin Miller's library to send data between two ESPs, but it didn't work at first because I forgot to get the mac address of each ESP. When I got it working, I plugged in two potentiometers, one for throttle, and one for turning, and sent data from the controller ESP to the car ESP. I also 3d printed a custom case for the controller to house the ESP and potentiometer. 

[![Milestone 3](https://res.cloudinary.com/marcomontalbano/image/upload/v1626926754/video_to_markdown/images/youtube---1FtQ3vBvkw-c05b58ac6eb4c4700831b2b3070cd403.jpg)](https://www.youtube.com/watch?v=-1FtQ3vBvkw "Milestone 3")

# Second Milestone
My second milestone was to get the obstacle avoidance working. I made a simple program that would move forward if there was no obstacle in front of it, and if there was, it would turn to the right. The main issue I ran into was that it would not detect obstacles that didn't touch the ground but were just above the ultrasonic. Other than that, it worked flawlessly, even with thin obstacles. I also switched from the Arduino to the ESP32, which has an integrated bluetooth module and created an app on a phone to control the car. 

[![Milestone 2](https://res.cloudinary.com/marcomontalbano/image/upload/v1625244690/video_to_markdown/images/youtube--ZUHUf0RwTek-c05b58ac6eb4c4700831b2b3070cd403.jpg)](https://www.youtube.com/watch?v=ZUHUf0RwTek "Second Milestone"){:target="_blank" rel="noopener"}

# First Milestone


My first milestone was getting the ultrasonic sensor working and allowing it to detect any obstacle, even if it was thin. To do so, I mounted it on a servo with 3d printed parts that were secure enough so it wouldn't move even though it was secured with tape. I also assembled the chassis and got all the electronics wired up, including the motor driver and all 4 motors. I also made a test program where the robot would only move if there was no obstacle in front of it.

[![First Milestone](https://res.cloudinary.com/marcomontalbano/image/upload/v1624639229/video_to_markdown/images/youtube--BcDiVngkLWw-c05b58ac6eb4c4700831b2b3070cd403.jpg)](https://www.youtube.com/watch?v=BcDiVngkLWw "First Milestone"){:target="_blank" rel="noopener"}

# Reflection

# Final Code
<pre>
<font color="#434f54">&#47;&#47;#include &lt;ultrasonic.h&gt;</font>
<font color="#5e6d03">#include</font> <font color="#434f54">&lt;</font><font color="#000000">motorlib</font><font color="#434f54">.</font><font color="#000000">h</font><font color="#434f54">&gt;</font>
<font color="#5e6d03">#include</font> <font color="#434f54">&lt;</font><font color="#000000">ESP32Servo</font><font color="#434f54">.</font><font color="#000000">h</font><font color="#434f54">&gt;</font>
<font color="#5e6d03">#include</font> <font color="#005c5f">&#34;BluetoothSerial.h&#34;</font> 

<font color="#00979c">int</font> <font color="#000000">SPEED</font> <font color="#434f54">=</font> <font color="#000000">140</font><font color="#000000">;</font>

<font color="#434f54">&#47;&#47; init Class:</font>
<b><font color="#d35400">BluetoothSerial</font></b> <font color="#000000">ESP_BT</font><font color="#000000">;</font> 

<b><font color="#d35400">Motor</font></b> <font color="#000000">leftMotor</font> <font color="#434f54">=</font> <b><font color="#d35400">Motor</font></b><font color="#000000">(</font><font color="#000000">33</font><font color="#434f54">,</font> <font color="#000000">32</font><font color="#434f54">,</font> <font color="#000000">25</font><font color="#000000">)</font><font color="#000000">;</font>
<b><font color="#d35400">Motor</font></b> <font color="#000000">rightMotor</font> <font color="#434f54">=</font> <b><font color="#d35400">Motor</font></b><font color="#000000">(</font><font color="#000000">5</font><font color="#434f54">,</font> <font color="#000000">18</font><font color="#434f54">,</font> <font color="#000000">19</font><font color="#000000">)</font><font color="#000000">;</font>
<font color="#434f54">&#47;&#47;Ultrasonic ut = Ultrasonic(26, 27);</font>
<b><font color="#d35400">Servo</font></b> <font color="#000000">angleServo</font><font color="#000000">;</font>

<font color="#00979c">int</font> <font color="#000000">minAngle</font> <font color="#434f54">=</font> <font color="#000000">10</font><font color="#000000">;</font>
<font color="#00979c">int</font> <font color="#000000">maxAngle</font> <font color="#434f54">=</font> <font color="#000000">75</font><font color="#000000">;</font>

<font color="#00979c">float</font> <font color="#000000">distance</font><font color="#000000">;</font>
<font color="#00979c">int</font> <font color="#000000">minDistance</font> <font color="#434f54">=</font> <font color="#000000">10</font><font color="#000000">;</font>
<font color="#00979c">bool</font> <font color="#000000">foundObstacle</font> <font color="#434f54">=</font> <font color="#00979c">false</font><font color="#000000">;</font>

<font color="#00979c">int</font> <font color="#000000">turnDuration</font> <font color="#434f54">=</font> <font color="#000000">200</font><font color="#000000">;</font>
<font color="#00979c">int</font> <font color="#000000">incoming</font><font color="#000000">;</font>
<font color="#00979c">float</font> <font color="#000000">times</font> <font color="#434f54">=</font> <font color="#000000">0</font><font color="#000000">;</font>

<font color="#00979c">int</font> <font color="#000000">trigPin</font> <font color="#434f54">=</font> <font color="#000000">26</font><font color="#000000">;</font>
<font color="#00979c">int</font> <font color="#000000">echoPin</font> <font color="#434f54">=</font> <font color="#000000">27</font><font color="#000000">;</font>

<font color="#00979c">void</font> <font color="#5e6d03">setup</font><font color="#000000">(</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;<b><font color="#d35400">Serial</font></b><font color="#434f54">.</font><font color="#d35400">begin</font><font color="#000000">(</font><font color="#000000">9600</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">ESP_BT</font><font color="#434f54">.</font><font color="#d35400">begin</font><font color="#000000">(</font><font color="#005c5f">&#34;ESP32_Control&#34;</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">leftMotor</font><font color="#434f54">.</font><font color="#5e6d03">setup</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">leftMotor</font><font color="#434f54">.</font><font color="#d35400">reverse</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">leftMotor</font><font color="#434f54">.</font><font color="#d35400">spin</font><font color="#000000">(</font><font color="#000000">0</font><font color="#000000">)</font><font color="#000000">;</font>

 &nbsp;<font color="#000000">rightMotor</font><font color="#434f54">.</font><font color="#5e6d03">setup</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
<font color="#434f54">&#47;&#47; &nbsp;rightMotor.reverse();</font>
 &nbsp;<font color="#000000">rightMotor</font><font color="#434f54">.</font><font color="#d35400">spin</font><font color="#000000">(</font><font color="#000000">0</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;

 &nbsp;<font color="#000000">angleServo</font><font color="#434f54">.</font><font color="#d35400">attach</font><font color="#000000">(</font><font color="#000000">23</font><font color="#000000">)</font><font color="#000000">;</font>

<font color="#434f54">&#47;&#47; &nbsp;ut.setup();</font>

 &nbsp;<font color="#000000">setupUltrasonic</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
<font color="#000000">}</font>

<font color="#00979c">void</font> <font color="#5e6d03">loop</font><font color="#000000">(</font><font color="#000000">)</font> <font color="#000000">{</font>
<font color="#434f54">&#47;&#47; &nbsp;Serial.println(&#34;WORKING&#34;);</font>
 &nbsp;<font color="#5e6d03">if</font> <font color="#000000">(</font><font color="#000000">ESP_BT</font><font color="#434f54">.</font><font color="#d35400">available</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">incoming</font> <font color="#434f54">=</font> <font color="#000000">ESP_BT</font><font color="#434f54">.</font><font color="#d35400">read</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">}</font>
<font color="#434f54">&#47;&#47; &nbsp;incoming = 49;</font>
 &nbsp;<font color="#5e6d03">if</font> <font color="#000000">(</font><font color="#000000">incoming</font> <font color="#434f54">==</font> <font color="#000000">49</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">angleServo</font><font color="#434f54">.</font><font color="#d35400">write</font><font color="#000000">(</font><font color="#000000">minAngle</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">checkUltrasonic</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">angleServo</font><font color="#434f54">.</font><font color="#d35400">write</font><font color="#000000">(</font><font color="#000000">maxAngle</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">checkUltrasonic</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">}</font> <font color="#5e6d03">else</font> <font color="#000000">{</font>
 &nbsp;&nbsp;&nbsp;<b><font color="#d35400">Serial</font></b><font color="#434f54">.</font><font color="#d35400">println</font><font color="#000000">(</font><font color="#005c5f">&#34;FALSE&#34;</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">leftMotor</font><font color="#434f54">.</font><font color="#d35400">spin</font><font color="#000000">(</font><font color="#000000">0</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">rightMotor</font><font color="#434f54">.</font><font color="#d35400">spin</font><font color="#000000">(</font><font color="#000000">0</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">}</font>
 &nbsp;
<font color="#000000">}</font>

<font color="#00979c">void</font> <font color="#000000">checkUltrasonic</font><font color="#000000">(</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;<font color="#00979c">unsigned</font> <font color="#00979c">long</font> <font color="#000000">curTime</font> <font color="#434f54">=</font> <font color="#d35400">millis</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#00979c">bool</font> <font color="#000000">tempFoundObstacle</font> <font color="#434f54">=</font> <font color="#00979c">false</font><font color="#000000">;</font>
 &nbsp;<font color="#5e6d03">while</font> <font color="#000000">(</font><font color="#d35400">millis</font><font color="#000000">(</font><font color="#000000">)</font> <font color="#434f54">-</font> <font color="#000000">curTime</font> <font color="#434f54">&lt;</font> <font color="#000000">turnDuration</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">updateUltrasonic</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
<font color="#434f54">&#47;&#47; &nbsp;&nbsp;&nbsp;ut.update();</font>
<font color="#434f54">&#47;&#47; &nbsp;&nbsp;&nbsp;Serial.println(ut.distance);</font>
 &nbsp;&nbsp;&nbsp;&nbsp;<b><font color="#d35400">Serial</font></b><font color="#434f54">.</font><font color="#d35400">println</font><font color="#000000">(</font><font color="#000000">distance</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;<font color="#5e6d03">if</font> <font color="#000000">(</font><font color="#000000">distance</font> <font color="#434f54">&lt;</font> <font color="#000000">minDistance</font> <font color="#434f54">&amp;&amp;</font> <font color="#000000">distance</font> <font color="#434f54">!=</font> <font color="#000000">0</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="#000000">tempFoundObstacle</font> <font color="#434f54">=</font> <font color="#00979c">true</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="#000000">foundObstacle</font> <font color="#434f54">=</font> <font color="#00979c">true</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">}</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">drive</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;<font color="#d35400">delay</font><font color="#000000">(</font><font color="#000000">5</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">}</font>
 &nbsp;<font color="#434f54">&#47;&#47;Serial.println(foundObstacle);</font>
 &nbsp;<font color="#000000">foundObstacle</font> <font color="#434f54">=</font> <font color="#000000">tempFoundObstacle</font><font color="#000000">;</font>
<font color="#000000">}</font>

<font color="#00979c">void</font> <font color="#000000">drive</font><font color="#000000">(</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;<font color="#5e6d03">if</font> <font color="#000000">(</font><font color="#434f54">!</font><font color="#000000">foundObstacle</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">leftMotor</font><font color="#434f54">.</font><font color="#d35400">spin</font><font color="#000000">(</font><font color="#000000">250</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">rightMotor</font><font color="#434f54">.</font><font color="#d35400">spin</font><font color="#000000">(</font><font color="#000000">250</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">}</font> <font color="#5e6d03">else</font> <font color="#000000">{</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">leftMotor</font><font color="#434f54">.</font><font color="#d35400">spin</font><font color="#000000">(</font><font color="#000000">200</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">rightMotor</font><font color="#434f54">.</font><font color="#d35400">spin</font><font color="#000000">(</font><font color="#434f54">-</font><font color="#000000">200</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">}</font>
<font color="#000000">}</font>

<font color="#00979c">void</font> <font color="#000000">setupUltrasonic</font><font color="#000000">(</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;<font color="#d35400">pinMode</font><font color="#000000">(</font><font color="#000000">trigPin</font><font color="#434f54">,</font> <font color="#00979c">OUTPUT</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#d35400">pinMode</font><font color="#000000">(</font><font color="#000000">echoPin</font><font color="#434f54">,</font> <font color="#00979c">INPUT</font><font color="#000000">)</font><font color="#000000">;</font>
<font color="#000000">}</font>

<font color="#00979c">void</font> <font color="#000000">updateUltrasonic</font><font color="#000000">(</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;<font color="#d35400">digitalWrite</font><font color="#000000">(</font><font color="#000000">trigPin</font><font color="#434f54">,</font> <font color="#00979c">HIGH</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#d35400">delayMicroseconds</font><font color="#000000">(</font><font color="#000000">10</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#d35400">digitalWrite</font><font color="#000000">(</font><font color="#000000">trigPin</font><font color="#434f54">,</font> <font color="#00979c">LOW</font><font color="#000000">)</font><font color="#000000">;</font>

 &nbsp;<font color="#000000">times</font> <font color="#434f54">=</font> <font color="#d35400">pulseIn</font><font color="#000000">(</font><font color="#000000">echoPin</font><font color="#434f54">,</font> <font color="#00979c">HIGH</font><font color="#000000">)</font><font color="#000000">;</font>

 &nbsp;<font color="#000000">distance</font> <font color="#434f54">=</font> <font color="#000000">(</font><font color="#000000">times</font> <font color="#434f54">*</font> <font color="#000000">0.034</font><font color="#000000">)</font> <font color="#434f54">&#47;</font> <font color="#000000">2</font><font color="#000000">;</font>
<font color="#000000">}</font>

</pre>

# Car CAD:

![FinalCar](https://user-images.githubusercontent.com/86121861/126816882-64669478-cf40-4221-8b0d-7a2c16caa94e.JPG)

# Controller CAD:

![FinalController](https://user-images.githubusercontent.com/86121861/126816888-9a50bae6-3377-4a0b-aeb5-4f9702541da8.JPG)

# Car Circuit

![FinalCarCircuit](https://user-images.githubusercontent.com/86121861/126817326-ca6445fa-2779-45ab-90c8-82d586135cbf.JPG)

# Controller Circuit:

![ControllerCircuit](https://user-images.githubusercontent.com/86121861/125980689-f67db829-a23b-4494-806a-ef16efa7f483.JPG)
