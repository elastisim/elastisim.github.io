---
layout: default
title: Get started
nav_order: 2
has_children: true
permalink: /get-started
---

# Get started

ElastiSim is a discrete-event simulator composed of two processes—a simulator and a scheduler process. While the simulator process is the primary process responsible for the simulation of the platform, the workload, and all system actors, the scheduler process is responsible for making scheduling decisions based on the provided algorithm.

To run simulations, users must provide a [platform description](/platform/),
a [workload specification](/workload/), and a [configuration file](/configuration). Users can design and simulate individual scheduling policies using a scheduling interface that ElastiSim provides. The simulator process starts the simulation and communicates with the scheduler process based on the scheduling protocol. After the simulation, users can inspect the multiple results ElastiSim provides.

![A figure describing ElastiSim's software architecture](/assets/images/ElastiSim_architecture.svg "ElastiSim architecture"){: .d-block .mx-auto width="80%" }
