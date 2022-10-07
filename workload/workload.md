---
layout: default
title: Workload
nav_order: 5
has_children: true
permalink: /workload
---

# Workload

ElastiSim divides workloads into two entities, jobs and application models. While jobs define scheduling-related parameters such as the requested number of nodes and time limits, application models represent the application executed on the simulated platform. Application models hold multiple phases, and each phase comprises multiple tasks. The entirety of simulated jobs—and their corresponding application models—constitutes the workload and defines the simulation scenario. Application models support performance models (i.e., human-readable mathematical functions) to model elastic workloads that can scale with the number of assigned resources.

![A figure describing the workload structure](/assets/images/Workload_structure.svg "Workload structure"){: .d-block .mx-auto width="75%" }
