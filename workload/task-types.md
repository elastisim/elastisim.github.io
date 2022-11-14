---
layout: default
title: Task types
parent: Workload
nav_order: 3
---

# Task types

Each task has a specific type defining the injected load into the simulated platform, which, depending on its type, has different properties specifying its execution behavior. This page summarizes all task types and their corresponding properties and introduces how ElastiSim distributes the simulated load on resources.

## Task payloads & distribution patterns

All tasks carry a load simulated on the platform, which ElastiSim defines as the task's *payload*, and the unit of a payload depends on the task type (e.g., FLOPS for compute tasks or bytes for communication). Payloads are always defined using a single number and distributed among participating resources following a *payload distribution pattern*.

ElastiSim defines two types of distribution patterns: *vector* and *matrix*. While vector distribution patterns consider a one-dimensional distribution (e.g., FLOPS per compute node), matrix distribution patterns define communication matrices.

### Vector distribution patterns

| Pattern                                    | Description                                                                                             |
|--------------------------------------------|---------------------------------------------------------------------------------------------------------|
| ~~``total``~~ *(deprecated)* ``all_ranks`` | The payload is evenly distributed among all resources                                                   |
| ``root_only``                              | Only the first resource performs the specified payload                                                  |
| ``even_ranks``                             | The payload is evenly distributed among all even-numbered resources                                     |
| ``odd_ranks``                              | The payload is evenly distributed among all odd-numbered resources                                      |
| ``uniform``                                | All assigned resources perform the specified payload *without any distribution*                         |
| ``vector``                                 | An explicit vector defining the payload for each participating resource (only applicable to rigid jobs) |

{: .note }
``uniform`` is the only exception to other distribution patterns (vector and matrix), as it does not distribute the workload but describes the payload *per resource*. It is syntactic sugar for the pattern ``all_ranks`` with the performance model ``<payload size> * num_nodes`` or ``<payload size> * num_gpus``.

### Matrix distribution patterns

| Pattern                    | Description                                                                                                                                                                 |
|----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ``all_to_all``             | Each resource communicates bi-directionally with every other resource                                                                                                       |
| ``gather``                 | The first resource receives uni-directionally from all remaining resources                                                                                                  |
| ``scatter``                | The first resource sends uni-directionally to all remaining other resources                                                                                                 |
| ``master_worker``          | The first resource bi-directionally communicates with all remaining resources                                                                                               |
| ``ring``                   | Each resource communicates bi-directionally with its direct neighbors                                                                                                       |
| ``ring_clockwise``         | Each resource communicates uni-directionally with its right neighbor                                                                                                        |
| ``ring_counter_clockwise`` | Each resource communicates uni-directionally with its left neighbor                                                                                                         |
| ``matrix``                 | An explicit matrix defining the payload for each possible pair of resources (defined as a vector with the dimension #resources × #resources, only applicable to rigid jobs) |

## CPU computation & communication task

ElastiSim divides computational tasks into two parts: compute and communication. The reasoning behind this twofold structure is to allow overlapping and coupling computation and communication. Each assigned node computes the load based on its computational capabilities and communicates using the links defined by the underlying topology. However, users are not required to specify both payloads. Specifying only a computation or communication is valid and will only simulate the specified payload. By setting the ``type`` property to ``cpu``, the following properties get available:

| Property                  | Description                                                                                                                   | Value type                  | Default value | Mandatory                          |
|---------------------------|-------------------------------------------------------------------------------------------------------------------------------|-----------------------------|---------------|------------------------------------|
| ``flops``                 | Computational load of the task                                                                                                | integer (FLOPS)             | -             | Yes, if ``bytes`` is not specified |
| ``computation_pattern``   | Payload distribution pattern of the computational load                                                                        | vector distribution pattern | -             | Yes, if ``flops`` is specified     |
| ``bytes``                 | Communication load of the task                                                                                                | integer (bytes)             | -             | Yes, if ``flops`` is not specified |
| ``communication_pattern`` | Payload distribution pattern of the communication load                                                                        | matrix distribution pattern | -             | Yes, if ``bytes`` is specified     |
| ``coupled``               | Whether computation and communication is strictly coupled (i.e., bound by the slowest resource among all participating nodes) | bool                        | false         | No                                 |

### Example

```json
{
  "type": "cpu",
  "name": "CPU compute & communication",
  "flops": 8e11,
  "computation_pattern": "uniform",
  "bytes": 5e10,
  "communication_pattern": "all_to_all",
  "coupled": true
}
```

## GPU computation & communication task

Analogous to CPU tasks, GPU tasks also comprise computation and communication. However, as compute nodes can be equipped with multiple GPUs, the communication among GPUs takes place using intra- or inter-node communication. Depending on the platform topology, ElastiSim automatically utilizes the correct links. The ``type`` property to ``gpu``, supports the following properties:

| Property                  | Description                                            | Value type                  | Default value | Mandatory                          |
|---------------------------|--------------------------------------------------------|-----------------------------|---------------|------------------------------------|
| ``flops``                 | Computational load of the task                         | integer (FLOPS)             | -             | Yes, if ``bytes`` is not specified |
| ``computation_pattern``   | Payload distribution pattern of the computational load | vector distribution pattern | -             | Yes, if ``flops`` is specified     |
| ``bytes``                 | Communication load of the task                         | integer (bytes)             | -             | Yes, if ``flops`` is not specified |
| ``communication_pattern`` | Payload distribution pattern of the communication load | matrix distribution pattern | -             | Yes, if ``bytes`` is specified     |

### Example

```json
{
  "type": "gpu",
  "name": "GPU compute & communication",
  "flops": 8e12,
  "computation_pattern": "all_ranks",
  "bytes": 7e10,
  "communication_pattern": "ring_clockwise"
}
```

## I/O tasks

All I/O tasks follow the same structure and define the operation (read or write) and the target of the operation (PFS or node-local burst buffer).

| ``type``      | Description                             |
|---------------|-----------------------------------------|
| ``pfs_read``  | Read operation targeting the PFS        |
| ``pfs_write`` | Write operation targeting the PFS       |
| ``bb_read``   | Read operation targeting burst buffers  |
| ``bb_write``  | Write operation targeting burst buffers |

In contrast to compute tasks, I/O tasks support asynchronous execution among the following properties:

| Property    | Description                                      | Value type                  | Default value | Mandatory |
|-------------|--------------------------------------------------|-----------------------------|---------------|-----------|
| ``bytes``   | I/O size                                         | integer (bytes)             | -             | Yes       |
| ``pattern`` | Payload distribution pattern of the I/O size     | vector distribution pattern | -             | Yes       |
| ``async``   | Whether the operation is executed asynchronously | bool                        | false         | No        |

### Example

```json
{
  "type": "pfs_write",
  "name": "PFS write",
  "bytes": 5e11,
  "pattern": "all_ranks"
}
```

## Delay tasks

Delay tasks are generic tasks occupying the compute node for a given amount of time, which can be useful to represent any task when computation, communication, or I/O tasks can not appropriately model the application. ElastiSim has two flavors of delay tasks representing either an idling or a busy wait activity. While idle tasks occupy compute nodes without resource utilization, busy wait tasks fully utilize the compute capabilities. Setting the ``type`` property to either ``idle`` or ``busy_wait`` introduces the following property:

| Property  | Description                        | Value type        | Default value | Mandatory |
|-----------|------------------------------------|-------------------|---------------|-----------|
| ``delay`` | Period of time to occupy resources | integer (seconds) | -             | Yes       |

### Example

```json
{
  "type": "busy_wait",
  "name": "Busy wait",
  "delay": 720,
  "pattern": "uniform"
}
```

## Task sequences

Task sequences are simple containers that are especially useful when used for repeated execution of a specific sequence. The ``type`` to ``sequence`` defines a task sequence and makes the following property available:

| Property    | Description    | Value type                  | Default value | Mandatory |
|-------------|----------------|-----------------------------|---------------|-----------|
| ``tasks``   | Array of tasks | array                       | -             | Yes       |

{: .note }
ElastiSim defines sequences recursively, allowing them to be nested.

### Example

```json
{
  "type": "sequence",
  "iterations": 12,
  "tasks": [
    {
      "type": "cpu",
      "flops": 8e10,
      "computation_pattern": "uniform"
    },
    {
      "type": "pfs_write",
      "name": "PFS write",
      "bytes": 6e10,
      "pattern": "all_ranks"
    }
  ]
}
```

## Resource contention

All tasks in ElastiSim (except ``idle``) utilize resources. While compute capabilities can be exclusively available to jobs if oversubscription is disabled (see Configuration), network communication depends on the underlying platform topology. The simulation engine evenly distributes the bandwidth of shared links when utilized by multiple jobs or overlapping asynchronous I/O tasks. However, if oversubscription is enabled, jobs can share computational resources. While CPUs are shared evenly and immediately with the execution of a new task, GPUs (and intra-node links) are utilized exclusively following a first come, first serve policy.

{: .note }
As ``busy_wait`` tasks utilize the compute capabilities of a node, multiple jobs oversubscribing the same node with a ``busy_wait`` (or even ``cpu``) task will compete for resources (e.g., two ``busy_wait`` tasks of 15 minutes will take 30 minutes to finish).
