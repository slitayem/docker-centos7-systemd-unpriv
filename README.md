[![Docker Repository on Quay](https://quay.io/repository/vlisivka/docker-centos7-systemd-unpriv/status "Docker Repository on Quay")](https://quay.io/repository/vlisivka/docker-centos7-systemd-unpriv)

[![Docker Repository on Docker hub](http://dockeri.co/image/vlisivka/docker-centos7-systemd-unpriv)](https://hub.docker.com/r/vlisivka/docker-centos7-systemd-unpriv/)

# Description

This is image of CentOS7 with systemd installed, which can be ran in
unprivileged mode, or even in privileged mode (but use it with caution:
in privileged mode systemd can affect host system, use
--cap-add=SYS_ADMIN instead).

It contains initialization script /usr/sbin/init.sh, which must be ran as
container initialization script (pid 1), either directly using CMD
["/usr/sbin/init.sh"], or from a wrapper script via exec.

# RPM
As alternative, rpm package can be used to modify an existing container,
see spec/ directory for details. Dockerfile must contain same
instructions, except that "copy files/ /" will be replaced by "yum
install docker-centos7-systemd-unpriv".

RPM package is designed in such way that it cannot harm a hw/vm based
system, because installed files will be ran when /usr/sbin/init.sh will
be executed only, so it safe to add package as dependency to a
metapackage without checking is hw/vm or container is targetted by
installation.

Add this text to your Dockerfile to use rpm instead of depending on
vlisivka/docker-centos7-systemd-unpriv container.


    # Instruct systemd to work properly in container. (Required).
    ENV container=docker

    # NOTE: Systemd needs /sys/fs/cgroup directoriy to be mounted from host in
    # read-only mode. (Required).

    # Systemd needs /run directory to be a mountpoint, otherwise it will try
    # to mount tmpfs here (and will fail).  (Required).
    VOLUME /run

    # Install initialization script, which will execute kickstart scripts and
    # then will start systemd as pid 1.
    RUN rpm -vi https://github.com/vlisivka/docker-centos7-systemd-unpriv/releases/download/v1.0/docker-centos7-systemd-unpriv-1.0-1.el7.centos.noarch.rpm

    # Run systemd by default via init.sh script, to start required services.
    CMD ["/usr/sbin/init.sh"]

    # NOTE: Run container with "--stop-signal=$(kill -l RTMIN+3)" option to
    # shutdown container using "docker stop CONTAINER", OR run
    # /usr/local/sbin/shutdown.sh script as root from container and then kill
    # container using "docker kill CONTAINER".

See run.sh script with example how to start container properly.

# Logging

Systemd and journald logs are forwarded to console, but option -t must be
used with docker run command to create /dev/console. Add option
"Storage=none" to journald configuration to disable writting of logs to
disk in container, i.e. to use docker/kubernetes logging system only.

# Shutdown

To shutdown container properly:

  * (docker 1.9) run container with option `docker run --stop-signal=$(kill -l RTMIN+3) CONTAINER`, so command `docker stop` will work properly;
  * OR kill server with signal RTMIN+3: `docker kill --signal=$(kill -l RTMIN+3) CONTAINER` ;
  * OR attach to container and execute command `halt` .
