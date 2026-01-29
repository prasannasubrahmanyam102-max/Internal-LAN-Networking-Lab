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

- Three Operating Systems has been installed on Virtual box.
- Kali Linux, Windows 10 and Ubuntu
-----------------------------------
|Host	          |Operating System	|
|Kali SOC=	    |Kali Linux	      |
|Win10-victim	  |Windows 10	      |
|Ubuntu-SOC-lab	|Ubuntu           |
-----------------------------------
## Network Configuration:

- Network Type: Internal SOC Lab (Internal Only)
- DNS Server: Not configured
- DHCP Server: Not Configured
- Default Gateway: Not Configured

## Initial Observation

- No DHCP server was available in the internal network.
- Windows self assigned an IPV4 link-local address (169.254.55.204) - APIPA and windows also configured an IPV6 link-local address as shown below.
- Windows implements IPv4 Link-Local Addressing by default to preserve limited connectivity when DHCP is unavailable. Therefore, windows self assigned IPV4 address (APIPA) and an IPV6 address
- Kali Linux and Ubuntu did not auto-assign IPv4 addresses by design.
- Initially, Linux systems did not display visible IPv6 addresses despite interfaces being UP.
- IPv6 was available but not yet visible in standard interface output.
- Link-local IPv6 addresses became visible after Neighbor Discovery was triggered.

### Evidence

- Windows IPv4 address observed: `169.254.55.204`
- Windows IPv6 link-local address observed: `fe80::/64`
- Kali and Ubuntu showed no IPv4 address in `ip a`
- IPv6 addresses appeared in ICMPv6 echo replies after multicast traffic
