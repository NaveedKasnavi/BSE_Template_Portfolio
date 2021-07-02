# RC Obstacle Avoidance Car
My project is a RC car that doesn't hit obstacles. 

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Naveed K | Saratoga High School | Electrical Engineering | Incoming Sophomore

# Second Milestone
My second milestone was to get the obstacle avoidance working. I made a simple program that would move forward if there was no obstacle in front of it, and if there was, it would turn to the righ. The main issue I ran into was that it would not detect obstacles that didn't touch the ground but were just above the ultrasonic. Other than that, it worked flawlessly, even with thin obstacles. 

[![Milestone 2](https://res.cloudinary.com/marcomontalbano/image/upload/v1625244690/video_to_markdown/images/youtube--ZUHUf0RwTek-c05b58ac6eb4c4700831b2b3070cd403.jpg)](https://www.youtube.com/watch?v=ZUHUf0RwTek "Second Milestone"){:target="_blank" rel="noopener"}

# First Milestone


My first milestone was getting the ultrasonic sensor working and allowing it to detect any obstacle, even if it was thin. To do so, I mounted it on a servo with 3d printed parts that were secure enough so it wouldn't move even though it was secured with tape. I also assembled the chassis and got all the electronics wired up, including the motor driver and all 4 motors. I also made a test program where the robot would only move if there was no obstacle in front of it.

[![First Milestone](https://res.cloudinary.com/marcomontalbano/image/upload/v1624639229/video_to_markdown/images/youtube--BcDiVngkLWw-c05b58ac6eb4c4700831b2b3070cd403.jpg)](https://www.youtube.com/watch?v=BcDiVngkLWw "First Milestone"){:target="_blank" rel="noopener"}

# Obstacle Avoidance Test
<pre>
<font color="#5e6d03">#include</font> <font color="#434f54">&lt;</font><font color="#000000">ultrasonic</font><font color="#434f54">.</font><font color="#000000">h</font><font color="#434f54">&gt;</font>
<font color="#5e6d03">#include</font> <font color="#434f54">&lt;</font><font color="#000000">motorlib</font><font color="#434f54">.</font><font color="#000000">h</font><font color="#434f54">&gt;</font>
<font color="#5e6d03">#include</font> <font color="#434f54">&lt;</font><font color="#000000">ESP32Servo</font><font color="#434f54">.</font><font color="#000000">h</font><font color="#434f54">&gt;</font>

<font color="#00979c">int</font> <font color="#000000">SPEED</font> <font color="#434f54">=</font> <font color="#000000">140</font><font color="#000000">;</font>

<b><font color="#d35400">Motor</font></b> <font color="#000000">leftMotor</font> <font color="#434f54">=</font> <b><font color="#d35400">Motor</font></b><font color="#000000">(</font><font color="#000000">2</font><font color="#434f54">,</font> <font color="#000000">4</font><font color="#434f54">,</font> <font color="#000000">15</font><font color="#000000">)</font><font color="#000000">;</font>
<b><font color="#d35400">Motor</font></b> <font color="#000000">rightMotor</font> <font color="#434f54">=</font> <b><font color="#d35400">Motor</font></b><font color="#000000">(</font><font color="#000000">5</font><font color="#434f54">,</font> <font color="#000000">18</font><font color="#434f54">,</font> <font color="#000000">19</font><font color="#000000">)</font><font color="#000000">;</font>
<b><font color="#d35400">Ultrasonic</font></b> <font color="#000000">ut</font> <font color="#434f54">=</font> <b><font color="#d35400">Ultrasonic</font></b><font color="#000000">(</font><font color="#000000">21</font><font color="#434f54">,</font> <font color="#000000">11</font><font color="#000000">)</font><font color="#000000">;</font>
<b><font color="#d35400">Servo</font></b> <font color="#000000">angleServo</font><font color="#000000">;</font>

<font color="#00979c">int</font> <font color="#000000">minAngle</font> <font color="#434f54">=</font> <font color="#000000">10</font><font color="#000000">;</font>
<font color="#00979c">int</font> <font color="#000000">maxAngle</font> <font color="#434f54">=</font> <font color="#000000">75</font><font color="#000000">;</font>

<font color="#00979c">int</font> <font color="#000000">minDistance</font> <font color="#434f54">=</font> <font color="#000000">10</font><font color="#000000">;</font>
<font color="#00979c">bool</font> <font color="#000000">foundObstacle</font> <font color="#434f54">=</font> <font color="#00979c">false</font><font color="#000000">;</font>

<font color="#00979c">int</font> <font color="#000000">turnDuration</font> <font color="#434f54">=</font> <font color="#000000">200</font><font color="#000000">;</font>

<font color="#00979c">void</font> <font color="#5e6d03">setup</font><font color="#000000">(</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;<font color="#000000">leftMotor</font><font color="#434f54">.</font><font color="#5e6d03">setup</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">leftMotor</font><font color="#434f54">.</font><font color="#d35400">reverse</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">leftMotor</font><font color="#434f54">.</font><font color="#d35400">spin</font><font color="#000000">(</font><font color="#000000">0</font><font color="#000000">)</font><font color="#000000">;</font>

 &nbsp;<font color="#000000">rightMotor</font><font color="#434f54">.</font><font color="#5e6d03">setup</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">rightMotor</font><font color="#434f54">.</font><font color="#d35400">reverse</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">rightMotor</font><font color="#434f54">.</font><font color="#d35400">spin</font><font color="#000000">(</font><font color="#000000">0</font><font color="#000000">)</font><font color="#000000">;</font>

 &nbsp;<font color="#000000">angleServo</font><font color="#434f54">.</font><font color="#d35400">attach</font><font color="#000000">(</font><font color="#000000">23</font><font color="#000000">)</font><font color="#000000">;</font>

 &nbsp;<font color="#000000">ut</font><font color="#434f54">.</font><font color="#5e6d03">setup</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
<font color="#000000">}</font>

<font color="#00979c">void</font> <font color="#5e6d03">loop</font><font color="#000000">(</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;<font color="#000000">angleServo</font><font color="#434f54">.</font><font color="#d35400">write</font><font color="#000000">(</font><font color="#000000">minAngle</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">checkUltrasonic</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">angleServo</font><font color="#434f54">.</font><font color="#d35400">write</font><font color="#000000">(</font><font color="#000000">maxAngle</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">checkUltrasonic</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
<font color="#000000">}</font>

<font color="#00979c">void</font> <font color="#000000">checkUltrasonic</font><font color="#000000">(</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;<font color="#00979c">int</font> <font color="#000000">curTime</font> <font color="#434f54">=</font> <font color="#d35400">millis</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#00979c">bool</font> <font color="#000000">tempFoundObstacle</font> <font color="#434f54">=</font> <font color="#00979c">false</font><font color="#000000">;</font>
 &nbsp;<font color="#5e6d03">while</font> <font color="#000000">(</font><font color="#d35400">millis</font><font color="#000000">(</font><font color="#000000">)</font> <font color="#434f54">-</font> <font color="#000000">curTime</font> <font color="#434f54">&lt;</font> <font color="#000000">turnDuration</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">ut</font><font color="#434f54">.</font><font color="#d35400">update</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;<font color="#5e6d03">if</font> <font color="#000000">(</font><font color="#000000">ut</font><font color="#434f54">.</font><font color="#000000">distance</font> <font color="#434f54">&lt;</font> <font color="#000000">minDistance</font><font color="#000000">)</font> <font color="#000000">{</font>
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="#000000">tempFoundObstacle</font> <font color="#434f54">=</font> <font color="#00979c">true</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="#000000">foundObstacle</font> <font color="#434f54">=</font> <font color="#00979c">true</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">}</font>
 &nbsp;&nbsp;&nbsp;<font color="#000000">drive</font><font color="#000000">(</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;&nbsp;&nbsp;<font color="#d35400">delay</font><font color="#000000">(</font><font color="#000000">5</font><font color="#000000">)</font><font color="#000000">;</font>
 &nbsp;<font color="#000000">}</font>

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

</pre>

![Circuit](https://user-images.githubusercontent.com/86121861/123466858-b1d7e100-d5a4-11eb-8ac4-753f93b7f290.JPG)

