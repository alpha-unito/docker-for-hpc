# Slurm on Docker

This folder contains a fully Dockerized version of the [Slurm](https://slurm.schedmd.com/) queue manager. The version of SLURM shipped is the one downloaded from the apt repository, which is currently `23.11.4`

This repository contains the source code of different container images:

- `alphaunito/slurmctld:23.11.4`, which runs the Slurm control plane
- `alphaunito/slurmd:23.11.4`, which runs a Slurm compute node

Plus, it also contains a [docker-compose.yml](./docker-compose.yml) file that can deplyo an entire Slurm cluster with a single controller and a set of compute nodes. All these components are detailed below

## Slurmctld

The `slurmctld` process is the central management daemon of Slurm. It constitutes the control plane of the Slurm queue manager. The `slurmctld` Docker image can be build and published on DockerHub using the following commands

```bash
docker build -t alphaunito/slurmctld:23.11.4 slurmctld
docker push alphaunito/slurmctld:23.11.4
```

To correctly populate the `slurm.conf` file, a `slurmctld` container needs 2 environment variables:

- The `SLURMD_NODES` variable must contain the number of compute nodes that Slurm should manage. If this variable is not set, the container displays an error message and terminates
- The `SLURMD_HOSTNAME_PREFIX` variable should contain the prefix of the hostname used to identify compute nodes. If this variable is not set, the container displays an error message and terminates

Note that all the compute nodes in the simulated HPC cluster should have a reachable hostname equal to `"${SLURMD_HOSTNAME_PREFIX}${X}"`, where `X` is an integer in the range `[1, ${SLURMD_NODES}]`

## Compute

The `slurmd` process is the compute node daemon for Slurm. It monitors all tasks running on the compute node , accepts work (tasks), launches tasks, and kills running tasks upon request. The `slurmd` Docker image can be build and published using the following commands

```bash
docker build -t alphaunito/slurmd:23.11.4 slurmd
docker push alphaunito/slurmd:23.11.4
```

To correctly connect to a `slurmctld` node, a `slurmd` container needs a `SLURMCTLD_HOSTNAME` variable that should contain the hostname of the target `slurmctld` container. If this variable is not set, the container displays an error message and terminates

## Cluster

The `slurmctld` and `slurmd` images described above can be used to set up a Docker-based Slurm cluster with a single controller and many compute nodes. This task can be achieved either manually or starting from the [docker-compose.yml](./docker-compose.yml) file contained in this repository

Note that the `slurmctld` node should have an identifiable hostname, as compute nodes must register with the control plane to be addressable. In Docker Compose, an explicit hostname can be set for a given service using the `hostname` keyword.

Slurm relies on [MUNGE](https://dun.github.io/munge/) for authentication. To setup a munge cluster, all the involved nodes should share a common key located in the the `/etc/munge` fodler. Given that, the `/etc/munge` folder should be mounted as a shared volume among all the containers in the Slurm cluster.

To allow for unprivileged workloads, an `hpcuser` has been configured inside the images. Commands can be executed by explicitly impersonating this user, through the `--user hpcuser` flag. For example

```bash
docker exec -it --user hpcuser slurmctld bash
```

In order to simulate an HPC facility, where the home folder is commonly mounted on a shared parallel file system, users may want to mount the `/home/hpcuser` folder as a shared volume among all the containers in the Slurm cluster.
