---
layout: post
title: How to setup docker to work with SELinux
subtitle: How to properly get docker working with Linux with SELinux enabled
tags: docker linux selinux
---

From time to time I need to get this working, either by a new computer or some reinstall. Lately I had to do this twice because
my new Dell XPS laptop needed to be serviced and I had to reinstall my OS of choice: Fedora Linux.

From now one, I assume you already have docker installed, in my case (Fedora Linux) is just a matter of typing:
`sudo dnf install docker`

# Change the default docker directory

Since I don't want docker to pollute my `/user` directory, I usually create a new directory just to hold the docker images and files.
In this case I will use `/home/docker`, since I have a very large partition for the `/home`.

`sudo vim /etc/sysconfig/docker`

add `-g /home/docker` to the OPTIONS line:

`OPTIONS='--selinux-enabled --log-driver=journald -g /home/docker'`

Now to make sure SELinux recognizes this in a proper context, we need to run:

`sudo chcon -Rt svirt_sandbox_file_t /home/docker`

To ensure you can run docker with your current user, instead of using the root user, you need to create one group and add it
to your user's groups.

# Setup docker to run with your user

<!-- from https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user -->

To ensure you can run docker with your user and ditch the `sudo` usage, you need to create group and add user to the group:

Create the docker group.

`sudo groupadd docker`

Now add your (current) user to the docker group:

`sudo usermod -aG docker $USER`

Now logout and login again, to make sure your user as the right groups loaded

To check if everything is properly set up, you can run docker:

`docker run hello-world`

and you'll be now able to run docker as your beloved user!

# Setup your workspace or project directory

Now that you have docker running by your user, you (may) want to be able to share your project folder with the container,
to make sure the process inside the docker can read your files and make some changes if necessary.

So got to the root directory of your project or your workspace, in case you have several projects and run:

`sudo chcon -Rt svirt_sandbox_file_t /path/to/worksapce_or_project`

# If you still cannot access internet inside your container

In my recent Fedora 33 installation I have to perform some changes to the firewal in order to access the internet. An bug report is already open at [Bugzilla](https://bugzilla.redhat.com/show_bug.cgi?id=1817022).
The root issue is we do not have [IP Masquerading](https://tldp.org/HOWTO/IP-Masquerade-HOWTO/ipmasq-background2.1.html) enabled.
First get the list of active zones: `sudo firewall-cmd --get-active-zones`

```
docker
  interfaces: docker0
public
  interfaces: enp56s0u1u1
```

Then I need to add the masquerade to the `public` interfaces.
`sudo firewall-cmd --zone=public --add-masquerade`

 Note, that in several search results, instead of `public` you may seen `FedoraWorkstation`, so in the previous command you need to replace `public` with `FedoraWorstation`.

# Conclusion

Since docker run their processed inside the containers as root, it's always a good idea to keep SELinux enabled, to ensure
nothing is _misbehaving_ and reading files that should not be read.

If you see anywhere people suggesting to disable SELinux, please ignore, because in this world security is never enough.
