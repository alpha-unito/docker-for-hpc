FROM ubuntu:jammy

ARG COMMIT="none"
ARG CREATION_DATE="none"

LABEL org.opencontainers.image.authors="Iacopo Colonnelli <iacopo.colonnelli@unito.it>"
LABEL org.opencontainers.image.base.name="docker.io/ubuntu:jammy"
LABEL org.opencontainers.image.created="${CREATION_DATE}"
LABEL org.opencontainers.image.licenses="OpenSSL"
LABEL org.opencontainers.image.revision="${COMMIT}"
LABEL org.opencontainers.image.ref.name="alphaunito/slurmctld"
LABEL org.opencontainers.image.source="https://github.com/alpha-unito/docker-for-hpc"
LABEL org.opencontainers.image.title="Slurm management daemon"
LABEL org.opencontainers.image.version="21.08.5"

RUN apt update                                                                             \
 && apt install -y --no-install-recommends                                                 \
        ca-certificates                                                                    \
        hwloc                                                                              \
        mailutils                                                                          \
        munge                                                                              \
        openmpi-bin                                                                        \
        openssh-server                                                                     \
        slurm-client                                                                       \
        slurmctld                                                                          \
        supervisor                                                                         \
        wget                                                                               \
 && rm -rf /var/lib/apt/lists/*                                                            \
 && wget -O /bin/gosu https://github.com/tianon/gosu/releases/download/1.17/gosu-amd64     \
 && chmod +x /bin/gosu                                                                     \
 && gosu nobody true                                                                       \
 && mkdir -p                                                                               \
        /run/munge                                                                         \
        /run/sshd                                                                          \
        /var/log/munge                                                                     \
        /var/log/slurm                                                                     \
        /var/run/slurm                                                                     \
        /var/spool/slurmctld                                                               \
 && chown munge:munge                                                                      \
        /run/munge                                                                         \
        /var/log/munge                                                                     \
 && chown slurm:slurm                                                                      \
        /var/log/slurm                                                                     \
        /var/run/slurm                                                                     \
        /var/spool/slurmctld                                                               \
 && adduser                                                                                \
        --disabled-password                                                                \
        --gecos ""                                                                         \
        hpcuser

COPY config/cgroups.conf                                                                   \
     config/slurm.conf                                                                     \
     /etc/slurm/

COPY config/supervisord.conf                                                               \
     /etc/supervisor/conf.d/

COPY scripts/healthcheck                                                                   \
     scripts/run-slurmctld                                                                 \
     /bin/

HEALTHCHECK --start-period=20s CMD healthcheck

EXPOSE 22 6817

WORKDIR /home/hpcuser

ENTRYPOINT supervisord