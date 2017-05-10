# FreeIPA server in Docker

This repository contains the Dockerfile and associated assets for
building a FreeIPA server Docker image from the official yum repo.

Docker is required:

Install docker 1.10+:

    yum install -y docker

Start the service:

    systemctl start docker

To build the image, run in the root of the repository:

    docker build -t freeipa-server .

The repository contains multiple `Dockerfile`s for various
operating systems. Use `-f` option to `docker build` to pick
different than default target.

Create directory which will hold the server data:

    mkdir /var/lib/ipa-data

On SELinux enabled systems,

    setsebool -P container_manage_cgroup 1

might be needed to enable running systemd in the containers.

You then run the container with

    docker run --name freeipa-server-container -ti \
       -h ipa.example.test \
       -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
       --tmpfs /run --tmpfs /tmp \
       -v /var/lib/ipa-data:/data:Z freeipa-server [ opts ]

Standard `ipa-server-install` will be started and you can configure
the server.

(A full example of how-to run, start and stop, review the repository file
called how-to-run-start-stop.txt)

The option `--name` assigns the container a name that can be used
later with `docker start`, `docker stop` and other commands.
Command `ipa-server-install` is invoked non-interactively the first
time the container is run.

The `-ti` parameters are optional and are used for get a terminal,
for interactive configuration sessions.

The container can the be started and stopped:

    docker stop freeipa-server-container
    docker start -ai freeipa-server-container

If you want to use the FreeIPA server not just from the host
where it is running but from external machines as well, you
might want to use the `-p` options to make the services accessible
externally. You will then likely want to also specify the
`IPA_SERVER_IP` environment variable via the `-e` option to
define what IP address should the server put to DNS as its
address. Starting the server would then be

    docker run -e IPA_SERVER_IP=10.12.0.98 -p 53:53/udp -p 53:53 \
        -p 80:80 -p 443:443 -p 389:389 -p 636:636 -p 88:88 -p 464:464 \
	-p 88:88/udp -p 464:464/udp -p 123:123/udp -p 7389:7389 \
	-p 9443:9443 -p 9444:9444 -p 9445:9445 ...

If you have existing container with data volume, it should be safe
to shut it down and run new one based on newer image, with the same
data directory bind-mounted to /data. The container will detect
that it is running with data produced by different image and attempt
to upgrade the configuration and data. Of course, keeping backup
of the data directory for cases when the upgrade process fails
is recommended.

# Copyright 

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
