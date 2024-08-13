---
title: Introduction to Zephyr RTOS - Why Arduino shift to Zephyr
description: Introduction to Zephyr RTOS - Rich features and support multiple architectures.
author: liam
date: 2024-08-06 23:24:00 +0800
categories: [Zephyr, Arduino]
tags: [RTOS, Zephyr, Arduino, Mbed]
pin: false
math: true
mermaid: true
image:
  path: common/img/posts/20240806/arduino_zephyr_logo.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Arduino's Transition to Zephyr
---

## Arduino.cc has announced the end of Mbed Platform

Mbed's End and the Shift to Zephyr: Impact on Arduino and Beyond
Arm's decision to discontinue Mbed in 2026 has sent ripples through the embedded systems community. One of the most significant impacts is on Arduino, which has relied on Mbed for several of its boards.

### Arduino's Transition to Zephyr
To address this, Arduino is migrating its Mbed-based boards to the Zephyr Real-Time Operating System (RTOS). This strategic move ensures the continuity of development for popular boards like the Arduino GIGA, Nano 33 BLE, and Portenta families.

### Benefits of Zephyr:
- Active community and development
- Wide range of supported hardware
- Strong focus on real-time performance and low power consumption


### Timeline:
- End of 2024: First beta release of Arduino core based on Zephyr
- 2025: Rollout for various boards

### Implications for Developers
While this transition may require adjustments for developers, the long-term benefits are promising:

- Familiar Arduino environment: Arduino plans to maintain its core structure and APIs, making the shift smoother for existing users.
- Expanded capabilities: Zephyr's features and performance improvements can open up new possibilities for embedded projects.
- Community support: Arduino and Zephyr have strong communities that provide resources and assistance.

### Beyond Arduino
The impact of Mbed's end extends beyond Arduino. Other embedded systems developers relying on Mbed must explore alternative platforms and RTOS options. Zephyr, with its growing ecosystem, is likely to become a popular choice.

### Key Takeaways:

- Mbed's end of life is imminent.
- Arduino is proactively migrating to Zephyr to ensure long-term support for its boards.
- Zephyr's capabilities and community make it a strong contender for replacing Mbed.
- Embedded developers should evaluate their options and consider Zephyr as a potential solution.

Would you like to know more about specific boards, the transition process, or how to get involved in the Zephyr community?

### Read More

[1]. [The end of Mbed marks a new beginning for Arduino](https://blog.arduino.cc/2024/07/24/the-end-of-mbed-marks-a-new-beginning-for-arduino)

[2]. [Zephyr Project Welcomes Analog Devices, Arduino and Technology Innovation Institute [...]](https://zephyrproject.org/zephyr-project-welcomes-analog-devices-arduino-and-technology-innovation-institute-as-new-members/)