title: container related open source projects
date: 2016-12-27 08:56:02
tags: [container]
---

This post is about container related open source projects.

<!-- more -->

Operating System
=====================

CoreOS
-----------------

[coreOS][https://coreos.com/]  is a new Linux distribution that has been re-architected to provide features needed to run modern infrastructure stacks. The strategies and architectures that influence CoreOS allow companies like Google, Facebook and Twitter to run their services at scale with high resilience.

By default CoreOS is pre-configured with tools to run Linux containers and be distributed. A key feature is that the container runtime (e.g. Docker) is automatically configured on each CoreOS machine. Also CoreOS provides automatic OS updates which means you get all Updates per default and you don’t have to worry about old versions.

On CoreOS you can use docker with fleet. Fleet is a distributed init system which presents your entire cluster as a single init system. You are able to start fleet units by using extended systemd unit files. With fleet you are able to run distributed containerized apps.

A major benefit of running CoreOS is etcd. etcd is a distributed key value store which is used by many projects like Kubernetes, Cloud Foundry and many more. You can use etcd for simple service discovery and much more. Also major cloud providers offer CoreOS support.


RancherOS
-----------------

[rancherOS][http://rancher.com/rancher-os/]  is the smallest, easiest way to run Docker in production. Everything in RancherOS is a container managed by Docker. 

This includes system services such as udev and syslog. RancherOS is dramatically smaller than most traditional operating systems, because it only includes the services necessary to run Docker. By removing unnecessary libraries and services, requirements for security patches and other maintenance are dramatically reduced. This is possible because with Docker, users typically package all necessary libraries into their containers.

Another way in which RancherOS is designed specifically for running Docker is that it always runs the latest version of Docker. This allows users to take advantage of the latest Docker capabilities and bug fixes.

Like other minimalist Linux distributions, RancherOS boots incredibly quickly, generally in 5-10 seconds. Starting Docker containers is nearly instant, similar to starting any other process. This quickness is ideal for organizations adopting microservices and autoscaling.


Container Tools
=================

Docker
-----------------

[Docker][https://www.docker.com/] is the world's leading software containerization platform. Docker containers wrap a piece of software in a complete filesystem that contains everything needed to run: code, runtime, system tools, system libraries – anything that can be installed on a server. This guarantees that the software will always run the same, regardless of its environment.

Hypercontainer
-----------------

[Hypercontainer][https://hypercontainer.io/] Hypervisor-agnostic Docker Runtime that allows you to run Docker images on any hypervisor (KVM, Xen, etc.).

Technically speaking,

HyperContainer = Hypervisor + Kernel + Docker Image
By containing applications within separate VM instances and kernel spaces, HyperContainer is able to offer an excellent Hardware-enforced Isolation, which is much needed in multi-tenant environments.

HyperContainer also promises Immutable Infrastructure by eliminating the middle layer of Guest OS, along with the hassle to configure and manage them.


runC
----------------------

[runC][https://github.com/opencontainers/runc] is a low-level container runtime and an implementation of the Open Container Initiative specification. runC exposes and expects a user to understand low-level details of the host operating system and configuration. It requires the user to separately download or cryptographically verify container images, and for "higher level tools" to prepare the container filesystem. runC does not have a centralized daemon, and, given a properly configured "OCI bundle", can be integrated with init systems such as upstart and systemd.

containerd
------------------

[containerd][https://containerd.tools/] is a daemon to control runC. It has a command-line tool called ctr which is used to interact with the containerd daemon. This makes the containerd process model similar to that of the Docker process model, illustrated above.

Unlike the Docker daemon it has a reduced feature set; not supporting image download, for example.

rkt
------------------

[rkt][https://github.com/coreos/rkt](pronounced "rock-it") is a CLI for running app containers on Linux. rkt is designed to be secure, composable, and standards-based.

Some of rkt's key features and goals include:

Security: rkt is developed with a principle of "secure-by-default", and includes a number of important security features like support for SELinux, TPM measurement, and running app containers in hardware-isolated VMs.
Composability: rkt is designed for first-class integration with init systems (systemd, upstart) and cluster orchestration tools (fleet, Kubernetes, Nomad), and supports swappable execution engines.
Open standards and compatibility: rkt implements the appc specification, supports the Container Networking Interface specification, can also run Docker images, and OCI images via docker2aci. Native OCI image support is in development.


