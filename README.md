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
## Observation-5:

After observing the no results from ip -6 neigh show, ping fails because no stable ND table.
If ND is not stable in IPV6, data panel is not stable and ping messages fail.
This may be because of topology i.e., SOC Lab internal LAN. In real world, Internal LAN working with physical switch, However VM doesn’t have that. Hence, Hosts are continuously searching for RA and RD, NA and ND.
Hence, Network has been changed to Host only network and added static IPV4 & IPV6 IP address for three hosts
![Ensuring the Unstable IPV6 results in unstable ND forwards unstable data panel on kali](obs22-no-ipv6-address-on-kali.png)
![changed network topology from link-local network to host only network i.e., fron SOC Lab internal LAN to Host only network on kali](obs23-adding-staic-ip-address.png)
![adding-static-ip-addresses-on-ubuntu](obs24-adding-static-ip-addresses-on-ubuntu.png)
![flushing the neighbor history ipv4 and ipv6 to clear the existing data if availabe](obs25-neigh-ipv4-&-ipv6-flushes-on-ubuntu.png)
![Even though link-local IP addressess are stable on windows, due to unstable linux behaviour, windows also changed to static network for making communication convinience](obs26-adding-ipv4-&-ipv6-addresses-on-windows.png)
## Results:

After providing the static IP addresses, ping messages are successful which means ND table is stable
![after assigning static ip addresses, communication checking through ping request from windows](obs27-ping-worked-successfully-in-static-network.png)
![after assigning static ip addresses, communication checking through ping request from windows](obs28-ping-worked-successfully-in-static-network.png)
![after assigning static ip addresses, communication checking through ping request from kali](obs29-ping-worked-successfully-in-static-network.png)
![after assigning static ip addresses, communication checking through ping request from kali](obs30-ping-worked-successfully-in-static-network.png)
![after assigning static ip addresses, communication checking through ping request from kali](obs31-ping-worked-successfully-in-static-network.png)
![after assigning static ip addresses, communication checking through ping request from ubuntu](obs32-ping-worked-successfully-in-static-network.png)
**Note:** Unintentionally, provided temporary IP addresses for Linux i.e., assigning the IP with sudo ip add comment. After restarting the System, assigned IP addresses were flushed. Then saved IP address to network /etc/network file
![assigned static ip addressess for kali had been removed as provided ip addressess were temporarily](obs33-no-static-ip-addresses-found-on-kali.png)
![assigned static ip addressess for ubuntu had been removed as provided ip addressess were temporarily](obs34-no-static-ip-addresses-found-on-ubuntu.png)
![assigned static ip addresses were available on windows](obs35-static-ip-addressess-found-on-windows.png)
![static ip addressess were re-assigned on networkmanager on kali](obs36-re-assigning-static-ip-in-networkmanagerfile-on-kali.png)
![static ip addressess were re-assigned on networkplan on ubuntu](obs37-re-assigning-static-ip-in-networkmanagerfile-on-ubuntu.png)
![observed warning messages on ubuntu during ip re-assigning in networkplan-directory](obs38-observed-warning-messages-due-to-open-permissions-of-network-manager-&-networkplan.png)
In Ubuntu, there are two networking files. So, I moved one of the networking file to backup folder without disturbing or deleting. I saved network configuration in netplan file and changed permission as that file is open to everyone.
![found two networking files on ubuntu, moved one networking file to backup](obs39-moved-networking-file-to-backup.png)
![successfully moved networking files](obs40-moved-netplan-to-backup-directory.png)
![changed permission to netplan](obs41-changed-netplan-permissions.png)
![moved netplan backto networking directory as systemd networkd is not running and changed permissions to netplan and finally, re-assigned ip addresses](obs42-re-locate netplan-to-networking-directory.png)
![ping worked successfully and assigned ip addressess were saved in networking directory and static ip were permanent till changed later](obs43-ping-worked-successfully-on-ubuntu.png)
![ping worked successfully on kali](obs44-ping-worked-successfully-on-kali.png)
![ping worked successfully on kali](obs45-ping-worked-successfully-on-kali.png)  
## Root Cause:

Although Windows Defender Firewall initially appeared to block ICMPv6 echo replies, further testing demonstrated that firewall enforcement was not the primary cause of the intermittent connectivity.
Multiple validation steps confirmed that the underlying issue was unstable IPv6 Neighbor Discovery (ND) convergence, which prevented the control plane from establishing consistent Layer-2 to Layer-3 mappings. Because the neighbor table failed to populate reliably, the data plane could not forward ICMPv6, TCP, or UDP traffic consistently.
This conclusion was supported by the following evidence:
**Proof-1 — Firewall Elimination**
Windows Defender Firewall was fully disabled, yet IPv6 echo requests still failed, ruling out host firewall policy as the dominant factor.
**Proof-2 — Neighbor Cache Failure**
During failure states, the `netsh interface ipv6 show neighbors` and `ip -6 neigh show` commands returned empty or incomplete neighbor entries, confirming that ND resolution was not occurring.
**Proof-3 — Network Mode Change Restored Convergence**
After switching the VirtualBox network to a Host-Only adapter and assigning static IPv4 and IPv6 addresses, Neighbor Discovery stabilized and bidirectional ICMPv6, TCP, and UDP communication resumed normally.
These results indicate that the original Internal Network configuration combined with dynamic addressing caused control-plane instability, which directly impacted the data plane and produced inconsistent connectivity across all transport protocols.
## Observing the LAN Traffic using Wireshark
Observed Neighbor solicitation,advertisement, ping replies
![after observing echo icmpv6 replies successfully, internal lan travel were observed on wireshark](obs46-observed-echo-ping-replies-&-few-internal-lan-protocols-on-wireshark.png)
![exploring internal lan traffic on wireshark](obs47-internal-lan-protocols-observed-on-wireshark.png)
![exploring internal lan traffic on wireshark](obs48-internal-lan-protocols-observed-on-wireshark.png)
## Advantages of This Internal LAN Visibility Experiment
Clear Proof of How IPv6 Really Works in a LAN
Shows Root-Cause Investigation Skills
Dual-Stack Awareness (IPv4 + IPv6)

## Disadvantages / Limitations of This Setup
No Central Logging or SIEM
VirtualBox Behavior May Differ from Physical Networks
Endpoint Security Was Minimal
No IDS/IPS or network detection sensors were deployed.
The network was flat with no VLAN segmentation or internal firewall zones.
No SIEM ingestion or long-term telemetry storage was configured.
Traffic analysis relied primarily on manual packet captures.
Attack simulations were intentionally excluded in this phase.

## Potential Attacks in Internal LAN Environments
Although this project focused on baseline IPv6 communication and Neighbor Discovery stability, real enterprise LANs face a variety of internal attack techniques that were **not executed in this phase** but are relevant to visibility design:
- **IPv6 Neighbor Discovery Spoofing:** Attackers send forged Neighbor Advertisement messages to redirect traffic or perform man-in-the-middle attacks.
- **Rogue Router Advertisements:** Malicious hosts advertise themselves as default gateways to hijack traffic.
- **Internal Reconnaissance Scanning:** Tools enumerate live hosts, open ports, and IPv6 prefixes.
- **Lateral Movement via IPv6:** Attackers pivot between hosts using less-monitored IPv6 paths.
- **Multicast Abuse:** Discovery protocols can be exploited to locate targets.
- **ARP/ND Cache Poisoning:** Control-plane manipulation to intercept or disrupt communications.
- **Protocol Downgrade or Shadow Traffic:** Forcing hosts onto IPv6 when IPv4 is filtered.
These techniques were not simulated in this project; they are listed to highlight the broader LAN threat landscape.

## Main Purposes of Attacking an Internal LAN
## 1) Lateral Movement
Once inside, attackers want to:
- move from one workstation to another
- reach servers
- reach domain controllers
- reach databases
- reach backups
This is how small breaches become big incidents.

## 2) Credential Harvesting
Internal traffic lets attackers:
- sniff authentication attempts
- relay NTLM/Kerberos
- capture hashes
- replay sessions
Goal: become a domain user or admin.
## 3) Data Exfiltration Preparation
Before stealing data, attackers:
- locate file servers
- find sensitive databases
- map storage systems
Internal reconnaissance is step one.

## 4) Persistence & Backdoors
Attackers plant:
- scheduled tasks
- rogue services
- hidden admin accounts
Internal reach makes persistence easier.

## 5) Command-and-Control (C2) Hiding
They use internal hops to:
- proxy traffic
- blend in
- avoid perimeter detection

## 6) Bypass Security Controls
If IPv4 is monitored but IPv6 is not?
Attackers will absolutely pivot to IPv6.

## What Is Usually Vulnerable in Internal LANs?
Weak Segmentation
Legacy Protocols
Poor Monitoring of East-West Traffic
Misconfigured IPv6
Weak Endpoint Hardening
Shared Credentials
# **Conclusion**

This project began by analyzing internal LAN communication using IPv6 link-local addressing in an isolated network without DNS, DHCP, or a default gateway. Initial testing showed that hosts could communicate only after Neighbor Discovery converged, and that instability in ND tables frequently disrupted ICMPv6.
To isolate the root cause, the lab architecture was modified by switching to a Host-Only network adapter and assigning static IPv4 and IPv6 addresses to all systems. Under this configuration, Neighbor Discovery stabilized and consistent bidirectional connectivity was achieved across Kali Linux, Ubuntu, and Windows hosts. IPv4 testing was then performed to independently verify Layer-3 reachability and confirm that the underlying network path was sound.
Packet captures in Wireshark validated control-plane activity through Neighbor Solicitation and Neighbor Advertisement messages and demonstrated how control-plane failures directly impacted data-plane traffic. The project therefore highlights how IPv6 misconfiguration or virtualization-layer behavior can create unreliable internal connectivity and corresponding visibility gaps for security teams.
From an enterprise SOC perspective, the lab reinforces the importance of monitoring both IPv6 and IPv4 east-west traffic, inspecting control-plane protocols such as Neighbor Discovery, and validating host firewall and adapter configurations. Establishing a stable baseline network is a prerequisite for accurately detecting lateral movement, rogue infrastructure, and reconnaissance activity in future defensive testing phases.
