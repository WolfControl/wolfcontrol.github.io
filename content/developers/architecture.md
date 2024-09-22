+++
title = "System Architecture"
type = "docs"
weight = 1
+++

## Overview

WolfControl is a complex system comprising various components that interact seamlessly to control and monitor all aspects of a controlled environment agriculture (CEA) system. This document provides a high-level overview of the system architecture and the interaction between different components.

## Communication: Clients and Gateways

Wolf Client devices interface with various sensors and actuators to control the environment. These client devices connect to a gateway over a 2.4GHz wireless network using the ESP-NOW protocol. Gateways aggregate data from all their client devices and communicates back to the WolfController over MQTT on WiFi, handling commands and lifecycle management for its clients.

```mermaid
graph
        subgraph C1 [Water Sensor]
            S1["pH Sensor"] -- "I2C" --> E1["ESP32-S3"]
            S2["EC Sensor"] -- "I2C" --> E1
            S3["RTD Sensor"] -- "I2C" --> E1
        end
        subgraph C2 [Quad Peristaltic Pump 1]
            P1["Pump 1"] -- "I2C" --> E2["ESP32-S3"]
            P2["Pump 2"] -- "I2C" --> E2
            P5["Pump 3"] -- "I2C" --> E2
            P6["Pump 4"] -- "I2C" --> E2
        end
        subgraph C3 [Quad Peristaltic Pump 2]
            P3["Pump 1"] -- "I2C" --> E3["ESP32-S3"]
            P4["Pump 2"] -- "I2C" --> E3
            P7["Pump 3"] -- "I2C" --> E3
            P8["Pump 4"] -- "I2C" --> E3
        end
        subgraph G1 [Gateway]
            E4["ESP32-S3"] <-- "UART" --> E5["ESP32-S3"]
        end
        
        E1 -- "ESP-NOW" --> E4
        E2 -- "ESP-NOW" --> E4
        E3 -- "ESP-NOW" --> E4
        E5 <-- "WiFi" --> M1["MQTT Broker"]
```

## Microservices: WolfController

The WolfController is the heart of the WolfControl system, handling data processing, storage, system state management, and providing a user interface for configuration and monitoring. It runs as a set of microservices in Docker containers, hosted on a Raspberry Pi 5.

Sensor readings and device heartbeats are received by the message broker from the gateways (not shown in the diagram) and passed to the appropriate services for processing.

```mermaid
graph LR
subgraph "Container Environment"
    Broker[Message Broker]
    Sense
    Pulse
    Balance
    Compose
    API[API Service]
    WebApp[Web Application]
    Influx[(InfluxDB)]
    Mongo[(MongoDB)]
    Redis[(Redis)]
    end

Broker -- "Sensor Readings" --> Sense --> Influx
Broker -- "Device Heartbeats" --> Pulse --> Redis
Broker -- "Service Update Requests" --> Compose
Broker -- "PID Control" --> Balance

API <--> Mongo

WebApp <--> API <--> Broker
```
