# Project 05 — Linux Network Security Investigation Lab

## Project Overview

This project demonstrates a basic Linux network security investigation
using command-line networking and security tools.

The investigation focuses on identifying network interfaces, routing
information, listening services, active connections, exposed ports,
service versions, SSH configuration, and basic firewall status.

The objective is to understand how a security analyst can assess the
network exposure of a Linux system and distinguish expected services
from potentially risky configurations.

## Objectives

- Identify Linux network interfaces and IP addresses.
- Examine routing and gateway information.
- Identify listening TCP and UDP services.
- Investigate active network connections.
- Scan the system using Nmap.
- Identify exposed ports and services.
- Perform basic service version detection.
- Examine SSH security configuration.
- Check basic firewall status.
- Distinguish local-only services from network-exposed services.
- Document network security findings professionally.

## Environment

- Ubuntu 24.04.3 LTS
- VirtualBox virtual machine
- OpenSSH Server
- Network interface: `enp0s3`
- IPv4 address: `10.0.2.15/24`
- Default gateway: `10.0.2.2`

## Tools Used

- `ip`
- `ss`
- `nmap`
- `resolvectl`
- `ssh`
- `ufw`
- `nft`

## Project Structure

```text
05-linux-network-security-investigation/
├── analysis/
│   └── network-analysis.txt
├── logs/
├── reports/
│   └── network-security-investigation.md
└── README.md
```
## Investigation Methodology

The investigation followed these stages:

1. Identify network interfaces and IP addresses.
2. Examine the routing table and default gateway.
3. Examine neighboring network devices.
4. Identify listening TCP and UDP services.
5. Investigate active network connections.
6. Perform local and network-interface Nmap scans.
7. Perform service version detection.
8. Examine SSH security configuration.
9. Check firewall status and packet-filtering rules.
10. Perform a UDP port scan.
11. Verify SSH connectivity through the VM's network address.
12. Assess the security significance of the findings.
13. Document the investigation and evidence.

## Key Findings

The Ubuntu laboratory system was assigned the IPv4 address
10.0.2.15/24 on interface enp0s3.

The default gateway was identified as 10.0.2.2, corresponding to the
VirtualBox NAT network.

The TCP investigation identified SSH listening on port 22 on all
IPv4 and IPv6 interfaces.

CUPS was listening on port 631, but only on the local loopback
addresses 127.0.0.1 and ::1.

An Nmap scan of 10.0.2.15 identified only 22/tcp as open among
the default TCP ports scanned.

Nmap identified the SSH service as OpenSSH 9.6p1.

The top 20 UDP ports were scanned and all were reported as closed.

No suspicious established TCP connections were observed during the
active connection investigation.

## SSH Security Assessment

SSH was found to have password authentication enabled.

Public-key authentication was also enabled.

The SSH service was listening on all interfaces:

0.0.0.0:22
[::]:22

The SSH service was successfully reached through the VM's network
address 10.0.2.15.

UFW was inactive and no nftables rules were displayed during the
investigation.

These findings represent a security configuration consideration.
They do not by themselves demonstrate that the system has been
compromised.

## Local vs Network-Exposed Services

The investigation demonstrated an important distinction between a
service listening on a system and a service exposed to the network.

CUPS was listening on:

127.0.0.1:631

and:

[::1]:631

Therefore, it was restricted to the local system.

SSH, however, was listening on:

0.0.0.0:22

and:

[::]:22

Therefore, SSH was available through the system's network interfaces.

## Security Assessment

The investigation did not identify evidence of an active compromise.

However, SSH represents the primary network-exposed service identified
during the investigation.

The combination of network-wide SSH listening, password authentication,
and inactive host firewall controls should be considered when assessing
the security posture of a production Linux system.

In a production environment, an analyst would investigate whether SSH
access is required, whether access should be restricted by network
source, whether key-based authentication should be preferred, and
whether host firewall rules should be implemented.

## Skills Demonstrated
- Linux network investigation
- IP addressing
- Routing analysis
- Network interface investigation
- Port and service identification
- ss
- ip
- Nmap scanning
- Service version detection
- SSH security assessment
- Firewall assessment
- Network exposure analysis
- Security documentation
- Evidence-based assessment

## Conclusion

This project demonstrates a basic workflow for investigating the
network security posture of a Linux system.

The investigation showed how security analysts can combine Linux
networking commands, socket information, Nmap scanning, service
identification, SSH configuration analysis, and firewall checks to
understand what services are exposed and how the system communicates
with its network.

The exercise also demonstrates that an open port is not automatically
evidence of malicious activity. Security findings must be interpreted
within the system's configuration, network architecture, and
investigation context.
