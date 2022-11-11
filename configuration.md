---
layout: default
title: Configuration
nav_order: 3
---

# Configuration

The configuration file is the only argument passed to ElastiSim and defines the parameters determining the conditions of the simulation. Following the JSON file format, users define configuration parameters by their keys and values. The following table lists all available configuration options.

| Key                           | Description                                                                     | Value type        | Default value           | Mandatory                   |
|-------------------------------|---------------------------------------------------------------------------------|-------------------|-------------------------|-----------------------------|
| ``scheduling_interval``       | Invocation interval of the scheduling algorithm                                 | integer (seconds) | -                       | Yes                         |
| ``min_scheduling_interval``   | Minimum time between two scheduling algorithm invocations                       | integer (seconds) | ``scheduling_interval`` | No                          |
| ``schedule_on_job_submit``    | Whether a job submission triggers a scheduling algorithm invocation             | bool              | false                   | No                          |
| ``schedule_on_job_finalize``  | Whether a job finalization triggers a scheduling algorithm invocation           | bool              | false                   | No                          |
| ``allow_oversubscription``    | Whether the scheduler can oversubscribe compute nodes with multiple jobs        | bool              | false                   | No                          |
| ``zmq_url``                   | URL to establish the connection between the simulator and the scheduler process | string            | -                       | Yes                         |
| ``sensing``                   | Whether the monitoring module is active                                         | bool              | false                   | No                          |
| ``sensing_interval``          | Interval of the monitoring module to sense platform utilization parameters      | integer (seconds) | -                       | Yes, if ``sensing`` is true |
| ``pfs_read_links``            | PFS read links to sense by the monitoring module                                | array of strings  | empty array             | No                          |
| ``pfs_write_links``           | PFS write links to sense by the monitoring module                               | array of strings  | empty array             | No                          |
| ``jobs_file``                 | Path to the jobs file                                                           | string            | -                       | Yes                         |
| ``platform_file``             | Path to the platform file                                                       | string            | -                       | Yes                         |
| ``job_statistics``            | Output path to write the job statistics file                                    | string            | -                       | Yes                         |
| ``node_utilization``          | Output path to write the compute node utilization file                          | string            | -                       | Yes                         |
| ``cpu_utilization``           | Output path to write the CPU utilization file                                   | string            | -                       | Yes, if ``sensing`` is true |
| ``network_activity``          | Output path to write the network activity file                                  | string            | -                       | Yes, if ``sensing`` is true |
| ``pfs_utilization``           | Output path to write the PFS utilization file                                   | string            | -                       | Yes, if ``sensing`` is true |
| ``gpu_utilization``           | Output path to write the GPU utilization file                                   | string            | -                       | Yes, if ``sensing`` is true |

{: .warning }
Setting the sensing interval too small can significantly increase simulation times, as the discrete-event simulation engine will fire an event at each sensing interval.

## Example configuration

```json
{
  "scheduling_interval": 60,
  "min_scheduling_interval": 30,
  "schedule_on_job_submit": false,
  "allow_oversubscription": false,
  "sensing": true,
  "sensing_interval": 5,
  "zmq_url": "ipc:///tmp/elastisim.ipc",
  "pfs_read_links": ["PFS_read"],
  "pfs_write_links": ["PFS_write"],
  "jobs_file": "/path/to/jobs.json",
  "platform_file": "/path/to/platform.xml",
  "job_statistics": "/path/to/job_statistics.csv",
  "cpu_utilization": "/path/to/cpu_utilization.csv",
  "node_utilization": "/path/to/node_utilization.csv",
  "network_activity": "/path/to/network_activity.csv",
  "pfs_utilization": "/path/to/pfs_utilization.csv",
  "gpu_utilization": "/path/to/gpu_utilization.csv"
}
```
