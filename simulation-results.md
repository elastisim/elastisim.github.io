---
layout: default
title: Simulation results
nav_order: 7
---

# Simulation results

ElastiSim provides detailed results after each simulated scenario as comma-separated value (CSV) files. By enabling the monitoring module and setting a sensing interval (see [Configuration](/configuration)), users gain deeper insight into the utilization of the platform. This page describes the available columns in each CSV file.

## Job statistics

The job statistics file provides the results of each job in the simulated scenario. Each row corresponds to one job.

| Columns         | Value type      | Description                                                                        |
|-----------------|-----------------|------------------------------------------------------------------------------------|
| ID              | integer         | see [Scheduling interface](/scheduling-interface/scheduling-interface)             |
| Type            | string          | see [Job](/workload/job)                                                           |
| Submit Time     | float (seconds) | see [Job](/workload/job)                                                           |
| Start Time      | float (seconds) | see [Scheduling interface](/scheduling-interface/scheduling-interface)             |
| End Time        | float (seconds) | see [Scheduling interface](/scheduling-interface/scheduling-interface)             |
| Wait Time       | float (seconds) | see [Scheduling interface](/scheduling-interface/scheduling-interface)             |
| Makespan        | float (seconds) | see [Scheduling interface](/scheduling-interface/scheduling-interface)             |
| Turnaround Time | float (seconds) | see [Scheduling interface](/scheduling-interface/scheduling-interface)             |
| Status          | string          | Whether the job completed gracefully or was killed (``"completed"`` or ``killed``) |

## Node utilization

The node utilization file provides insights into node state changes during the simulation. Each row describes a change of state and its corresponding node.

| Columns       | Value type      | Description                                   |
|---------------|-----------------|-----------------------------------------------|
| Time          | float (seconds) | Time (absolute) of node state change          |
| Node          | string          | Node name                                     |
| State         | string          | Node state                                    |
| Running jobs  | string          | IDs of running jobs separated by a semicolon  |
| Expected jobs | string          | IDs of expected jobs separated by a semicolon |

## CPU utilization

{: .note }
CPU utilization requires active monitoring.

The monitoring module measures CPU utilization per node. The column count in the resulting CSV file depends on the number of compute nodes.

| Columns      | Value type                 | Description     |
|--------------|----------------------------|-----------------|
| Time         | float (seconds)            | Time (absolute) |
| ``<Node_0>`` | float (percentage, [0..1]) | CPU utilization |
| ...          | ...                        | ...             |
| ``<Node_n>`` | float (percentage, [0..1]) | CPU utilization |

{: .note }
Values represent the utilization of the compute node's total computational power. As CPU workloads fully utilize the compute node, the utilization of a single node is either 0 % or 100 %.

## Network activity

{: .note }
Network activity requires active monitoring.

ElastiSim defines network activity as the sum of the maximum bandwidth of all links in the simulated platform. Realistically, the network activity will never reach 100 %, as it is unlikely that all links in the network topology will be fully utilized simultaneously.

| Columns     | Value type                 | Description      |
|-------------|----------------------------|------------------|
| Time        | float (seconds)            | Time (absolute)  |
| Utilization | float (percentage, [0..1]) | Network activity |

{: .note }
Network activity does not consider PFS or loopback links.

## PFS utilization

{: .note }
PFS utilization requires active monitoring.

The monitoring module senses the utilization of the PFS links specified in the configuration file. The results include absolute and relative utilization (relative to maximum bandwidth).

| Columns      | Value type                 | Description           |
|--------------|----------------------------|-----------------------|
| Time         | float (seconds)            | Time (absolute)       |
| Read         | float (bytes/s)            | PFS read utilization  |
| Write        | float (bytes/s)            | PFS write utilization |
| Read (rel.)  | float (percentage, [0..1]) | PFS read utilization  |
| Write (rel.) | float (percentage, [0..1]) | PFS write utilization |

## GPU utilization

{: .note }
GPU utilization requires active monitoring.

Analogous to CPU utilization, the monitoring module measures GPU utilization per node. The utilization is a relative value and reflects partial usage in the case of multiple GPUs per node.

| Columns      | Value type                 | Description     |
|--------------|----------------------------|-----------------|
| Time         | float (seconds)            | Time (absolute) |
| ``<Node_0>`` | float (percentage, [0..1]) | GPU utilization |
| ...          | ...                        | ...             |
| ``<Node_n>`` | float (percentage, [0..1]) | GPU utilization |
