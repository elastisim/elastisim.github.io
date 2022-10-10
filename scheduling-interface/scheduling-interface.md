---
layout: default
title: Scheduling interface
nav_order: 6
has_children: true
permalink: /scheduling-interface
---

# Scheduling interface

ElastiSim exposes an interface to forward scheduling decisions during the simulation and thus allows the integration of custom scheduling policies. Following the scheduling protocol, the simulator process periodically invokes the scheduler process. Users must provide an algorithm (function) that returns after modifying arguments passed via the interface. Each invocation of the algorithm contains the following information on the current state of the simulated scenario:

| Parameter             | Description                                                                         | Value type      |
|-----------------------|-------------------------------------------------------------------------------------|-----------------|
| Time                  | Current simulation time                                                             | float (seconds) |
| Jobs                  | List of all jobs submitted at the current simulation time (including finished jobs) | list            |
| Nodes                 | List of all compute nodes of the platform                                           | list            |
| PFS read bandwidth    | Maximum available read bandwidth to the PFS                                         | float (bytes/s) |
| PFS write bandwidth   | Maximum available write bandwidth to the PFS                                        | float (bytes/s) |
| PFS read utilization  | Current utilization of the read bandwidth to the PFS                                | float (bytes/s) |
| PFS write utilization | Current utilization of the write bandwidth to the PFS                               | float (bytes/s) |

## Job properties

In addition to all properties specified by the user (see [Job](/workload/job)), each job in the forwarded list has the following additional properties that the scheduler can use for decision-making:

| Parameter                        | Description                                                      | Value type      | Note                                                                             |
|----------------------------------|------------------------------------------------------------------|-----------------|----------------------------------------------------------------------------------|
| ID                               | Job ID                                                           | integer         | Ascending order based on submission time                                         |
| State                            | Job state                                                        | enum            | -                                                                                |
| Start time                       | Time at which the job started running                            | float (seconds) | -1 if job is pending                                                             |
| End time                         | Time at which the job finished                                   | float (seconds) | -1 if job is pending or running                                                  |
| Wait time                        | Time spent in the queue before running                           | float (seconds) | -1 if job is pending                                                             |
| Makespan                         | Elapsed time from start to end                                   | float (seconds) | -1 if job is pending or running                                                  |
| Turnaround time                  | Elapsed time from submission to end                              | float (seconds) | -1 if job is pending or running                                                  |
| Assigned nodes                   | List of assigned compute nodes                                   | list            | -                                                                                |
| Assigned number of GPUs per node | Assigned number of GPUs per node                                 | integer         | -                                                                                |
| Total phase count                | Total number of phases in the application (including iterations) | integer         | Not counting *on initialization*, *on reconfiguration*, or *on expansion* phases |
| Completed phases                 | Number of completed phases (including iterations)                | integer         | -                                                                                |

### Job states

| State                       | Description                                                                     |
|-----------------------------|---------------------------------------------------------------------------------|
| ``PENDING``                 | Waiting in the queue                                                            |
| ``RUNNING``                 | Running                                                                         |
| ``PENDING_RECONFIGURATION`` | Running and a pending reconfiguration at the next scheduling point              |
| ``IN_RECONFIGURATION``      | Executing the *on reconfiguration* phase                                        |
| ``COMPLETED``               | Job completed gracefully                                                        |
| ``KILLED``                  | Job killed by either exceeding the specified walltime or the scheduler directly |


## Node properties

The following compute node properties are available to the scheduler with each invocation:

| Parameter     | Description                      | Value type | Note                                                                             |
|---------------|----------------------------------|------------|----------------------------------------------------------------------------------|
| ID            | Compute node ID                  | integer    | Ascending order based on the lexicographically sorted list of SimGrid host names |
| Type          | Compute node type                | enum       | -                                                                                |
| State         | Compute node state               | enum       | -                                                                                |
| Assigned jobs | List of assigned jobs            | list       | Number of assigned jobs can be greater than one if oversubscription is enabled   |
| GPUs          | List of GPUs on the compute node | list       | -                                                                                |

### Node types

| Type                                  | Description                                          |
|---------------------------------------|------------------------------------------------------|
| ``COMPUTE_NODE``                      | Compute node without any burst buffer                |
| ``COMPUTE_NODE_WITH_BB``              | Compute node with burst buffers accessed exclusively |
| ``COMPUTE_NODE_WITH_WIDE_STRIPED_BB`` | Compute node with wide-striped burst buffers         |

### Node states

| State         | Description                                                       |
|---------------|-------------------------------------------------------------------|
| ``FREE``      | No jobs running on the compute node                               |
| ``ALLOCATED`` | At least one active job running on the compute node               |
| ``RESERVED``  | No jobs running but expecting a job expanding on the compute node |

### GPUs

#### Properties

| Parameter | Description | Value type | Note |
|-----------|-------------|------------|------|
| State     | GPU state   | enum       | -    |

#### States

| State         | Description |
|---------------|-------------|
| ``FREE``      | GPU unused  |
| ``ALLOCATED`` | GPU in use  |

## Scheduling operations

Custom schedulers can apply the following operations on jobs:

| Operation                      | Arguments             | Description                                                      |
|--------------------------------|-----------------------|------------------------------------------------------------------|
| Assign node                    | node or list of nodes | Assigns the specified node(s) to the job                         |
| Remove node                    | node or list of nodes | Removes the specified node(s) from the job                       |
| Assign number of GPUs per node | integer               | Assigns the number of GPUs per node to use                       |
| Kill                           | none                  | Instructs the batch system to kill the job with immediate effect |
