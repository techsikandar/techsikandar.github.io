---
layout: post
title:  "Docker Deep Dive"
date:   2019-06-29 21:13:27 -0500
categories: docker
---

<h1>{{ "Docker 101" }}</h1>

The idea of shipping software, in a reliable, automated & decoupled way, is fascinating. It’s even more critical specially these days when software is becoming a complex piece of art, global yet local, polygot and deployed in public-private-hybrid cloud environments. We need a standardized way to package and deploy our software anywhere without worrying how to do it.

So, the question is how to package & deploy software in various environments without any versioning or libraries conflict?
How about putting everything in a virtual machine? Well, it’s not too bad. But the problem is that VM will have lots of other things as well, including the OS kernel and other resources like hard drive, memory, etc. It’s a lot. Plus, it will consume more CPU resources and will take more time to start. We don’t what that.

Fast forward ..

Sandbox everything we need but not too much. Linux kernel’s LXC has this cool namespace feature which helps us to isolate the system resources for individual processes. Docker uses this very feature to create a sandbox (a container) which is totally isolated from other containers.

<h1>{{ "Linux Container (LXC)" }}</h1>

LXC is Linux kernel’s containment feature which allows us to create and manage application container. It uses features like CGroups, Kernel namespace, Kernel capabilities, etc. LXC helps Docker to create an environment as closely as possible to the standard Linux installation but without the Kernel. Namespace & CGroups are at the core of Docker container. Let’s see what are CGroups & Kernel Namespace.

<h1>{{ "CGroups" }}</h1>

CGroups allows you to allocate, limit or control the resources allocated to a process. With CGroups, you can have a group of processes to not exceed the memory & CPU limit which you have configured, or, you can prioritize their resource usage. So, why will you need that? Well, you certainly don’t want the container to consume all of the host system’s resources.

CGroups are hierarchical in nature and each hierarchy represents a resource, called subsystem, like CPU, memory, etc. You can view this file to see all mounted subsystems:

{% highlight ruby %}
# /proc/cgroups
{% endhighlight %}

![cgroups](/assets/docker/cgroups.png)

And then you can view the contents of these subsystems with this command:

{% highlight ruby %}
# systemd-cgls cpu 
# systemd-cgls cpuset 
# systemd-cgls memory
{% endhighlight %}

<h1>{{ "Kernel Namespace" }}</h1>

Kernel namespace on the other hand limits what a process can see. This is important from container’s point of view, i.e., to control what a process can or cannot see. This is really essential for containers to work. PID namespace isolate the container processes. For example,

![busybox](/assets/docker/busybox.png)

I am running a simple container `busybox` in interactive terminal mode. Have a look at the PIDs `1` & `7` for `sh` & `ps -ef` respectively. Now, let’s have a look at the host system where docker is running.

![busybox pids](/assets/docker/busyboxpids.png)

The PIDs are mapped to different PIDs in host. You can confirm that by executing the command below which will list all PIDs under a container.

{% highlight ruby %}
# docker top <container id>
{% endhighlight %}

So, what does it mean? Well, the container thinks it is running it’s own process within it’s own boundary but it reality is running within the host system mapped to a different process id. You can remap the PIDs if you want but that would involve some configuration. You can also view the processes running inside the container from the Linux container group (CGroups) by executing the following command:

![systemd cgls](/assets/docker/systemdcgls.png)

<h2>{{ "Wait a second .." }}</h2>

If Docker is using Linux kernel’s features to create the container then how i am able to run it on my Windows / Mac? Because, under the hood, it installs Linux Virtual Machine and then runs the container on this VM

<h1>{{ "Storage Driver — another game changer" }}</h1>

Before going to to storage drivers, let’s first understand docker image which is built from a series of layers. Layer is nothing but an instruction in the Dockerfile. There are lower level layers which are read-only and a thin upper level layer, which is the last layer, and is also called container layer, have read-write access. The combined view of these layers is combined into a single merged view. The writable layer is the major difference between the image and container.

The storage driver controls how images and containers are stored. Docker supports different storage drivers but recommends `overlay2`. You can change it if you want. The recommendation is based upon the OS you are using.

![ubuntu](/assets/docker/ubuntu.png)

`overlay2` is a newer version of overlay which is a union file system which in turn employs `copy-on-write (COW)` strategy. It is this copy-on-write strategy which speeds up container creation. You should `keep this layer as thin as possible` and if you have write-intensive applications, then you should use docker volumes which are independent of container life cycle and can also be shared among containers.

<h1>{{ "What’s next?" }}</h1>

If you are interested in docker (or container) internals, then you can play with Linux namespaces & CGroups and even create your own simple container. This will involve some work but will help in developing a better understanding.

You can also look at the different storage drivers, switch to them, observe how they are storing files.