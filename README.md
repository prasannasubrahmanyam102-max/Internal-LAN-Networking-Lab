# Internal-LAN-Networking-Lab
Hands-on lab exploring internal LAN communication using IPv4/IPv6, Neighbor Discovery behavior, static addressing, and traffic analysis in an isolated environment.
## Project Overview:

- This project analyzes internal LAN communication in an isolated virtual network.
- The environment operates without DNS, DHCP, or a default gateway.
- All hosts rely on IPv6 link-local addressing for communication.
## Objective:

- Analyze internal LAN communication without IPv4 infrastructure
- Observe IPv6 link-local behavior and Neighbor Discovery
- Capture and analyze traffic from a SOC visibility perspective

## Lab Setup:

- Three Operating Systems were installed on Virtual box.
- Kali Linux, Windows 10 and Ubuntu
|Host	          |Operating System	|
-----------------------------------
|Kali SOC=	    |Kali Linux	      |
|Win10-victim	  |Windows 10	      |
|Ubuntu-SOC-lab	|Ubuntu           |

## Network Configuration:

- Network Type: Internal SOC Lab (Internal Only)
- DNS Server: Not configured
- DHCP Server: Not Configured
- Default Gateway: Not Configured

## Initial Observation

- No DHCP server was available in the internal network.
- Windows self assigned an IPV4 link-local address (169.254.55.204) - APIPA and Windows also configured an IPV6 link-local address as shown below.
- Windows implements IPv4 Link-Local Addressing by default to preserve limited connectivity when DHCP is unavailable. Therefore, windows self assigned IPV4 address (APIPA) and an IPV6 address
- Kali Linux and Ubuntu did not auto-assign IPv4 addresses by design.
- Initially, Linux systems did not display visible IPv6 addresses despite interfaces being UP.
- IPv6 was available but not yet visible in standard interface output.
- Link-local IPv6 addresses became visible after Neighbor Discovery was triggered.

### Evidence

- Windows IPv4 address observed: `169.254.55.204`
- Windows IPv6 link-local address observed: `fe80::/64`
- Kali and Ubuntu showed no IP address in `ip a`
- IPv6 addresses appeared in ICMPv6 echo replies after multicast traffic
![Windows IP configuration showing APIPA and IPv6](obs1-win-ipconfig.png)
![Kali IP a showing Connection up but no ip address](obs2-kali-ipa.png)
![Ubuntu IP a showing Connection up but no ip address](obs3-ubuntu-ipa.png)
### **IPv6 Activation Observation**

- IPv6 multicast traffic was initiated using `ping -6 ff02::1` on Linux.
- On Kali Linux, multicast required explicit interface scoping (`%eth0`).
- Initial multicast attempts failed without interface scope.
- After specifying the interface, ICMPv6 echo replies were received.
- Neighbor Discovery activity caused Linux systems to display IPv6 link-local addresses in `ip a`.
![Kali linux doesn't support ping -6 messages without interface scoping](obs4-ping-IPV6-from-linux.png)
![Kali linux IPV6 ping messages were successful after using interface](obs5-ping-IPV6-from-linux-successful.png)
![Checking IP a on Ubuntu for re-confirmation of IPV6 address in link-local network](obs6-checking-IP-address-on-Ubuntu.png)
### **Impact**

- IPv6 communication occurred even when IPv4 connectivity was unavailable.
- Hosts were able to discover and communicate with each other without DNS, DHCP, or a gateway.
- Linux systems participated in IPv6 traffic before showing visible IP addresses in standard interface output.
- IPv6 multicast enables internal host discovery by default.
- Security monitoring focused only on IPv4 traffic may miss active IPv6-based communication.
## Observation-2

- All host have stable IPV6 address
- Send Ping commands from one host to other using IPV6 address
- In an isolated internal LAN without DHCPv6, DNS, or router advertisements, all hosts rely on IPv6 link-local addresses (`fe80::/64`) for local communication.
- IPv6 link-local connectivity depends entirely on Neighbor Discovery Protocol (NDP), which maintains a short-lived neighbor cache. In virtualized environments, this cache is frequently invalidated due to interface resets, VM state changes, or OS network stack behavior.
- As a result, ICMPv6 echo requests may succeed intermittently and later fail without configuration changes. This behavior is expected and does not indicate network failure, but rather highlights the instability of relying solely on link-local IPv6 for persistent communication.

**Commands used in Lab**

- On Ubuntu, Ping IPV6 address%interface -c 3
- On kali, Ping -c 3 IPV6 address%interface
- On windows Ping IPV6 address%Idx -n 3
- For getting Index (Idx) using netsh interface IPV6 show interfaces
- On Linux, To fresh temporary IPV6 manually, ip -6 neigh show or ip -6 neigh flush all
- On ubuntu , to reset interface, sudo ip link set interface down
- sudo ip link set interface up
- On kali, sudo ip interface down
- sudo ip interface up

## Evidence

- Sometimes, IPV6 address is not shown visibly and Neighbor solicitation, Neighbor Advertisement are mandatory in link-local network. However, Ping reply doesn’t work for sometimes.
- IPV6 Echo replies are not stable.

**Ubuntu:**

- does not “hold” link-local aggressively
- may not display it until traffic triggers NDP
- may remove it after cache timeout

**Windows:**

- aggressively keeps fe80::
- responds locally even when others drop cache

**Kali:**

- more aggressive than Ubuntu
- but still volatile
![Re-checking IP addresses on windows and pinged IPV6 messages to another host](obs7-checking-IP-address-on-Windows-&-ping-IPV6-messages.png)
![pinged IPV6 messages to another host from windows failed](obs8-IPV6-pinged-messages-failed.png)
![Re-checking IP addresses on Ubuntu & forwarded ping IPV6 messages](obs9-IPV6-pinged-messages-on-Ubuntu.png)
![Re-checking IP addresses on Kali](obs10-checking-IP-address-on-kali.png)
## Observation-3

- **What I observed in my internal LAN IPv6 lab is this:**
- After restarting a system, the **first ping works successfully.**
- But after some time, **ping to other hosts stops working.**
- However, **ping to the same host from itself always works 100%**
- If I try to ping another host after the first successful ping, it often fails.
![Ping works successfully after restarting the system](obs11-ping-works-successfully-after-restart.png)
![Ping failed](obs12-ping-failed.png)
![Ping failed](obs13-ping-failed.png)
## Observation-4:

I have observed the continues ping fails. The probable root cause of this issue is due to Window defender firewall.
As windows work aggressively in Link-Local network but, Window defender firewall blocks the Echo replies. 
Window defender firewall was disabled but still ping messages failed
Ping between Kali vs Ubuntu is also not working. So, There may be an another root cause
![Suspecting window defender firewall blocks the traffic. So, window defender firewall disabled](obs14-window-defender-firewall-disabling.png)
![Suspecting window defender firewall blocks the traffic. So, window defender firewall disabled](obs15-window-defender-firewall-disabling.png)
![Ping messages forwarding after firewall disabling](obs16-ping-checking-after-firewall-disabling.png)
![Ping messages forwarding after firewall disabling](obs17-ping-checking-after-firewall-disabling.png)
![Ping messages forwarding after firewall disabling](obs18-ping-checking-after-firewall-disabling.png)
![Ping messages forwarding after firewall disabling](obs19-ping-failing.png)
![checking neighbor discovery after continues ping fails](obs20-neighbor-discovery-flushing.png)
![observing no neighbor](obs21-no-neighbor-showing.png)



  
