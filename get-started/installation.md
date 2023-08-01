---
layout: default
title: Installation
parent: Get started
nav_order: 2
---

# Installation

## Required libraries

ElastiSim requires four libraries to be installed (or available):
- [ZeroMQ](https://zeromq.org/) for C++ ([cppzmq](https://github.com/zeromq/cppzmq) binding) to communicate with the scheduler process
- [JSON for Modern C++](https://json.nlohmann.me/) to read input files and wrap messages (provided with ElastiSim)
- [ExprTk](https://www.partow.net/programming/exprtk/) to evaluate performance models (provided with ElastiSim)
- [SimGrid](https://simgrid.org/) to simulate the underlying platform

Depending on the scheduling interface, an additional ZeroMQ binding is required. ElastiSim currently supports a Python interface to write scheduling algorithms and therefore requires:
- [PyZMQ](https://pyzmq.readthedocs.io/) to communicate with the simulator process

## Build (on Linux systems)

To build and install ElastiSim, install [CMake](https://cmake.org/), and execute the following commands (replacing the values in the angle brackets):

```sh
git clone https://github.com/elastisim/elastisim.git
cd elastisim/
cmake -DCMAKE_INSTALL_PREFIX=<desired_installation_path> -DSIMGRID_SOURCE_DIR=<simgrid_installation_path> -DCMAKE_BUILD_TYPE="Release" .
make -j<num_recipes>
```
