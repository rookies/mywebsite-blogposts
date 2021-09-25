---
title: Temperature-dependent fan control using Arduino
published: 2020-10-31T12:20:00+01:00
mainImg: board_front.jpg
mainImgOffset: 40
github: https://github.com/rookies/fancontrol
tags:
  - programming
  - electronics
...
I'm building a box to dampen the noise of the projector I bought (more on that
in a later post) and to keep it cool I added two fans, one sucking cold air
in and one pushing warm air out.
To reduce the noise as much as possible I wanted to run the fans as slow as
possible, but of course they should still run fast enough to properly cool
the projector.
So I built a little fan controller with an Arduino that uses temperature to
determine the fan speed.

It features an Arduino Nano and a DS18B20 1-Wire temperature sensor that is
very easy to use and needs no additional components except of one
UNIT(4.7,kOhm) pull-up resistor.
The board is powered with a UNIT(12,V) power supply that is fed into the VIN
pin of the Arduino and directly connected to the fan.
The PWM input of the fan is directly connected to a PWM output of the
Arduino.
This is only possible if you're using a 4-pin fan (that has a separate PWM
input), but for a 3-pin fan you could drive a transistor with the Arduino
that controls the supply voltage of the fan via PWM instead of supplying it
with constant UNIT(12,V).

I mounted all the components on a small piece of prototyping board, not the
fanciest solution but it works and I like it for such small projects.
In the background you can see the fan and a sneak peek of the (unfinished)
box for my projector.
The fan plugs directly into the board and conveniently has a connector where
you can add another fan.
The temperature sensor with its pull-up resistor is on the left-hand side of
the board, the power input on the right.

![Front of the board](board_front.jpg)
![Back of the board](board_back.jpg)

The software for the project is
[on my GitHub](https://github.com/rookies/fancontrol).
You can configure a minimum temperature at which the fan should start and a
maximum temperature where it should reach its full speed.
For temperatures between those two limits the speed is proportional to the
temperature.
The minimum and maximum speeds of the fan are configurable, which is useful
if your fan needs a minimum PWM duty cycle to start or if you want to limit
the speed of your fan, respectively.

There's also a small safety feature: if the temperature drops below a
configurable threshold, the controller assumes that there's an error, ignores
the temperature reading, and runs the fan at full speed.
This could e.g. happen if the connection to the temperature sensor gets
interrupted.
But keep in mind that this is still a flimsy Arduino project which you should
definitely not use for critical applications!
So don't use it to cool your DIY nuclear reactor.

![Influence of the temperature on the fan PWM value](temperature_pwm.svg)

But wait, there's another feature I didn't tell you about so far.
The temperature sensor typically delivers values at a resolution of about
UNIT(0.5,degCdiff), which (depending on your configuration) leads to a
relatively large jump of the used PWM value.
If it's really quiet you might be able to hear that and especially if it
jumps back and forth that can get annoying pretty quickly.
To fix this the software slowly approaches the desired PWM value with a
configurable maximum change rate.
If the desired change however exceeds a configurable threshold, the target
is set immediately to prevent a very long delay.

You can see this in the plot below, at the first jump the change is small so
it is approached slowly.
The second jump however is large enough to exceed the threshold, therefore
it is applied instantly.

![Slow approach of the PWM target value](pwm_approach.svg)
