# OpenPBS on Docker

This folder contains a fully Dockerized version of the [OpenPBS](https://www.openpbs.org/) queue manager. The version of OpenPBS shipped is the `23.06.06`, which runs on Rocky Linux v8.8.

This repository contains the source code of different container images:

- `alphaunito/openpbs-server:23.06.06`, which runs the OpenPBS control plane
- `alphaunito/openpbs-execution:23.06.06`, which runs a OpenPBS compute node

Plus, it also contains a [docker-compose.yml](./docker-compose.yml) file that can deplyo an entire OpenPBS cluster with a single controller and a set of compute nodes. All these components are detailed below

## OpenPBS Server

The OpenPBS control plane is composed of three main processes:

- The `pbs_server` process is the OpenPBS batch server;
- The `pbs_sched` program queries the server about the state of PBS, communicates with `pbs_mom` nodes to get information, and decides as what jobs to run;
- The `pbs_comm` daemon handles communication between the `pbs_server`, `pbs_sched`, and the `pbs_mom` daemons on execution hosts.

The `openpbs-server` Docker image can be build and published on DockerHub using the following commands

```bash
docker build -t alphaunito/openpbs-server:23.06.06 server
docker push alphaunito/openpbs-server:23.06.06
```

To correctly register the execution nodes, an `openpbs-server` container needs 2 environment variables:

- The `PBS_EXECUTION_NODES` variable must contain the number of compute nodes that OpenPBS should manage. If this variable is not set, the container displays an error message and terminates
- The `PBS_EXECUTION_HOST_NAME_PREFIX` variable should contain the prefix of the hostname used to identify compute nodes. If this variable is not set, the container displays an error message and terminates

Note that all the compute nodes in the simulated HPC cluster should have a reachable hostname equal to `"${PBS_EXECUTION_NODES}${X}"`, where `X` is an integer in the range `[1, ${PBS_EXECUTION_NODES}]`

## OpenPBS Execution

The `pbs_mom` process is the compute node daemon for OpenPBS. It places jobs into execution as directed by the server, establishes resource usage limits, monitors the job's usage, and notifies the server when the job completes. The `openpbs-execution` Docker image can be build and published using the following commands

```bash
docker build -t alphaunito/openpbs-execution:23.06.06 execution
docker push alphaunito/openpbs-execution:23.06.06
```

To correctly connect to an `openpbs-server` node, an `openpbs-execution` container needs a `PBS_SERVER_HOST_NAME` variable that should contain the hostname of the target `openpbs-server` container. If this variable is not set, the container displays an error message and terminates

## Cluster

The `openpbs-server` and `openpbs-execution` images described above can be used to set up a Docker-based OpenPBS cluster with a single controller and many compute nodes. This task can be achieved either manually or starting from the [docker-compose.yml](./docker-compose.yml) file contained in this repository

Note that the `openpbs-server` node should have an identifiable hostname, as compute nodes must register with the control plane to be addressable. In Docker Compose, an explicit hostname can be set for a given service using the `hostname` keyword.

To allow for unprivileged workloads, an `hpcuser` has been configured inside the images. Commands can be executed by explicitly impersonating this user, through the `--user hpcuser` flag. For example

```bash
docker exec -it --user hpcuser openpbs-server bash
```

In order to simulate an HPC facility, where the home folder is commonly mounted on a shared parallel file system, users may want to mount the `/home/hpcuser` folder as a shared volume among all the containers in the OpenPBS cluster.
