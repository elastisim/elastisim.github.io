---
layout: default
title: Python
parent: Scheduling interface
nav_order: 1
---

# Python interface

The Python scheduling interface ([elastisim-python](https://github.com/elastisim/elastisim-python) on GitHub) comes as an importable package providing a single function as a starting point to integrate a custom scheduler.

| Function signature                | Arguments                                                 | Argument types        | Description                                                                                                                     |
|-----------------------------------|-----------------------------------------------------------|-----------------------|---------------------------------------------------------------------------------------------------------------------------------|
| ``pass_algorithm(function, url)`` | Scheduling algorithm (``function``), ZeroMQ URL (``url``) | ``function``, ``str`` | Passes the scheduling algorithm to the communication interface ans establishes the ZeroMQ connection with the simulator process |

The scheduling interface invokes the passed function periodically, which must implement the required signature representing the list of jobs, nodes, and a map holding system parameters. Each invocation populates the arguments with the current state of the simulated scenario.

```python
from elastisim_python import pass_algorithm

def schedule(jobs, nodes, system):
    ...

if __name__ == '__main__':
    url = 'ipc:///tmp/elastisim.ipc'
    pass_algorithm(schedule, url)
```

## System information

## Job class

### Job type

```python
class JobType(Enum):
    RIGID
    MOLDABLE
    MALLEABLE
    EVOLVING
    ADAPTIVE
```

### Job state

```python
class JobState(Enum):
    PENDING
    RUNNING
    PENDING_RECONFIGURATION
    IN_RECONFIGURATION
    COMPLETED
    KILLED
```

### Member variables

| Variable                       | Type         | Description                                                             |
|--------------------------------|--------------|-------------------------------------------------------------------------|
| ``identifier``                 | ``int``      | see [Scheduling interface](/scheduling-interface)                       |
| ``type``                       | ``JobType``  | see [Job](/workload/job)                                                |
| ``state``                      | ``JobState`` | see [Scheduling interface](/scheduling-interface)                       |
| ``walltime``                   | ``float``    | see [Job](/workload/job)                                                |
| ``num_nodes``                  | ``int``      | see [Job](/workload/job)                                                |
| ``num_gpus_per_node``          | ``int``      | see [Job](/workload/job)                                                |
| ``num_nodes_min``              | ``int``      | see [Job](/workload/job)                                                |
| ``num_nodes_max``              | ``int``      | see [Job](/workload/job)                                                |
| ``num_gpus_per_node_min``      | ``int``      | see [Job](/workload/job)                                                |
| ``num_gpus_per_node_max``      | ``int``      | see [Job](/workload/job)                                                |
| ``submit_time``                | ``float``    | see [Job](/workload/job)                                                |
| ``start_time``                 | ``float``    | see [Scheduling interface](/scheduling-interface)                       |
| ``end_time``                   | ``float``    | see [Scheduling interface](/scheduling-interface)                       |
| ``wait_time``                  | ``float``    | see [Scheduling interface](/scheduling-interface)                       |
| ``makespan``                   | ``float``    | see [Scheduling interface](/scheduling-interface)                       |
| ``turnaround_time``            | ``float``    | see [Scheduling interface](/scheduling-interface)                       |
| ``assigned_nodes``             | ``list``     | see [Scheduling interface](/scheduling-interface)                       |
| ``assigned_node_ids``          | ``set``      | Set of node IDs assigned to the job                                     |
| ``assigned_num_gpus_per_node`` | ``int``      | see [Scheduling interface](/scheduling-interface)                       |
| ``arguments``                  | ``dict``     | see [Job](/workload/job)                                                |
| ``attributes``                 | ``dict``     | see [Job](/workload/job)                                                |
| ``total_phase_count``          | ``int``      | see [Scheduling interface](/scheduling-interface)                       |
| ``completed_phases``           | ``int``      | see [Scheduling interface](/scheduling-interface)                       |
| ``modified``                   | ``bool``     | Whether the scheduler modified the job's configuration since invocation |
| ``kill_flag``                  | ``bool``     | Whether the scheduler requested the kolling of the job since invocation |

### Member functions

| Function signature                          | Arguments                                             | Argument types       | Description                                                                   |
|---------------------------------------------|-------------------------------------------------------|----------------------|-------------------------------------------------------------------------------|
| ``assigned(nodes)``                         | Node or list of nodes (``nodes``)                     | ``list`` or ``Node`` | see [Scheduling interface](/scheduling-interface)                             |
| ``remove(nodes)``                           | Node or list of nodes (``nodes``)                     | ``list`` or ``Node`` | see [Scheduling interface](/scheduling-interface)                             |
| ``assign_num_gpus_per_node(gpus_per_node)`` | Number of GPUs per node (``gpus_per_node``)           | ``int``              | see [Scheduling interface](/scheduling-interface)                             |
| ``kill()``                                  | None                                                  | -                    | see [Scheduling interface](/scheduling-interface)                             |
| ``update_runtime_argument(key, value)``     | Key (``key``) and its corresponding value (``value``) | ``string``, Any      | Update (or initially assign) a runtime argument (valid in performance models) |

## Node class

### Node type

```python
class NodeType(Enum):
    COMPUTE_NODE
    COMPUTE_NODE_WITH_BB
    COMPUTE_NODE_WIDE_STRIPED_BB
```

### Node state

```python
class NodeState(Enum):
    FREE
    ALLOCATED
    RESERVED
```

### Member variables

| Variable             | Type          | Description                                       |
|----------------------|---------------|---------------------------------------------------|
| ``identifier``       | ``int``       | see [Scheduling interface](/scheduling-interface) |
| ``type``             | ``NodeType``  | see [Scheduling interface](/scheduling-interface) |
| ``state``            | ``NodeState`` | see [Scheduling interface](/scheduling-interface) |
| ``assigned_jobs``    | ``list``      | see [Scheduling interface](/scheduling-interface) |
| ``assigned_job_ids`` | ``set``       | Set of job IDs assigned to the node               |
| ``gpus``             | ``list``      | see [Scheduling interface](/scheduling-interface) |

### GPU class

#### GPU state

```python
class GpuState(Enum):
    FREE
    ALLOCATED
```

#### Member variables

| Variable       | Type         | Description                                       |
|----------------|--------------|---------------------------------------------------|
| ``identifier`` | ``int``      | see [Scheduling interface](/scheduling-interface) |
| ``state``      | ``GpuState`` | see [Scheduling interface](/scheduling-interface) |

## Example usage

```python
from typing import Any
from elastisim_python import JobState, JobType, NodeState, pass_algorithm, Job, Node, InvocationType


def schedule(jobs: list[Job], nodes: list[Node], system: dict[str, Any]) -> None:
    
    time = system['time']
    pending_jobs = [job for job in jobs if job.state == JobState.PENDING]
    free_nodes = [node for node in nodes if node.state == NodeState.FREE]

    if system['invocation_type'] == InvocationType.INVOKE_SCHEDULING_POINT:
        job = system['job']
        num_nodes_to_expand = min(len(free_nodes), job.num_nodes_max - len(job.assigned_nodes))
        if num_nodes_to_expand > 0:
            job.assign(free_nodes[:num_nodes_to_expand])
            del free_nodes[:num_nodes_to_expand]
    elif system['invocation_type'] == InvocationType.INVOKE_EVOLVING_REQUEST:
        job = system['job']
        evolving_request = system['evolving_request']
        num_nodes = len(job.assigned_nodes)
        diff = evolving_request - num_nodes
        if diff < 0:
            job.remove(job.assigned_nodes[diff:])
        elif len(free_nodes) >= diff:
            job.assign(free_nodes[:diff])
    else:
        for job in pending_jobs:
            if job.type == JobType.RIGID:
                if job.num_nodes <= len(free_nodes):
                    job.assign(free_nodes[:job.num_nodes])
                    del free_nodes[:job.num_nodes]
                else:
                    break
            else:
                num_nodes_to_assign = min(len(free_nodes), job.num_nodes_max)
                if num_nodes_to_assign >= job.num_nodes_min:
                    job.assign(free_nodes[:num_nodes_to_assign])
                    del free_nodes[:num_nodes_to_assign]
                else:
                    break


if __name__ == '__main__':
    url = 'ipc:///tmp/elastisim.ipc'
    pass_algorithm(schedule, url)
```
