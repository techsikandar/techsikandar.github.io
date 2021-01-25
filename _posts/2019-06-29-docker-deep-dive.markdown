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
<h1>{{ "Storage Driver — another game changer" }}</h1>

<h1>{{ "What’s next?" }}</h1>

If you are interested in docker (or container) internals, then you can play with Linux namespaces & CGroups and even create your own simple container. This will involve some work but will help in developing a better understanding.

You can also look at the different storage drivers, switch to them, observe how they are storing files.