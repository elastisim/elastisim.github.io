---
layout: default
title: Quickstart
parent: Get started
nav_order: 1
---

# Quickstart

The easiest way to get started with ElastiSim is by cloning the example project available on [GitHub](https://github.com/elastisim/example-project). This scenario simulates an FCFS (first come, first serve) scheduling algorithm applied on 32 rigid jobs with alternating compute and I/O phases running on a crossbar topology with 128 compute nodes. The following steps will create two Docker containers and start the simulation.

## Installation

To build the container required to run ElastiSim, install Docker and execute the following command:
```sh
docker build -t elastisim .
```

## Simulation

To run the simulation, execute the following commands in two different sessions:

### \*nix:
```sh
docker run -v $PWD/data:/data -v $PWD/algorithm:/algorithm -u `id -u $USER` --name elastisim -it --rm elastisim /data/input/configuration.json --log=root.thresh:warning
docker exec -u `id -u $USER` -it elastisim python3 /algorithm/algorithm.py
```

### Mac OS:
```sh
docker run -v $PWD/data:/data -v $PWD/algorithm:/algorithm --name elastisim -it --rm elastisim /data/input/configuration.json --log=root.thresh:warning
docker exec -it elastisim python3 /algorithm/algorithm.py
```

### Windows (PowerShell):
```sh
docker run -v ${PWD}\data:/data -v {PWD}\algorithm:/algorithm --name elastisim -it --rm elastisim /data/input/configuration.json --log=root.thresh:warning
docker exec -it elastisim python3 /algorithm/algorithm.py
```

The first command runs the ElastiSim simulator process and accepts two inputs:
- the configuration file (JSON)
- the logging level

For a more detailed output change `--log=root.thresh:warning` to `--log=root.thresh:info` (caution: verbose)

The second command runs the scheduling algorithm.
