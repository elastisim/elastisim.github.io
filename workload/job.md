---
layout: default
title: Job
parent: Workload
nav_order: 1
---

# Job
Jobs in ElastiSim have a designated type that defines (1) whether the job supports reconfigurations (i.e., resource modifications during runtime) and (2) the responsible entity initiating the request. Reconfiguration requests are either initiated by the scheduler or the job itself. While jobs must accept reconfiguration requests initiated by the scheduler, the scheduler can decline those initiated by jobs (so-called evolving requests).

In addition to rigid, moldable, malleable, and evolving jobs, ElastiSim supports a fifth job type, introduced as _adaptive_ jobs, combining the features of malleable and evolving jobs. The following table describes the job types with their corresponding characteristics:

| Type      | Number of resources | Reconfigurable during runtime | Accepts reconfiguration requests | Can request reconfigurations |
|-----------|---------------------|-------------------------------|----------------------------------|------------------------------|
| Rigid     | Fixed               | No                            | -                                | -                            |
| Moldable  | Variable            | No                            | -                                | -                            |
| Malleable | Variable            | Yes                           | Yes                              | No                           |
| Evolving  | Variable            | Yes                           | No                               | Yes                          |
| Adaptive  | Variable            | Yes                           | Yes                              | Yes                          |


Each job holds properties that the scheduling algorithm can use to make scheduling decisions. Users can specify the following properties using the JSON format:

| Property              | Description                                                                                | Value type      | Default value | Mandatory |
|-----------------------|--------------------------------------------------------------------------------------------|-----------------|---------------|-----------|
| ``jobs``              | Array of all jobs (top-level structure)                                                    | array           | -             | Yes       |
| ``type``              | Job type (``"rigid"``, ``"moldable"``, ``"malleable"``, ``"evolving"``, or ``"adaptive"``) | string          | -             | Yes       |
| ``submit_time``       | Submission time of the job (absolute value)                                                | float (seconds) | -             | Yes       |
| ``application_model`` | Application model of the job (path to file)                                                | string          | -             | Yes       |
| ``walltime``          | Time limit of a job before it is killed (0 for no limit)                                   | float (seconds) | 0             | No        |
| ``arguments``         | Custom arguments (i.e., variables) used in performance models                              | map             | empty map     | No        |
| ``attributes``        | Custom attributes forwarded to the scheduler (e.g., priority)                              | map             | empty map     | No        |

![A figure visualizing the different classifications of a job](/assets/images/Job_classification.svg "Job classifications"){: .d-block .mx-auto width="100%" }

As rigid jobs have a fixed number of resources, they require the number of nodes or GPUs, respectively:

| Property              | Description                       | Value type | Default value | Mandatory |
|-----------------------|-----------------------------------|------------|---------------|-----------|
| ``num_nodes``         | Requested number of nodes         | integer    | -             | Yes       |
| ``num_gpus_per_node`` | Requested number of GPUs per node | integer    | 0             | No        |

{: .important }
Jobs make requests based on compute nodes, not CPUs or CPU cores, respectively.

Moldable, malleable, evolving and adaptive jobs have a variable number of resources, specified using the following properties:

| Property                  | Description                                 | Value type | Default value | Mandatory |
|---------------------------|---------------------------------------------|------------|---------------|-----------|
| ``num_nodes_min``         | Requested number of nodes (minimum)         | integer    | -             | Yes       |
| ``num_nodes_max``         | Requested number of nodes (maximum)         | integer    | -             | Yes       |
| ``num_gpus_per_node_min`` | Requested number of GPUs per node (minimum) | integer    | 0             | No        |
| ``num_gpus_per_node_max`` | Requested number of GPUs per node (maximum) | integer    | 0             | No        |

{: .note }
ElastiSim allows minimum and maximum values to be the same (e.g., malleable jobs requesting a fixed number of GPUs per node).

## Example job file

```json
{
  "jobs": [
    {
      "type": "rigid",
      "submit_time": 0,
      "num_nodes": 12,
      "num_gpus_per_node": 4,
      "application_model": "/path/to/application_model_1.json",
      "arguments": {
        "x": 20,
        "y": 60.8
      },
      "attributes": {
        "priority": 50
      }
    },
    {
      "type": "rigid",
      "submit_time": 80,
      "num_nodes": 12,
      "application_model": "/path/to/application_model_2.json",
      "arguments": {
        "alpha": 15
      },
      "attributes": {
        "priority": 0
      }
    },
    {
      "type": "evolving",
      "submit_time": 120,
      "num_nodes_min": 8,
      "num_nodes_max": 24,
      "application_model": "/path/to/application_model_3.json",
      "attributes": {
        "priority": 40
      }
    },
    {
      "type": "malleable",
      "submit_time": 360,
      "num_nodes_min": 12,
      "num_nodes_max": 36,
      "application_model": "/path/to/application_model_3.json",
      "arguments": {
        "beta": 80.4
      },
      "attributes": {
        "name": "special_job",
        "priority": 110
      }
    },
    {
      "type": "adaptive",
      "submit_time": 480,
      "num_nodes_min": 10,
      "num_nodes_max": 20,
      "num_gpus_per_node_min": 2,
      "num_gpus_per_node_max": 4,
      "application_model": "/path/to/application_model_4.json",
      "arguments": {
        "gamma": 14.7
      },
      "attributes": {
        "name": "special_job_2",
        "dependency": "special_job",
        "priority": 30
      }
    },
    {
      "type": "moldable",
      "submit_time": 420,
      "num_nodes_min": 16,
      "num_nodes_max": 32,
      "num_gpus_per_node_min": 4,
      "num_gpus_per_node_max": 4,
      "application_model": "/path/to/application_model_5.json",
      "attributes": {
        "priority": 140
      }
    }
  ]
}
```
