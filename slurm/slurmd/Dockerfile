FROM ubuntu:noble

ARG COMMIT="none"
ARG CREATION_DATE="none"

LABEL org.opencontainers.image.authors="Iacopo Colonnelli <iacopo.colonnelli@unito.it>"
LABEL org.opencontainers.image.base.name="docker.io/ubuntu:noble"
LABEL org.opencontainers.image.created="${CREATION_DATE}"
LABEL org.opencontainers.image.licenses="OpenSSL"
LABEL org.opencontainers.image.revision="${COMMIT}"
LABEL org.opencontainers.image.ref.name="alphaunito/slurmd"
LABEL org.opencontainers.image.source="https://github.com/alpha-unito/docker-for-hpc"
LABEL org.opencontainers.image.title="Slurm compute node"
LABEL org.opencontainers.image.version="23.11.4"

RUN apt update                                                                             \
 && apt install -y --no-install-recommends                                                 \
        ca-certificates                                                                    \
        dnsutils                                                                           \
        hwloc                                                                              \
        mailutils                                                                          \
        munge                                                                              \
        openmpi-bin                                                                        \
        openssh-server                                                                     \
        iproute2                                                                           \
        slurm-client                                                                       \
        slurmd                                                                             \
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
        /var/spool/slurmd                                                                  \
 && chown munge:munge                                                                      \
        /run/munge                                                                         \
        /var/log/munge                                                                     \
 && chown slurm:slurm                                                                      \
        /var/log/slurm                                                                     \
        /var/run/slurm                                                                     \
        /var/spool/slurmd                                                                  \
 && adduser                                                                                \
        --disabled-password                                                                \
        --gecos ""                                                                         \
        hpcuser

COPY config/supervisord.conf                                                               \
     /etc/supervisor/conf.d/

COPY scripts/healthcheck                                                                   \
     scripts/run-slurmd                                                                    \
     /bin/

HEALTHCHECK --start-period=10s CMD healthcheck

EXPOSE 22

WORKDIR /home/hpcuser

ENTRYPOINT ["supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]