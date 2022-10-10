---
layout: default
title: Performance models
parent: Workload
nav_order: 4
---

# Performance models

Performance models (i.e., human-readable mathematical functions) are a crucial feature of ElastiSim, enabling malleable (and moldable) workloads. All tasks (see [Task types](/workload/task-types)) implicitly support performance models to specify either the load simulated on the platform or the number of iterations.

The simulation engine evaluates performance models on each (re)configuration to a single number. In combination with variables representing the number of assigned resources, performance models are a powerful feature to describe adaptive workloads. ElastiSim supports the following variables in performance models for tasks:

| Variable              | Description                                                                               |
|-----------------------|-------------------------------------------------------------------------------------------|
| ``num_nodes``         | The number of assigned compute nodes                                                      |
| ``num_gpus_per_node`` | The number of assigned GPUs per compute node                                              |
| ``num_gpus``          | The total number of assigned GPUs (syntactic sugar for ``num_nodes * num_gpus_per_node``) |

## Job arguments

Arguments specified for a job (see [Job](/workload/job)) are automatically valid variables in all performance models. As multiple jobs can use the same application model, arguments can enable different workloads without modeling a new application. Furthermore, arguments also allow phases to use performance models in their number of iterations.

{: .note }
Phases do not support variables representing the number of assigned resources in performance models, as this would break malleability based on scheduling points between phases (and phase iterations).

## Example

The following example shows two jobs specifying different arguments to adjust the simulated load. The computational load per node (note the ``uniform`` distribution pattern) in the ``cpu`` task depends on the actively configured number of nodes.

### Jobs

```json
{
  "jobs": [
    {
      "type": "malleable",
      "submit_time": 120,
      "num_nodes_min": 16,
      "num_nodes_max": 32,
      "application_model": "/path/to/application_model.json",
      "arguments": {
        "phase_i": 10,
        "seq_i": 25,
        "comp": 8e12,
        "checkpoint_size": 7e11
      }
    },
    {
      "type": "malleable",
      "submit_time": 360,
      "num_nodes_min": 12,
      "num_nodes_max": 24,
      "application_model": "/path/to/application_model.json",
      "arguments": {
        "phase_i": 15,
        "seq_i": 40,
        "comp": 6e12,
        "checkpoint_size": 9e11
      }
    }
  ]
}
```

### Application model

```json
{
  "phases": [
    {
      "iterations": "phase_i",
      "tasks": [
        {
          "type": "pfs_read",
          "name": "PFS write",
          "bytes": "model_size",
          "pattern": "uniform"
        },
        {
          "type": "sequence",
          "iterations": "seq_i",
          "tasks": [
            {
              "type": "cpu",
              "flops": "comp/num_nodes^0.8",
              "computation_pattern": "uniform"
            },
            {
              "type": "pfs_write",
              "name": "PFS write",
              "bytes": "checkpoint_size",
              "pattern": "uniform"
            }
          ]
        }
      ]
    }
  ]
}
```
