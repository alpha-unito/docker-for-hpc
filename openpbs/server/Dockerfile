FROM rockylinux:8.8

ARG COMMIT="none"
ARG CREATION_DATE="none"

LABEL org.opencontainers.image.authors="Iacopo Colonnelli <iacopo.colonnelli@unito.it>"
LABEL org.opencontainers.image.base.name="docker.io/rockylinux:8.8"
LABEL org.opencontainers.image.created="${CREATION_DATE}"
LABEL org.opencontainers.image.licenses="OpenSSL"
LABEL org.opencontainers.image.revision="${COMMIT}"
LABEL org.opencontainers.image.ref.name="alphaunito/openpbs-server"
LABEL org.opencontainers.image.source="https://github.com/alpha-unito/docker-for-hpc"
LABEL org.opencontainers.image.title="OpenPBS management node"
LABEL org.opencontainers.image.version="23.06.06"

RUN curl -fsSL -O https://vcdn.altair.com/rl/OpenPBS/openpbs_23.06.06.rockylinux_8.8.zip    \
 && dnf install -y epel-release                                                             \
 && dnf install -y                                                                          \
        findutils                                                                           \
        munge                                                                               \
        openssh-clients                                                                     \
        openssh-server                                                                      \
        procps                                                                              \
        supervisor                                                                          \
        unzip                                                                               \
 && unzip openpbs_23.06.06.rockylinux_8.8.zip                                               \
 && dnf install -y openpbs_23.06.06.rockylinux_8.8/openpbs-server-23.06.06-0.x86_64.rpm     \
 && rm -rf openpbs_23.06.06.rockylinux_8.8 openpbs_23.06.06.rockylinux_8.8.zip              \
 && dnf clean all                                                                           \
 && ssh-keygen -A                                                                           \
 && curl -fsSL -o /bin/gosu                                                                 \
        https://github.com/tianon/gosu/releases/download/1.17/gosu-amd64                    \
 && chmod +x /bin/gosu                                                                      \
 && gosu nobody true                                                                        \
 && mkdir -p                                                                                \
        /run/munge                                                                          \
        /run/sshd                                                                           \
        /var/log/munge                                                                      \
 && chown munge:munge                                                                       \
        /run/munge                                                                          \
        /var/log/munge                                                                      \
 && ln -s /usr/lib64/libmunge.so.2 /usr/lib64/libmunge.so                                   \
 && adduser hpcuser

ENV PATH="${PATH}:/opt/pbs/bin"

COPY config/pbs.conf                                                                        \
    /etc/pbs.conf

COPY config/supervisord.conf                                                                \
     /etc/supervisor/conf.d/

COPY scripts/healthcheck                                                                    \
     scripts/run-munge                                                                      \
     scripts/run-openpbs                                                                    \
     scripts/run-sshd                                                                       \
     /bin/

HEALTHCHECK --start-period=10s CMD healthcheck

EXPOSE 22 15001-15009 17001

WORKDIR /home/hpcuser

ENTRYPOINT ["supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]