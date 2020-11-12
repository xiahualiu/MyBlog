---
title: "Raspi-Powerchair - Dev Journal One"
date: "2020-11-12"
draft: false
---

This is the first developper journal of my project, using a Raspberry Pi 4B to control an electric wheelchair. This project is partly funded by the IRAS lab at University of Cincinnati and by my own pocket. The project can be found at my GitHub repository [raspi-powerchair](https://github.com/xiahualiu/raspi-powerchair).

<!--more-->

For the first week I have measured the bus signal using a logic analyser, after serveral days I sucessfully extracted all control commands from the communication bus and everything went pretty well. 

![Black line is GND, white line is Tx/Rx, yellow line is unused.](/img/raspi-wheel-dev1/wire_connection.jpg)

![Read the wave from a logic analyser with proper decode setting.](/img/raspi-wheel-dev1/uart_wave.png)

It used a very common communication protocol, UART. However, it is not very common to see this wire configuration, it only uses a single line to transmit and receive data at the same time. Someone called it one wire UART, docs about it aren't very much, but I found a very useful note [here](http://ww1.microchip.com/downloads/en/AppNotes/USART-in-One-Wire-Mode-ApplicationNote-DS00002658.pdf), made by Microchip. 

At first I thought it is just to connect the Tx and Rx port simply but it is not this simple. Normal serial output ports on a microprocessor, including Raspberry Pi, are **push-pull** type. A push-pull type port is able to output very sharp signal edge, so is good for data transmission. However in this push-pull configuration, **only one** output can be connect to the bus. If two outputs are present on the same bus, with different output levels, it will cause undefined behavior, depending on which output has higher drive strength, and eventually it will damage the output ports. 

In one wire configuration like IIC and this case, the output port is configured to be **open-drain** type, this type usually comes with a pull up resistor, and in this configuration:

* When a node's output is 1, the output port is at tri-state, so the pull-up resistor pull the bus up to `Vcc`.
* When a node's output is 0, the output enable the drain mosfet, and short the output pin to ground, the bus shows `GND`.

We can see that in open-drain mode, 0 is stronger than 1, if one node on the line output 0, then the bus will be 0 no matter other outputs.

So the first step is to design a open-drain converter, but luckily there are a few available on market. I am planning to use SN74LVC1G07 from Texas Instrument to implement a open-drain converter. This means some extra job like PCB design has to be done in the next few weeks.

## Protocol Manual for the Powerchair

This protocol manual shall update if new commands are found by me.

| Entry | Value |
| :---  | :---  |
| Type | UART |
| Baud Rate | 38400 |
| Bit Order | LSB First |
| Data Bits | 8 |
| Stop Bits | 1 |
| Parity Check | Even |
| Voltage Level | 5.0V |

### Messages

![Protocol Manual](/img/raspi-wheel-dev1/protocol_manual.svg)
