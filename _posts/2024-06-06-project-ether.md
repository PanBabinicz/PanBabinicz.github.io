---
title: Project Ether
date: 2024-06-06 06:30:00 +0200
categories: [embedded,software,hardware,mqtt,wireless,c]
tags: [embedded,software,hardware,mqtt,wireless,c]
---

# PROJECT ETHER

## ❓ What is ETHER?

▶️ [Cambridge dictionary:](https://dictionary.cambridge.org/dictionary/english/ether)

> the sky or the air, especially considered as being full of radio waves

▶️ [The fantasy wiki with fictional characters:](https://gods-and-demons.fandom.com/wiki/Aether_(Deity))

> ***"Hi! My name is Aether and I am one of the Protogenoi. I'm the Lord of the stratosphere,
> or as I call it, the Upper Air, Bringer of Light, and the eldest son of Darkness and Night.
> Today, I will be your captain and guide as we explore the strange but fantastic world of Olympus.
> So hold tight to your seatbelts, 'cause this is gonna be one hell of a ride!"***
>
> Aether the god himself is the embodiment of the upper sky, the stratosphere is his domain and 
> with it the air that the gods breathe from. Every world has its own air for its creatures to breathe,
> the underworld is breathed by its monsters, the Human Realm is breathed by its humans, and the Skyland 
> is breathed by the divine beings who control reality; that is the air that Aether controls. It's not just 
> the air of Skyland that he controls however, a portion of the Earth's sky belongs to him as well. The Earthly sky 
> isn't the only cosmic body that Aether can control, the Moon, Stars, Clouds, and partially even the Sun can be controlled by him.

▶️ [The Concept of 'Ether' in Radio Wave Propagation:]

> In the context of radio waves, "ether" refers to the medium through which radio waves propagate.
> This medium is often conceptualized as the space surrounding us, particularly the atmosphere or the open air.
> In the early days of radio communication, scientists believed that radio waves traveled through a substance called
> "ether," filling the space much like air or water fills physical space. While the concept of a physical ether 
> was later debunked by advancements in physics, the term "ether" is still used metaphorically to describe the 
> invisible medium through which radio waves travel.

## ⚒️  Project goal 

> ***The main goal of the project is to create a comprehensive air quality monitoring system, 
> based on the ESP32 platform and integrated with OpenHAB and Grafana. The plan involves the development of 
> a dedicated hardware device equipped with ESP32, appropriate sensors measuring carbon dioxide, temperature, and humidity, 
> as well as a power supply unit. Additionally, the project envisages the use of the MQTT protocol,
> enabling efficient communication between devices, ensuring fast and reliable transmission of sensor data to the server.
> OpenHAB and Grafana will be deployed on the Raspberry Pi platform.***

> [!NOTE]
> **Please bear in mind that the project was primarily created for enjoyment and educational endeavors.**

## Idea Schematic

![IDEA-SCHEMATIC](https://github.com/PanBabinicz/project-ether/blob/main/doc/screenshots/idea-schematic.png?raw=true)

## 🔌 Hardware

![Front](https://github.com/PanBabinicz/project-ether/blob/PROJECT-ETHER-REFACTOR/hardware/project-ether-front.png?raw=true)

![Back](https://github.com/PanBabinicz/project-ether/blob/PROJECT-ETHER-REFACTOR/hardware/project-ether-back.png?raw=true)

> The hardware for this project was meticulously designed using KiCad, a popular open-source electronic design automation software. 
> The board is a 4-layer design featuring the ESP32-WROOM-32 module at its core. Additionally, it integrates key sensors such as the BME280 and SCD41,
> enabling environmental monitoring capabilities.
>
> The device can be conveniently powered either via USB Type-C connection or an external battery, providing flexibility for different usage scenarios and environments.

## 📖 Software

### Logic Analyzer Output

#### BME280

![BME280](https://github.com/PanBabinicz/project-ether/blob/main/doc/screenshots/bme280.png?raw=true)

#### PMS7003

![PMS7003](https://github.com/PanBabinicz/project-ether/blob/main/doc/screenshots/pms7003.png?raw=true)
