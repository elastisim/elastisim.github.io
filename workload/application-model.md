---
layout: default
title: Application model
parent: Workload
nav_order: 2
---

# Application model

Application models have a strict hierarchical structure and follow a building-block approach comprising *phases* and *tasks*. While phases represent stages within an application, tasks represent the load executed on the simulated platform.

To allow dynamic reconfigurations during runtime, ElastiSim introduces *scheduling points*. Malleable applications reaching a scheduling point can adapt to new resources if the scheduler has decided to reconfigure the job. Thus, dividing applications into phases and tasks allows for modeling malleable workloads. ElastiSim implicitly includes scheduling points at the end of each phase.

![A figure describing ElastiSim's application execution model of a malleable job](/assets/images/Application_execution.svg "Application execution model of a malleable job"){: .d-block .mx-auto width="75%" }

{: .note }
Scheduling decisions to reconfigure a job are not applied immediately but when the job reaches its next scheduling point. If a job has no further scheduling point, the reconfiguration remains unapplied.

## Phases

Users define phases by specifying the following properties using the JSON format:

| Property                   | Description                                                                                                          | Value type | Default value | Mandatory |
|----------------------------|----------------------------------------------------------------------------------------------------------------------|------------|---------------|-----------|
| ``phases``                 | Array containing all phases (top-level structure)                                                                    | array      | -             | Yes       |
| ``iterations``             | Number of iterations (i.e., repetitions) of the phase                                                                | integer    | 0             | No        |
| ``scheduling_point``       | Whether a scheduling point is included at the end of (each iteration of) the phase                                   | bool       | true          | No        |
| ``final_scheduling_point`` | Whether the final scheduling point is included                                                                       | bool       | true          | No        |
| ``barrier``                | Whether there is a barrier at the end of the phase (only considered when there is no corresponding scheduling point) | bool       | true          | No        |
| ``tasks``                  | Array holding the tasks of the phase                                                                                 | array      | -             | Yes       |

### Reconfiguration penalty

Reconfiguring applications during runtime can introduce additional overhead while the application adapts to its new configuration (e.g., data redistribution). ElastiSim defines that overhead as the *reconfiguration penalty* and provides two particular phases executed that can reflect such overhead. The *on reconfiguration* phase runs on all assigned resources on each reconfiguration. In contrast, the *on expansion* phase runs only on newly assigned resources. As applications also might have an initialization phase, ElastiSim provides an additional *on initialization* phase executed only on the first configuration of a job to facilitate the modeling of all stages during application execution.

| Phase                  | Description                                                        |
|------------------------|--------------------------------------------------------------------|
| ``on_init``            | Executed only on the first job configuration (i.e., once in total) |
| ``on_reconfiguration`` | Executed on each reconfiguration on all resources                  |
| ``on_expansion``       | Executed on each reconfiguration only on newly assigned resources  |

## Tasks

Analogously, tasks are defined with the following properties:

| Property         | Description                                                                                                                        | Value type | Default value | Mandatory |
|------------------|------------------------------------------------------------------------------------------------------------------------------------|------------|---------------|-----------|
| ``name``         | Name of the task (only relevant in log messages)                                                                                   | bool       | None          | No        |
| ``iterations``   | Number of iterations (i.e., repetitions) of the task                                                                               | integer    | 0             | No        |
| ``synchronized`` | Whether all resources (i.e., compute nodes) synchronize before executing the task (similar to an *MPI_Barrier()* before execution) | bool       | false         | No        |
| ``type``         | Task type (see [Task types](/workload/task-types))                                                                                 | bool       | true          | No        |

## Example application model

An example of an application model with ten repetitions of a compute (CPU) and PFS write task.

{: .note }
See [Task types](/workload/task-types) for detailed task descriptions.

```json
{
  "on_init": {
    "tasks": [
      {
        "type": "pfs_read",
        "name": "PFS read",
        "bytes": 8e10,
        "pattern": "all_ranks"
      }
    ]
  },
  "phases": [
    {
      "iterations": 10,
      "tasks": [
        {
          "type": "cpu",
          "name": "Compute",
          "flops": 8e12,
          "computation_pattern": "uniform"
        },
        {
          "type": "pfs_write",
          "name": "PFS write",
          "bytes": 5e10,
          "pattern": "all_ranks"
        }
      ]
    }
  ]
}
```
