---
image: 
layout: post
sitemap: false
---


# Cluster Server

<img src="/assets/img/portfolio/cluster/Cluster.png" alt="Cluster Server">

In information technology, redundancy is crucial for ensuring system reliability. By eliminating single points of failure, it enhances overall system stability. We’ve all experienced the frustration of trying to access an unavailable system—especially when it’s needed most. For large organizations, unexpected downtime can result in significant financial and operational consequences.
For this project, I set out to gain hands-on experience in building highly available systems on a smaller scale. Using Docker Swarm and a cluster of Raspberry Pis, I created a highly available server to explore the principles of redundancy and fault tolerance.

## Why Docker Swarm

Previously, I used my Raspberry Pi to run Docker containers, including <a href="https://pi-hole.net/" target="_blank"> Pi-Hole.</a> While researching highly available container orchestration solutions, I explored both Docker Swarm and Kubernetes. Kubernetes, with its complexity and extensive customization options, is well-suited for large, intricate systems. However, for my use case, it felt like overkill. Instead, I opted for Docker Swarm due to its simplicity and ease of use, making it a better fit for small-scale environments.

### Swarm Components

A docker swarm cluster consists of:

<ul>

	<li> Manager nodes - handling cluster management tasks such as scheduling services and maintaining the overall state of the system.</li>
	<li> Worker nodes - Nodes that carry out tasks and run the containers.</li>
</ul>


Manager nodes can also be worker nodes, in my case this is what I am doing.


### Overlay network

An overlay network is a virtual network that sits on top of the physical network interface. By having each node sitting on the same overlay network, services are able to communicate reagardless of their physical location.


## Hardware

Having a history of tinkering with Raspberry Pis, I decided to use them for my cluster. I already owned three, so I only needed to purchase one more to complete the setup. Raspberry Pis are low-power devices, making them ideal for always-on systems, especially when handling lightweight workloads. Another reason for choosing the Raspberry Pi is its strong community of enthusiasts, who provide a wealth of resources and support.
For my cluster, I used <a href="https://www.amazon.com/GeeekPi-Cluster-Raspberry-Heatsink-Stackable/dp/B07MW24S61?source=ps-sl-shoppingads-lpcontext&ref_=fplfs&smid=AOP0CH6UTUPHT&gQT=2&th=1" target="_blank">this case</a>, which offers ample space for proper ventilation. To enable communication between the systems, they needed to be connected through a switch, so I used a 5-port switch for the setup.


## System Setup

One of the most satisfying parts of this project was assembling the system. I wanted a setup where a single power connection would bring everything online seamlessly. To achieve this, I used a 65W power supply and USB power cables for the Raspberry Pis.
Ethernet switches typically come with a standard socket plug, but since I wanted to power the entire system from a single source, I modified the switch’s power connection. I cut off the socket plug from its 5V power cable and soldered it to a USB port, allowing me to power the switch via a power bank. This way, the entire cluster required only one cable from the power bank to stay operational.
For the operating system, I chose Raspberry Pi OS, a system based on Debian Linux. It is reliable, open-source, and well-optimized for Raspberry Pi hardware. 


## Setting up Docker Swarm

The initial setup was quite easy, only requiring the following two commands.

On the manager node, we run:
```
docker swarm init
```

After running this we are given a token that allows other nodes to join as workers.


On the worker nodes we run: 
```
docker swarm join --token <token> <manager_node_ip>:2377
```

With just these commands, the cluster is up and running. All devices are a part of the 


## Security

Cybersecurity is a multifaceted endeavor—securing a system requires vigilance at every level. The best approach depends on the system’s specific implementation, with various strategies to consider. Below are some of the key security configurations I implemented for the Raspberry Pi system.

### Require Root Password

When I first configured a Raspberry Pi system, I was surprised to find that it does not require a root password for sudo commands. As security practitioners, we recognize this as a significant security risk—any user with access to the system can easily escalate their privileges. Since the Raspberry Pi is designed to be beginner-friendly, I assume this decision was made to simplify usability.
However, for security reasons, I changed this default behavior. To do so, I modified the sudoers configuration by navigating to the /etc/sudoers.d directory and changing the line: 
```
pi ALL=(ALL) NOPASSWD: ALL 
```
To

```
pi ALL=(ALL) ALL
```

### SSH Config

The next change I made was to the sshd_config file, as SSH is my primary method of connecting to these systems. To enhance security, I enabled key-based authentication, which is generally more secure than password-based access.
I also changed the default SSH listening port. While I don’t expose SSH ports to the internet—since doing so invites unnecessary automated attacks—I still take this precaution on any system where SSH is enabled. Instead of exposing SSH directly, I use a VPN tunnel to establish a secure connection to the system.


### Sysctl.conf

Whenever I configure a Linux server, I always review the /etc/sysctl.conf file, which contains kernel and networking parameters. Some of the key changes I made include disabling IPv6, disabling ICMP echo and broadcasts, enabling tcp_syncookies to mitigate SYN flood DDoS attacks, and applying several other security and performance optimizations.
This file offers a wide range of configurable options, allowing for fine-tuned adjustments to enhance both security and system performance.


## NFS Server

To share configuration files across all nodes in the cluster, I decided to implement a Network File System (NFS) server. NFS is a protocol that enables file sharing between computers, similar to the SMB protocol in Windows. In this setup, the NFS server is hosted on the second node.
I also utilized AutoFS, a program that automatically mounts file systems when accessed and can be configured to unmount them after a specified period of inactivity. This feature helps mitigate potential issues related to improper mounting of the file system.


## Conclusion

This project has been an invaluable learning experience. It has allowed me to deepen my understanding of high availability systems, redundancy, and fault tolerance. The simplicity of Docker Swarm suited my small-scale environment perfectly, while the Raspberry Pi provided an accessible and powerful platform for experimentation. Moving forward, I feel more confident in applying these principles to larger, more complex systems, andI look forward to expanding the number of services I will run on the cluster.
