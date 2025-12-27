# Running Podman Containers with Systemd using Quadlets

It is nice to have systemd run containers in a runtime that is well-understood. We are at the point now in history where this is easy to do on a system with Podman installed, using Quadlets.

Files for root containers go in `/etc/containers/systemd`. They include `.container`, `.network`, `.volume` and other files. These files define the resources required for the containers to run. These files can be easily generated with [podlet](https://github.com/containers/podlet), which is an awesome standalone tool that can generate these files for you from a `podman run` command line, or `docker-compose.yml` file.

When using `podlet`, use the `--install` option if you want the containers to start at bootup. This adds an [Install] section to the `.container` file(s), which is the way that systemd knows to start the container at system boot. You can't `systemctl enable` the containers because they are dynamically generated.

From firsthand experience, `podlet` seems to be good at knowing which options are available in the systemd/quadlet parser on the system where it is running. It said that `PodmanArgs=--memory 2g` should be used, even though the man page for `systemd.container` says that `Memory=` is a valid directive. It probably is, but not on the Rocky 9 system that I was running the containers on.

You can troubleshoot the quadlet files using the method described in [this RedHat blog post](https://www.redhat.com/en/blog/quadlet-podman):
```sh
$ /usr/libexec/podman/quadlet -dryrun
```

An example generated `.container` file that is in use right now is
```ini
[Container]
Image=ghcr.io/rhebl/games-node:latest
PodmanArgs=--memory 2g
# Fronted by NGINX serving TLS
PublishPort=127.0.0.1:3080:3000
Volume=/srv/games-node/config/:/usr/app/config:Z
# Sometimes this container has to be force-stopped because Node doesn't want to die
StopTimeout=10

# Start by default on boot
[Install]
WantedBy=default.target
```
