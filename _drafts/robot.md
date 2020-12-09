---
layout: single
title:  "Cybot project"
date: 2020-12-09 09:00:00 +0100
categories: projects robots robotics arduino
excerpt: "For many year I try to hook up a robot toy with Arduino. Now I've created a nice software architecture to do it." 
---

# TLDR;
I added an Arduino powered microcontroller to an old robot toy. I created a nice reusable _Sense-Think-Act_ architecture in the process. You can find the code [here](https://github.com/ychevalier/SonarCybot/).

![Cybot](/assets/images/open_cybot.jpg)

# The Story
Ok, so this has been by far my longest project. I started building this robot in 2002, when I was 12. It's not like I knew electronics, coding or anything back then, I just happened to stumble upon a monthly publication, the ones where you get parts of a product and that are really cheap for the first few issues and then get quite expensive and last a really long time before you get the actual, complete, advertised product (?). 
It happened just like that, except in the process I got to build my first robot, from scratch, and see that it wasn't that much, and I also got to learn about existing robots, technology and people actually making different kind of robots. For me it was the kickstarter of wanting to build moving things! And then... Nothing happened...
The truth is, when you're a kid, it is quite difficult to get into electronics. It is complicated, you need components, which costs money (that you don't have), even if you have the money there was no place to buy them from in France, pre-internet. And even if you had them, well... You needed to design circuits and solder stuff which was just not something I could even grasp at the time. So I turned to software, which is the cheapest way of building big stuff! I actually spoke about this with my classmates at uni, and most of us who started coded young comes from that exact same place: wanting to build stuff but lacking money, know-how and an helping environment.

Anyway, I built Cybot (that's its name) until I had a fully working robot that could avoid obstacles, follow light, or a black line on the floor, depending on the selecting mode. It was inspiring, but if the process was awesome, the finished product was just another toy. I left it somewhere hidden in my bedroom for years.

# Starting hacking
Until... Arduino. I was in engineering school studying software development when some friends and I became interested in the Arduino platform. It just seemed fun. Basically, it transformed electronics into software, something I could understand. After I build the basic circuits, I remembered my old robot and the next time I got back to my parents, I took the robot with me. I just wanted to remove all the electronics and plug everything in my Arduino. Well, that's not how you do it... I managed to plug the light sensors to Arduino but nothing else. The motors wouldn't move, the sonars would not send any meaningful data.

[LightCode]

Later on, I became interested in robotics again and I acquired a new robotic platform that could do serve as a base for my grand robotics projects.
It was cheap, and came in a kit easy to assemble. It had 2 motors, and a driver board. You plug the motors to the driver, and you connect Arduino to the driver, not directly the motor.

[Code?]
[Picture?]

I also got an ultrasonic sensor. Much like the motors, you need some kind of driver circuit to send the proper pulses and decode what's coming back. Arduino connects to that circuit, not the sensors.

[Code?]
[Picture?]

# Getting serious

Forward to today, I found Cybot again. I now understood that I should use specific circuits to control the sonars and the motors, that would themselves be controller by Arduino. I browsed the Internet to check if somebody managed to do some kind of project or documentation with their Cybot.

This [website](http://www.lpilsley.co.uk/cybot/driver.htm) is the most complete I could find. I didn't understand it all, but I got how to make the robot move. I kept the first circuit that handle power and motors, and connected my Arduino to it. I just needed to set 4 output pins on my Arduino a bit of code to create a little routine to make it go forward, turn, go back etc.

[CodeOnGithub]
[FritzImage]

To use the sonars, I decided to use the circuits of the HC-SR04 and remove the sensors. I un-solder them and put pins instead. I then plugged the robots sensors to these new circuits and it worked (-ish)! The result is not really precise, there is a lot of noise. I think it's because the sensors are not really good quality to begin with, and the placement is not really good either. I'm thinking about replacing them, but it definitely can detect incoming walls, and on which side. I don't really expected more.

[Code]
[Picture]
[Fritz?]

Having cracked this, I wanted a nice software architecture which would allow me to use the same code on both my robotic platforms. No matter the kind of driver I used, or even sensors. I've always read stuff about robotics and some people give this definition of a robot:

> A device is a robot if it can SENSE its environment, THINK and ACT upon it.

[SenseThinkActDiagram]

A note on the tools I used:
To write good code on Arduino, I prefer to use PlaftormIO, which is much more complete than the Arduino IDE. It facilitates using multiple files and libraries, as well as provides linting, refactoring etc, which are essential tools when you're project becomes bigger than a few lines.

I've actually moved away from Arduino a bit. I mostly uses a WemosD1 Mini board for this project. It is MUCH cheaper (a few bucks), and has builtin WiFi. It is also fully compatible with the Arduino ecosystem which means you've got all the libraries. The downside is that you get fewer input/output pins. Also, it works with a 3V logic instead of 5V for a standard Arduino, which means you've got to use HC-SR04P sensors, which can work with both logics unlike the previous versions without the "P".

To create this, I wrote simple interfaces:

- `Sensors.h`: Defines a few methods to acquire data and give distances.

``` cpp
#define UNDEFINED_PROPERTY -1

class Sensor {
public:
  virtual void acquire() = 0;

  virtual int getDistanceFront() const {
    return UNDEFINED_PROPERTY;
  }
  
  virtual int getDistanceRight() const {
    return UNDEFINED_PROPERTY;
  }
  
  virtual int getDistanceLeft() const {
    return UNDEFINED_PROPERTY;
  }
  
  virtual int getDistanceBack() const {
    return UNDEFINED_PROPERTY;
  }
};
```

- `Actuators.h`: Defines methods to make something move.

``` cpp
class Actuator {
public:
  virtual void forward() = 0;
  virtual void back() = 0;
  virtual void stop() = 0;

  virtual void turnRight() = 0;
  virtual void turnLeft() = 0;
};
```

- `Thinkers.h`: Takes a `Sensor`, an `Actuator` and provides a single method to do stuff.


I then created `CybotMotors` and `DoubleSonar` to implement how these methods work with the Cybot platform. A new platform would simply need a different implementation of these.

The `LocalThinker` implementation just goes forward until it sees something, in which case it turns in the opposite direction and continues.
The plan is to write a `RemoteThinker` which will be a WiFi-enabled "head" for the platform. I started to write code to use MQTT to communicate with a remote RaspberryPi server.
As you can see from the repo, it is easy to add platforms, without changing the obstable avoiding algorithm, we could also use infrared sensors and it wouldn't change a thing for the rest of the codebase.

## Notes
- Which problem I wanted to solve
    - Make a robot (for fun)
    - Using a reusable architecture (2 robots with different motors and sensors)
    - Think - Sense - Act paradigm