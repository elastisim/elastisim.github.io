---
layout: default
title: Configuration
nav_order: 3
---

# Configuration

The configuration file is the only argument passed to ElastiSim and defines the parameters determining the conditions of the simulation. Following the JSON file format, users define configuration parameters by their keys and values. The following table lists all available configuration options.

| Key                              | Description                                                                                                                          | Value type        | Default value | Mandatory                                                                   |
|----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|-------------------|---------------|-----------------------------------------------------------------------------|
| ``jobs_file``                    | Path to the jobs file                                                                                                                | string            | -             | Yes                                                                         |
| ``platform_file``                | Path to the platform file                                                                                                            | string            | -             | Yes                                                                         |
| ``zmq_url``                      | URL to establish the connection between the simulator and the scheduler process                                                      | string            | -             | Yes                                                                         |
| ``schedule_on_job_submit``       | Whether a job submission triggers a scheduling algorithm invocation                                                                  | bool              | false         | Yes, if ``scheduling_interval`` is false                                    |
| ``schedule_on_job_finalize``     | Whether a job finalization triggers a scheduling algorithm invocation                                                                | bool              | false         | Yes, if ``scheduling_interval`` is false                                    |
| ``schedule_on_scheduling_point`` | Whether a job reaching a scheduling point triggers a scheduling algorithm invocation                                                 | bool              | false         | No                                                                          |
| ``schedule_on_reconfiguration``  | Whether a job reconfiguration triggers a scheduling algorithm invocation                                                             | bool              | false         | No                                                                          |
| ``scheduling_interval``          | Invocation interval of the scheduling algorithm                                                                                      | integer (seconds) | 0 (disabled)  | Yes, if ``schedule_on_job_submit`` or ``schedule_on_job_finalize`` is false |
| ``min_scheduling_interval``      | Minimum time between two scheduling algorithm invocations                                                                            | integer (seconds) | 0 (disabled)  | No                                                                          |
| ``allow_oversubscription``       | Whether the scheduler can oversubscribe compute nodes with multiple jobs                                                             | bool              | false         | No                                                                          |
| ``clip_evolving_requests``       | Whether evolving requests are clipped to stay in the possible range of configurations (i.e., [``num_nodes_min``, ``num_nodes_max``]) | bool              | true          | No                                                                          |
| ``forward_io_information``       | Whether the scheduler receives I/O information (PFS read/write bandwidth and utilization)                                            | bool              | false         | No                                                                          |
| ``job_kill_grace_period``        | Time to wait to kill the job after exceeding its walltime                                                                            | integer (seconds) | 0             | No                                                                          |
| ``show_progress_bar``            | Whether the progress bar is shown (only shown when log level is higher than ``info``)                                                | bool              | true          | No                                                                          |
| ``sensing``                      | Whether the monitoring module is active                                                                                              | bool              | false         | No                                                                          |
| ``sensing_interval``             | Interval of the monitoring module to sense platform utilization parameters                                                           | integer (seconds) | -             | Yes, if ``sensing`` is true                                                 |
| ``log_task_times``               | Whether task time logging is active                                                                                                  | bool              | false         | No                                                                          |
| ``pfs_read_links``               | PFS read links to sense by the monitoring module                                                                                     | array of strings  | empty array   | No                                                                          |
| ``pfs_write_links``              | PFS write links to sense by the monitoring module                                                                                    | array of strings  | empty array   | No                                                                          |
| ``job_statistics``               | Output path to write the job statistics file                                                                                         | string            | -             | Yes                                                                         |
| ``node_utilization``             | Output path to write the compute node utilization file                                                                               | string            | -             | Yes                                                                         |
| ``cpu_utilization``              | Output path to write the CPU utilization file                                                                                        | string            | -             | Yes, if ``sensing`` is true                                                 |
| ``network_activity``             | Output path to write the network activity file                                                                                       | string            | -             | Yes, if ``sensing`` is true                                                 |
| ``pfs_utilization``              | Output path to write the PFS utilization file                                                                                        | string            | -             | Yes, if ``sensing`` is true                                                 |
| ``gpu_utilization``              | Output path to write the GPU utilization file                                                                                        | string            | -             | Yes, if ``sensing`` is true                                                 |
| ``task_times``                   | Output path to write the task times file                                                                                             | string            | -             | Yes, if ``log_task_times`` is true                                          |

{: .warning }
Setting the sensing interval too small can significantly increase simulation times, as the discrete-event simulation engine will fire an event at each sensing interval. Logging task times can also introduce a significant overhead.

{: .note }
ElastiSim supports node migration (i.e., transferring nodes from one job to another) in a single scheduling step when the decision is taken at an invocation triggered by a scheduling point (requires ``schedule_on_scheduling_point`` to be ``true``) or evolving request.

{: .important }
``schedule_on_reconfiguration`` triggers the scheduling algorithm _after_ applying a **pending** resource reconfiguration but _before_ executing a potential ``on_reconfiguration`` or ``on_expansion`` phase. This invocation trigger enables scheduling decisions when resources change their state, such as nodes becoming free after a shrink operation when a job reaches its next scheduling point. However, a job reconfigured during a scheduling point or evolving request will not trigger the algorithm again when the reconfiguration is applied.

## Example configuration

```json
{
  "jobs_file": "/path/to/jobs.json",
  "platform_file": "/path/to/platform.xml",
  "zmq_url": "ipc:///tmp/elastisim.ipc",
  "schedule_on_job_submit": true,
  "schedule_on_job_finalize": true,
  "schedule_on_scheduling_point": true,
  "scheduling_interval": 0,
  "min_scheduling_interval": 0,
  "allow_oversubscription": false,
  "clip_evolving_requests": true,
  "forward_io_information": true,
  "sensing": true,
  "sensing_interval": 1,
  "log_task_times": true,
  "pfs_read_links": ["PFS_read"],
  "pfs_write_links": ["PFS_write"],
  "job_statistics": "/path/to/job_statistics.csv",
  "cpu_utilization": "/path/to/cpu_utilization.csv",
  "node_utilization": "/path/to/node_utilization.csv",
  "network_activity": "/path/to/network_activity.csv",
  "pfs_utilization": "/path/to/pfs_utilization.csv",
  "gpu_utilization": "/path/to/gpu_utilization.csv",
  "task_times": "/path/to/task_times.csv"
}
```
