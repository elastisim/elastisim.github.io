---
layout: default
title: Platform
nav_order: 4
---

# Platform

ElastiSim is fully compatible with SimGrid platforms (i.e., every valid SimGrid platform is a valid ElastiSim platform) and supports additional functionalities by specifying extended properties. This page introduces ElastiSim platforms with an incremental example.

{: .important }
Please refer to the SimGrid [platform documentation](https://simgrid.org/doc/latest/Platform.html) before continuing with ElastiSim examples.

## Compute nodes & interconnect

By default, ElastiSim assigns each SimGrid host the role of a compute node. We start with a simple example of a crossbar topology with 128 compute nodes, where each node has a combined (i.e., all CPU cores) processing power of 1 TFLOPS/s and a 100 Gbit/s connection to the network with a latency of 50 microseconds.

```xml
<?xml version='1.0'?>
<!DOCTYPE platform SYSTEM "https://simgrid.org/simgrid.dtd">
<platform version="4.1">
    <zone id="Darmstadt" routing="Full">
        <cluster id="Crossbar" prefix="DA_" radical="100-227" suffix=""
                 speed="1000Gf" bw="100Gbps" lat="50us"/>
    </zone>
</platform>
```

{: .note }
ElastiSim supports any SimGrid platform. Predefined topologies as well as platforms built from scratch. We continue with the crossbar topology for the sake of simplicity.

Each platform requires a particular actor running on a single node: the batch system—responsible for communicating the scheduling decisions to the corresponding nodes. Users can assign this role by setting the property ``batch_system`` to ``true`` for a single host. However, this property is optional; if not specified, ElastiSim assigns this role to the first host in the list of all hosts. Specifying this role makes no difference for the simulation but will help identify log messages that belong to the batch system. We extend the platform with a zone that contains such a host.

```xml
<?xml version='1.0'?>
<!DOCTYPE platform SYSTEM "https://simgrid.org/simgrid.dtd">
<platform version="4.1">
    <zone id="Darmstadt" routing="Full">
        <zone id="Batch-system_zone" routing="Full">
            <host id="Batch_system" speed="0Gf">
                <prop id="batch_system" value="true"/>
            </host>
        </zone>
        <cluster id="Crossbar" prefix="DA_" radical="100-227" suffix=""
                 speed="1000Gf" bw="100Gbps" lat="50us"/>
    </zone>
</platform>
```

## Storage model

ElastiSim supports two storage systems: parallel file systems (PFSs) and node-local burst buffers. In the following example, we first extend the initial platform with a PFS, and then attach a node-local burst buffer to each compute node.

### Parallel file system

PFSs comprise multiple dedicated hosts behind a single namespace. Setting the ``pfs_host`` property to ``true`` assigns the PFS host role to a single host. By default, compute nodes target and stripe the data over all available PFS hosts when executing a PFS task. We attach a single PFS host to our platform in the following example.

{: .note }
The current storage model simulates data transfer but does not consider details such as file system semantics or packet sizes.

```xml
<?xml version='1.0'?>
<!DOCTYPE platform SYSTEM "https://simgrid.org/simgrid.dtd">
<platform version="4.1">
    <zone id="Darmstadt" routing="Full">
        <zone id="Batch-system_zone" routing="Full">
            <host id="Batch_system" speed="0Gf">
                <prop id="batch_system" value="true"/>
            </host>
        </zone>
        <cluster id="Crossbar" prefix="DA_" radical="100-227" suffix=""
                 speed="1000Gf" bw="100Gbps" lat="50us"/>
        <zone id="PFS_zone" routing="Full">
            <host id="PFS" speed="0Gf">
                <prop id="pfs_host" value="true"/>
            </host>
        </zone>
        <link id="PFS_read" bandwidth="80GBps" latency="50us"/>
        <link id="PFS_write" bandwidth="50GBps" latency="50us"/>
        <zoneRoute src="PFS_zone" dst="Crossbar" gw_src="PFS"
                   gw_dst="DA_Crossbar_router" symmetrical="NO">
            <link_ctn id="PFS_read"/>
        </zoneRoute>
        <zoneRoute src="Crossbar" dst="PFS_zone" gw_src="DA_Crossbar_router"
                   gw_dst="PFS" symmetrical="NO">
            <link_ctn id="PFS_write"/>
        </zoneRoute>
    </zone>
</platform>
```

We extended the platform with the new zone ``PFS_zone`` containing the host ``PFS``. Creating an asymmetrical zone route between the crossbar topology (which implicitly is a zone) and the new PFS zone, we established a connection between the crossbar router and the PFS. The asymmetric route consists of two links, defining an independent read and write bandwidth.

Although compute nodes stripe the data when targeting the PFS, users can assign an affinity to specific PFS hosts by passing a semicolon-separated string to the property ``pfs_targets`` and associating a compute node only to target the specified PFS hosts. The following example describes a compute node with an affinity for the PFS hosts ``PFS1``, ``PFS2``, and ``PFS3``.

```xml
<host id="Host" speed="500Gf">
    <prop id="pfs_targets" value="PFS1;PFS2;PFS3"/>
</host>
```

### Node-local burst buffer

Compute nodes support local storage systems buffering peaks during high I/O loads. We attach a node-local burst buffer with a read bandwidth of 4 GB/s and a write bandwidth of 2 GB/s to each host in the platform. Setting the property ``node_local_bb`` to ``true`` and passing the bandwidths via ``bb_read_bw`` and ``bb_write_bw`` is sufficient to attach a node-local burst buffer to a compute node.

{: .note }
Properties defined in ``<cluster>`` environments apply to all hosts.

```xml
<?xml version='1.0'?>
<!DOCTYPE platform SYSTEM "https://simgrid.org/simgrid.dtd">
<platform version="4.1">
    <zone id="Darmstadt" routing="Full">
        <zone id="Batch-system_zone" routing="Full">
            <host id="Batch_system" speed="0Gf">
                <prop id="batch_system" value="true"/>
            </host>
        </zone>
        <cluster id="Crossbar" prefix="DA_" radical="100-227" suffix=""
                 speed="1000Gf" bw="100Gbps" lat="50us">
            <prop id="node_local_bb" value="true"/>
            <prop id="bb_read_bw" value="4GBps"/>
            <prop id="bb_write_bw" value="2GBps"/>
        </cluster>
        <zone id="PFS_zone" routing="Full">
            <host id="PFS" speed="0Gf">
                <prop id="pfs_host" value="true"/>
            </host>
        </zone>
        <link id="PFS_read" bandwidth="80GBps" latency="50us"/>
        <link id="PFS_write" bandwidth="50GBps" latency="50us"/>
        <zoneRoute src="PFS_zone" dst="Crossbar" gw_src="PFS"
                   gw_dst="DA_Crossbar_router" symmetrical="NO">
            <link_ctn id="PFS_read"/>
        </zoneRoute>
        <zoneRoute src="Crossbar" dst="PFS_zone" gw_src="DA_Crossbar_router"
                   gw_dst="PFS" symmetrical="NO">
            <link_ctn id="PFS_write"/>
        </zoneRoute>
    </zone>
</platform>
```

Modern file systems for node-local burst buffers such as BeeOND or GekkoFS support wide-striped data access, accumulating the storage capacity of all burst buffers behind a single namespace. Reading from and writing to such file systems occur distributedly. ElastiSim supports wide striping and automatically introduces network communication for wide-striped file systems, which the user can define by setting the property ``wide_striping`` to ``true``. As wide striping can introduce a computational overhead, users can specify how many FLOPS the compute has to process per transferred byte by setting the ``flops_per_byte`` property.


```xml
<host id="Host" speed="500Gf">
    <prop id="node_local_bb" value="true"/>
    <prop id="bb_read_bw" value="4GBps"/>
    <prop id="bb_write_bw" value="2GBps"/>
    <prop id="wide_striping" value="true"/>
</host>
```

## GPU model

Users can extend compute nodes with multiple GPUs by setting the ``num_gpus`` property to any integer number and defining the processing power of each GPU by specifying ``flops_per_gpu``. In the case of multiple GPUs on a node, ElastiSim creates a fully connected GPU topology and expects the property ``gpu_to_gpu_bandwidth``, defining the bandwidth for each possible GPU pair on a node. We extend the previous platform by four GPUs per node with a processing power of 20 TFLOPS/s per GPU and connect each GPU pair with 50 GByte/s.

```xml
<?xml version='1.0'?>
<!DOCTYPE platform SYSTEM "https://simgrid.org/simgrid.dtd">
<platform version="4.1">
    <zone id="Darmstadt" routing="Full">
        <zone id="Batch-system_zone" routing="Full">
            <host id="Batch_system" speed="0Gf">
                <prop id="batch_system" value="true"/>
            </host>
        </zone>
        <cluster id="Crossbar" prefix="DA_" radical="100-227" suffix=""
                 speed="1000Gf" bw="100Gbps" lat="50us">
            <prop id="node_local_bb" value="true"/>
            <prop id="bb_read_bw" value="4GBps"/>
            <prop id="bb_write_bw" value="2GBps"/>
            <prop id="num_gpus" value="4"/>
            <prop id="flops_per_gpu" value="20Tf"/>
            <prop id="gpu_to_gpu_bw" value="50GBps"/>
        </cluster>
        <zone id="PFS_zone" routing="Full">
            <host id="PFS" speed="0Gf">
                <prop id="pfs_host" value="true"/>
            </host>
        </zone>
        <link id="PFS_read" bandwidth="80GBps" latency="50us"/>
        <link id="PFS_write" bandwidth="50GBps" latency="50us"/>
        <zoneRoute src="PFS_zone" dst="Crossbar" gw_src="PFS"
                   gw_dst="DA_Crossbar_router" symmetrical="NO">
            <link_ctn id="PFS_read"/>
        </zoneRoute>
        <zoneRoute src="Crossbar" dst="PFS_zone" gw_src="DA_Crossbar_router"
                   gw_dst="PFS" symmetrical="NO">
            <link_ctn id="PFS_write"/>
        </zoneRoute>
    </zone>
</platform>
```

## Summary

The following table summarizes all ElastiSim-specific host properties:

| Property           | Default       | Mandatory                             |
|--------------------|---------------|---------------------------------------|
| ``batch_system``   | ``false``     | No                                    |
| ``pfs_host``       | ``false``     | No                                    |
| ``pfs_targets``    | All PFS hosts | No                                    |
| ``node_local_bb``  | ``false``     | No                                    |
| ``bb_read_bw``     | -             | Yes, if ``node_local_bb`` is ``true`` |
| ``bb_write_bw``    | -             | Yes, if ``node_local_bb`` is ``true`` |
| ``wide_striping``  | ``false``     | No                                    |
| ``flops_per_byte`` | 0             | No                                    |
| ``num_gpus``       | 0             | No                                    |
| ``flops_per_gpu``  | -             | Yes, if ``num_gpus`` > 0              |
| ``gpu_to_gpu_bw``  | -             | Yes, if ``num_gpus`` > 1              |
