---
title: "Failure and plan b - 2nd Dev Journal"
date: "2020-12-21"
draft: false
---

This is the second developper journal of my project, using a Raspberry Pi 4B to control an electric wheelchair. This project is partly funded by the IRAS lab at University of Cincinnati and by my own pocket. The project can be found at my GitHub repository [raspi-powerchair](https://github.com/xiahualiu/raspi-powerchair). The first post can be found at [xiahualiu.github.io](https://https://xiahualiu.github.io/posts/raspi-wheel-dev1/).

<!--more-->

## A big failure

The reason why I did not update until a month has passed since the first dev journal is that, not only due to the end-semaster tests and projects, the attempt to communicate with the motor controller ended with a catatrophic failure, I tried everything I could think of, the motor controller did not respond to any messages I gave.

The main reason is that the output physical configuration on the original controller is pretty weird, it seems to use a push-pull type configuration, but this would potentially pose danger to damage the output driver. 

## Lesson learnt

Never think something is sure to work until understand how does it work. Because everyone has its own mind of designing a function.

## Next step - plan B

Plan B is designing the motor controller myself, it will take more time but it is sure to work because I just directly control the motors.

The schematics have been finished, see my circuitmaker project [raspi-powerchair](https://circuitmaker.com/Projects/Details/XiahuaLiu/Raspi-powerchair).
