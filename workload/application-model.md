---
layout: default
title: Application model
parent: Workload
nav_order: 2
---

# Application model

Application models have a strict hierarchical structure and follow a building-block approach comprising *phases* and *tasks*. While phases represent stages within an application, tasks represent the load executed on the simulated platform.

To allow dynamic reconfigurations during runtime, ElastiSim introduces *scheduling points*. Malleable applications reaching a scheduling point can adapt to new resources if the scheduler has decided to reconfigure the job. Thus, dividing applications into phases and tasks allows for modeling malleable (and evolving) workloads. ElastiSim includes scheduling points by default at the beginning of each phase (unless specified otherwise).

![A figure describing ElastiSim's application execution model of a malleable job](/assets/images/Application_execution.svg "Application execution model of a malleable job"){: .d-block .mx-auto width="75%" }

{: .note }
Scheduling decisions not taken at a job's scheduling point (e.g., periodic invocation) will be applied when the job reaches its next scheduling point. If a job has no further scheduling point, the reconfiguration remains unapplied.

## Phases

Users define phases by specifying the following properties using the JSON format:

| Property                                    | Description                                                                                                                                    | Value type                                                                 | Default value | Mandatory |
|---------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------|---------------|-----------|
| ``phases``                                  | Array containing all phases (top-level structure)                                                                                              | array                                                                      | -             | Yes       |
| ``iterations``                              | Number of iterations (i.e., repetitions) of the phase (has to be inferable at start time)                                                      | integer or string (see [Performance models](/workload/performance-models)) | 0             | No        |
| ``scheduling_point``                        | Whether a scheduling point is included before (each iteration of) the phase                                                                    | bool                                                                       | true          | No        |
| ``evolving_request``                        | Evolving request specifying the number of nodes to issue before (each iteration of) the phase (unspecified implies no evolving request)        | integer or string (see [Performance models](/workload/performance-models)) | unspecified   | No        |
| ~~``final_scheduling_point``~~ (deprecated) | ~~Whether the final scheduling point is included~~                                                                                             | ~~bool~~                                                                   | ~~true~~      | ~~No~~    |
| ``barrier``                                 | Whether there is a barrier at the beginning of the phase (only considered when there is no corresponding scheduling point or evolving request) | bool                                                                       | true          | No        |
| ``tasks``                                   | Array holding the tasks of the phase                                                                                                           | array                                                                      | -             | Yes       |

{: .important }
By definition, applications do not hold scheduling points or evolving requests before the first phase (for phases specified in the ``phases`` array).

{: .note }
ElastiSim clips evolving requests by default to stay in the possible range of configurations (i.e., [``num_nodes_min``, ``num_nodes_max``], see [Configuration](/configuration)). Applications consider evolving requests only if the number of requested nodes differs from the number of assigned nodes. In those cases, invoking the scheduler with an evolving request is mandatory. For adaptive jobs, if a phase specifies an evolving request and a scheduling point, the evolving request has a higher priority and will take precedence over the scheduling point.

### Reconfiguration penalty

Reconfiguring applications during runtime can introduce additional overhead while the application adapts to its new configuration (e.g., data redistribution). ElastiSim defines that overhead as the *reconfiguration penalty* and provides two particular phases executed that can reflect such overhead. The *on reconfiguration* phase runs on all assigned resources on each reconfiguration. In contrast, the *on expansion* phase runs only on newly assigned resources. As applications also might have an initialization phase, ElastiSim provides an additional *on initialization* phase executed only on the first configuration of a job to facilitate the modeling of all stages during application execution.

| Phase                  | Description                                                        |
|------------------------|--------------------------------------------------------------------|
| ``on_init``            | Executed only on the first job configuration (i.e., once in total) |
| ``on_reconfiguration`` | Executed on each reconfiguration on all resources                  |
| ``on_expansion``       | Executed on each reconfiguration only on newly assigned resources  |

## Tasks

Analogously, tasks are defined with the following properties:

| Property         | Description                                                                                                                        | Value type                                                                 | Default value | Mandatory |
|------------------|------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------|---------------|-----------|
| ``type``         | Task type (see [Task types](/workload/task-types))                                                                                 | string                                                                     | -             | Yes       |
| ``name``         | Name of the task (only relevant in log messages)                                                                                   | string                                                                     | None          | No        |
| ``iterations``   | Number of iterations (i.e., repetitions) of the task                                                                               | integer or string (see [Performance models](/workload/performance-models)) | 0             | No        |
| ``synchronized`` | Whether all resources (i.e., compute nodes) synchronize before executing the task (similar to an *MPI_Barrier()* before execution) | bool                                                                       | false         | No        |

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
                "name": "Read model",
                "bytes": "model_size",
                "pattern": "root_only"
            },
            {
                "type": "cpu",
                "name": "Scatter model",
                "bytes": "model_size",
                "communication_pattern": "scatter"
            }
        ]
    },
    "on_reconfiguration": {
        "tasks": [
            {
                "type": "pfs_read",
                "name": "Read model",
                "bytes": "model_size",
                "pattern": "root_only"
            },
            {
                "type": "cpu",
                "name": "Scatter model",
                "bytes": "model_size",
                "communication_pattern": "scatter"
            }
        ]
    },
    "phases": [
        {
            "iterations": "iterations",
            "evolving_request": "num_nodes % 2 == 0 ? num_nodes + 1 : num_nodes - 1",
            "tasks": [
                {
                    "type": "cpu",
                    "name": "Compute & communicate",
                    "flops": "flops/num_nodes^alpha",
                    "computation_pattern": "uniform",
                    "bytes": "communication_size",
                    "communication_pattern": "all_to_all",
                    "coupled": true
                },
                {
                    "type": "pfs_write",
                    "name": "Checkpoint",
                    "bytes": "checkpoint_size",
                    "pattern": "all_ranks"
                }
            ]
        }
    ]
}
```
