---
layout: base.ejs
title: Theremin | Data Structures
---

# Theremin

_[GitHub](https://github.com/rijkvanzanten/ds-fa-3) • [Live Demo](https://ds-fa-3.rijks.website)_

## Brief

Create a web-based interactive visual representation for the real-time data generated by the sensor you were assigned.

## Accelerometer

I was assigned the accelerometer, a sensor that measures gravitational pull on three axis: x, y, and z. With this data, you can measure if something moves, and roughly how. It can work pretty okay as a way to measure tilt, but it's not the most accurate.

The idea of the assignment was to put your sensor somewhere in your home and have it collect data over a long period of time. What can you measure in your home with an accelerometer? Eg, what moves? My mind went to a pet or a roommate, but the sensor has to be plugged in for power, so you can track something that moves and is stationary. For me, that would mean either the front door, or the fridge. Not the _most_ interesting thing out there.

I figured I could try to make this a little more interesting...

## Taking the Unbeaten Path

I remember my first iPod. It was an iPod Touch with a massive screen (tiny by today's standards) and an accelerometer. There was this little game where you would move a ball through a wooden maze by tilting the whole device. My mind was blown.

I wanted to build something like that. By treating my sensor as an input device rather than a stationary data gathering device, a whole new world of possibilities opened up.

I thought of flying a drone using the accelerometer as the controller. I did run into a few issues with that idea though:

* Flying a drone is insanely hard
* An accelerometer isn't near accurate enough for that kind of precision input
* I couldn't risk decapitating my classmates during the final presentation

Then I remembered that I was able to track changes in velocity in all three directions. After another long brainstorming session, I came up with the idea to try creating a theremin:

<iframe width="560" height="315" src="https://www.youtube.com/embed/K6KbEnGnymk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

This means that I would have to track movement in both the up-and-down direction (volume) as left-and-right (frequency). It also meant that I had to learn to work with the WebAudio API and figure out how synthesizers work. Fun!

## Getting real-time data

In order to make actual music, I had to get data from my sensor at an extremely fast rate. The way that was tought during class (where you would poll the chip every 5 minutes or so) was obviously not going to work. _I needed more_. Instead of once every 5 minutes, I needed my sensor value like every.. 50 milliseconds or so.

![Me in Undergrad](/th-me.png)

I have worked with Arduino's before at my undergrad program. For the projects there, I had been using the serial connection (USB) to get insanely fast data throughput. While that technically gets the job done, I wanted to see if I could get a similar steady real time stream of data over the internet.

## WebSockets

The goal was simple:

1. Read the values from the sensor
2. Pass it to the server
3. Server reads it and passes it to all the connected websites

This flow would make it possible for all connected websites (eg all the computers in the class) to listen in on the performance of the artist with the Photon chip. However, this simple looking flow gets unnecessarily difficult at 20 times a second.

I have worked with WebSockets before to stream real time data from the server to the connected clients, and I have successfully connected an IoT board (in Lua) to said server. However, the Photon didn't play along nicely. The only WebSockets library available for the board didn't support SSL encrypted WSS channels, which made it impossible for me to use with my server (as browsers start complaining about SSL missing when getting data from a WebSocket).

I hit a roadblock..

## HTTP

The Photon chip can send HTTP requests which could've been a way to get the sensor value to the server. However, a HTTP request has a tremendous amount of overhead (headers) compared to the amount of data that we're sending (1 number). This big overhead combined with the fact that every HTTPS request to the server had multiple back and forth requests (to confirm the certificate), the amount of latency is simply too much. Waiting 50ms for a request that happens every 50ms causes _a lot_ of issues. I hit another roadblock..

## TCP

I didn't need a way to _send_ data; I needed a way to _stream_ data. Have a steady connection with the server over which I could constantly send values. TCP was the answer. TCP, or Transmission Control Protocol, is a standard that defines how to establish and maintain a network conversation via which applications can exchange data. Perfect! I had never worked with it before, but to my surprise Node has (just like `http`) a native `net` module that handles TCP servers. There was a slight problem though: browser don't support TCP.

## HTTPS + WSS + TCP = ❤️

I got myself in a bit of a complex situation. My one Node application now basically had three different protocols it's supporting. HTTPS for serving the actual front-end, WSS for having a real time connection with the connected browser clients, and a TCP port for the incoming sensor data. The flow is actually surprisingly simple: every time a new value is shot over the TCP socket, the server converts the buffer to the value and passes it to all connected browser. The browser will then calculate the difference between this and the previous sensor value and adjust the oscillators frequency based on that.

## Backpressure

There is a problem that occurs during data handling called backpressure. This is when a buffer gets "clogged up" during data transfer. This happens when the receiving end (in this case my Node server) is slower than the sending party (the Photon chip). I noticed that my Node server would clog up every now and again, and start spitting out massive values all at once instead of a steady stream. I immediately pointed my finger at the 50ms rate of data, but after tweaking the values to a way slower interval (going as slow as twice a second) I realized something else was going on: the Node server didn't know when a new data packet had arrived. The stream of data was at a pretty constant interval, but it would ocassionally happen that two data packets would arrive at the same time. In that case, the Node server would return something like: `2011`-`2010`-`20112014`-`2010`. I managed to solve this by sending a stop character after each value coming from the sensor, so the Node app has something to reference. However, even after this optimization and some other performance optimizations (like only sending a data packet if the sensor value had changed), the Node server still clogs up sometime. I honestly have no clue what's causing it or how to solve it. It seems to happen more when a new browser has connected fresh, and it seems to go away after having had a steady stream of data for about a minute.

## Result

![Result](/th-1.png)

<video src="/th-vid.mp4" controls></video>

[Live demo](https://ds-fa-3.rijks.website)\*

\* There isn't really that much to see without me operating my Photon..

## Conclusion

I have been fighting sensor data, network engineering, and basic maths for way too long. I'm super super super happy that I managed to get it working as I had invisioned, even though a shitty network or another backpressure can really screw with the result. I have learned a great deal about data streaming, buffers, TCP, the web audio API, and visualizing wavelengths. (I had no clue how to draw a sinewave before this).

### Future improvements

* Instead of sending the raw sensorvalue to all browsers, the server should send the new frequency. This will make sure all connected browsers always blast the correct note.
* I create a new event listener for incoming data for all connected browsers, this is a possible memory leak as they're not cleared yet. I should change that.
* I want to create a mode where the user can use their phone instead of a Photon chip. Right now, you need a very specific set of gear to use the visualization.
* Create a better demo video
