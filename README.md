# Docker for HPC

This repository collects Dockerized versions of the most widespread HPC stacks. In particular, each folder contains:

- A set of `Dockerfiles` that can be used standalone or manually assembled in a microservices application;
- A `docker-compose.yml` file that simulates an HPC cluster orchestrated by the selected stack.

This repository's main purpose is to provide the HPC community with Docker-based simulated environments of HPC stacks, which will be used mainly for small-scale experiments, debugging, and CI/CD pipelines.

All Docker containers are published on DockerHub under the `alphaunito` organization. At the moment, this repository contains Dockerized versions of:

- [Slurm](./slurm/README.md)

All images support the explicit definition of opencontainers [annotations](https://github.com/opencontainers/image-spec/blob/main/annotations.md) through two build args:

- The `COMMIT` build arg should contain the digest of the commit the image is being created from. It populates the `org.opencontainers.image.revision` label
- The `CREATION_DATE` build arg should contain the creation time of the image in `RFC 3339` format. It populates the `org.opencontainers.image.created` label
