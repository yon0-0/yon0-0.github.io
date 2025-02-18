---
image: 
layout: post

sitemap: false
---

# Open Source Firewall

 <img src="/assets/img/portfolio/firewall/dashboard.png" alt="Network Diagram"> 


I set out to implement a robust firewall and routing solution for my home network, with the goal of enhancing security while expanding my networking skills. Additionally, I aimed to reduce the cost of monthly bills by replacing traditional cable with streaming services and eliminating rental fees for third-party network equipment. To achieve this, I deployed an OPNsense firewall alongside a dedicated wireless access point (WAP). 

<a href="https://opnsense.org/about/about-opnsense/" target="_blank">OPNsense</a> is an open-source firewall and routing platform based on the FreeBSD operating system. As a fork of pfSense, it provides a modern, community-driven alternative with focus on network security. I chose OPNsense for its intuitive and polished, intuitive user interface, as well as  its fully open-source licensing model. 
 

## Network Diagram 
 <img src="/assets/img/portfolio/firewall/SOHO.png" alt="Network Diagram"> 


## Hardware

To enhance security and optimize network performance, I implemented network segmentation using VLANs. This setup consists of four VLANs:

VLAN 10 - Dedicated to network infrastructure, including switches, routers, and access points
VLAN 20 - Allocated to servers such as the NAS and cluster server
VLAN 30 - Reserved for end-user devices
VLAN 40 - Isolated for IoT devices to enhance network security  

For the firewall, I chose the <a href="https://protectli.com/product/fw4b/" target="_blank">Protectli Vault FW4B</a>, a low power, fanless mini-pc with a large heatsink for passive cooling. It has 8gb RAM, 125gb SSD, a quad core intel cpu, and 4 gigabit ethernet ports, making it a reliable choice for running OPNsense.
 
For wireless networking I deployed a <a href="https://webresources.ruckuswireless.com/datasheets/r610/ds-commscope-r610.html" target="_blank">Ruckus R610</a> access point, known for its innovative adaptive antenna technology, which dynamically optimizes Wi-Fi performance based on environmental factors. To simplify network expansion, I flashed the AP with the Ruckus Unleashed firmware, allowing one unit to act as the master, eliminating the need for a separate controller. 

The network backbone includes two managed switches. To leverage the existing coaxial infrastructure without running new CAT 6 wiring, I integrated some <a href="https://hackaday.com/2022/11/03/moca-networking-is-a-niche-solution-for-coax-lovers/" target="_blank">MoCA</a> 2.5 adapters.


## Switch configuration
 <img src="/assets/img/portfolio/firewall/switch_conf.png" alt="Switch Config"> 

On the switch, I configured 802.1q for VLAN tagging to properly segment network traffic. In my setup, I designated the ports connected to the firewall and AP as tagged (trunk) ports, Allowing them to carry traffic for multiple VLANs. Meanwhile, all other ports were configured as untagged (access) ports, each assigned to a specific VLAN. I also ensured that untagged traffic is passed to the MoCA adapter, which allows it to be sent to the second switch without requiring VLAN tagging on that link.
The reason for this configuration is that the firewall and AP are VLAN-aware devices, meaning they can properly process tagged VLAN traffic. By keeping all other devices on untagged ports, I prevent potential issues with devices that do not support VLAN tagging.  

Additionally, I configured 802.1Q PVID (Port VLAN ID) on untagged ports. This ensures that any untagged traffic arriving at a port is automatically assigned to the correct VLAN, simplifying network management and maintaining proper VLAN segmentation. 


## AP configuration

 <!--UPDATE THIS PHOTO --- <img src="/assets/img/portfolio/firewall/ruckus.png" alt="Ruckus Unleashed"> -->

The initial setup of the access point was relatively straightforward. I Created  two SSIDs: one for 5Ghz and another for 2.4Ghz. The 5Ghz SSID is designated for end-user devices and is mapped to VLAN 30, ensuring all connected clients are properly segmented. THe 2.4Ghz SSID is reserved for IoT devices and mapped to VLAN 40.

During the setup, I encountered issues with DNS resolution and the gateway not functioning correctly. After reviewing my switch configuration, I was able to resolve the gateway issue. However, DNS remained non-functional until I implemented a firewall rule explicitly allowing DNS traffic. Once this rule was in place, the access point operated as expected.


## Firewall Configuration
<img src="/assets/img/portfolio/firewall/rules.png" alt="Firewall Rules"> 

On the OPNsense firewall, I configured all four VLANs and applied the appropriate firewall rules. Once these rules were implemented, the DHCP server began assigning IP addresses as expected. As noted earlier, I had to create a rule allowing DNS traffic for VLAN 10 to ensure the access point was functional.

The VLAN setup process involved a fair amount of trial and error, particularly because I wasn’t initially sure which ports needed to be opened for certain services to work. For instance, there is a broad range of UDP ports required for certain VoIP services, which added complexity to the configuration. However, since only a few individuals are using the service, the scope of the configuration was manageable.

Fortunately, OPNsense, like pfSense, makes use of aliases—a feature that significantly simplifies the creation of firewall rules. Aliases allow you to define multiple port ranges, devices, and protocols under a single reference. This feature streamlined my rule management, enabling me to apply specific rules to certain devices. To implement this, I created static IP mappings for the relevant devices, which was straightforward given the relatively small size of the network.


### DNS
For DNS resolution, I implemented a local recursive DNS resolver combined with a DNS sinkhole. All DNS requests are first directed to the sinkhole, where they are filtered before being forwarded upstream to the recursive resolver. This approach allows legitimate DNS requests to pass through while blocking domains associated with ads, trackers, and malicious content.

For the DNS sinkhole, I chose <a href="https://github.com/AdguardTeam/AdGuardHome?tab=readme-ov-file#privacy-protection-center-for-you-and-your-devices" target="_blank">AdGuard Home</a>, an efficient tool for filtering and blocking unwanted content. As for the upstream recursive DNS resolver, I used Unbound DNS, which comes pre-installed with OPNsense. Unbound supports DNS over TLS (DoT), ensuring that DNS requests are encrypted to prevent eavesdropping and enhance privacy.

To provide additional security and reliability for DNS resolution over DoT, I configured Cloudflare and Quad9 as my nameservers. This setup not only ensures the privacy of DNS queries but also improves the overall performance of domain lookups.

### VPN

<a href="https://www.paloaltonetworks.com/cyberpedia/what-is-wireguard" target="_blank">Wireguard</a>, which comes preinstalled with OPNsense, is a modern and efficient VPN protocol known for its speed, security, and minimal codebase. The simplicity of WireGuard makes it easier to audit and faster to run compared to other VPN protocols. To configure WireGuard, I only needed to set up the interfaces, generate the keys, and define the allowed IPs for routing.

For my VPN gateway network, I selected 10.1.1.1/24, with subsequent clients receiving 10.1.1.x/32 addresses. I configured two WireGuard clients—one for my phone and another for my laptop—both of which can simultaneously establish tunnels to my firewall, ensuring secure internet access. Due to their unique IP addresses, I can apply custom firewall rules, such as granting one client access to specific resources (e.g., my servers), while keeping the other client more restricted.

In addition to WireGuard, I set up Dynamic DNS (DDNS) to associate a domain name with my dynamic IP address. Home ISPs typically assign dynamic IP addresses that change regularly, making DDNS essential for remote access. To enable DDNS on OPNsense, I installed the necessary plugin. Since I already owned a domain name from a previous reverse proxy project, setting up DDNS was a seamless process.


## Conclusion

This project has been both exciting and rewarding. It provided an excellent opportunity to apply everything I’ve learned about networking and security. While there was a significant amount of trial and error, the experience was incredibly valuable, and I gained a deeper understanding of network configuration, troubleshooting, and VPN setup.
