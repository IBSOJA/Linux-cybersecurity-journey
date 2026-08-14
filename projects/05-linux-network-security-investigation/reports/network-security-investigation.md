# Linux Network Security Investigation Report

## 1. Executive Summary

This investigation assessed the network security posture of an Ubuntu
24.04.3 LTS laboratory system.

The investigation examined network interfaces, routing information,
neighboring devices, listening services, active connections, exposed
ports, service versions, SSH configuration, and host firewall status.

The primary network-exposed service identified was SSH on TCP port 22.

No evidence of active compromise was identified during the investigation.
However, SSH was found to be listening on all IPv4 and IPv6 interfaces,
password authentication was enabled, and UFW was inactive.

These findings represent security configuration considerations rather
than evidence of malicious activity.

## 2. Investigation Objectives

The objectives were to:

- Identify network interfaces and IP addresses.
- Examine routing and gateway information.
- Identify listening services.
- Investigate active network connections.
- Identify exposed ports.
- Perform service version detection.
- Assess SSH security configuration.
- Examine firewall status.
- Distinguish locally restricted services from network-exposed services.
- Assess the overall network security posture.

## 3. Environment

| Item | Value |
|---|---|
| Operating System | Ubuntu 24.04.3 LTS |
| Hostname | Ubuntu |
| Network Interface | enp0s3 |
| IPv4 Address | 10.0.2.15/24 |
| Default Gateway | 10.0.2.2 |
| Network Type | VirtualBox NAT |
| SSH Service | OpenSSH |
| SSH Port | 22 |

## 4. Network Configuration Findings

The primary network interface was identified as:

`enp0s3`

The system was assigned:

`10.0.2.15/24`

The routing table identified:

`10.0.2.0/24` as the local network.

The default gateway was:

`10.0.2.2`

The neighbor table also identified the gateway at `10.0.2.2`.

These findings are consistent with the VirtualBox NAT network used by
the laboratory system.

## 5. Listening Service Investigation

The socket investigation identified several listening services.

### SSH

SSH was listening on:

```text
0.0.0.0:22
[::]:22

This means SSH was bound to all IPv4 and IPv6 interfaces.

CUPS

CUPS was listening on:

127.0.0.1:631
[::1]:631

These are loopback addresses, meaning CUPS was restricted to the local
system.

DNS Resolver

The system resolver was listening on local addresses including:

127.0.0.54:53
127.0.0.53:53

The socket information associated these services with
systemd-resolved.

## 6. Active Connection Investigation

The active connection investigation identified a UDP DHCP-related
connection:

Local:  10.0.2.15:68
Remote: 10.0.2.2:67
Process: NetworkManager

No suspicious established TCP connections were observed during the
investigation.

The observed connection was consistent with normal network
configuration activity.

## 7. Nmap Port Investigation

A local scan of 127.0.0.1 identified:

22/tcp   open   ssh
631/tcp  open   ipp

A scan of the system's network address, 10.0.2.15, identified:

22/tcp   open   ssh

The difference between the two scans demonstrated an important
security concept:

A service may be listening locally without being exposed through the
network interface.

CUPS was available through the loopback interface, while SSH was
network reachable.

## 8. Service Version Detection

Nmap service detection identified the SSH service as:

OpenSSH 9.6p1 Ubuntu 3ubuntu13.18

The service was identified as running on Linux.

Service version information is useful during security assessments
because analysts can compare identified software versions against
known vulnerabilities, organizational requirements, and patching
standards.

Version identification alone does not demonstrate that a service is
vulnerable.

## 9. SSH Security Configuration

The effective SSH configuration showed:

PermitRootLogin without-password
PubkeyAuthentication yes
PasswordAuthentication yes
KbdInteractiveAuthentication no

The most significant observation was:

PasswordAuthentication yes

Password authentication was enabled while public-key authentication
was also available.

SSH was listening on:

0.0.0.0:22
[::]:22

Therefore, SSH was available through the system's network interfaces.

A successful SSH connection to:

10.0.2.15:22

confirmed that the service was reachable through the VM's network
address.

## 10. UDP Investigation

A UDP scan of the top 20 ports on 10.0.2.15 was performed.

All scanned ports were reported as:

closed

No open UDP service was identified by this particular scan.

This result should be interpreted within the scope of the scan because
only the top 20 UDP ports were examined.

## 11. Firewall Investigation

UFW reported:

Status: inactive

The nftables ruleset did not display any active rules during the
investigation.

Therefore, the checks performed did not identify active UFW or
displayed nftables filtering rules.

This represents a security configuration consideration rather than
evidence of compromise.

## 12. Network Exposure Assessment

The investigation identified SSH as the primary network-exposed
service.

The exposure can be summarized as follows:

Service	Port	Binding	Exposure
SSH	22/tcp	0.0.0.0 / [::]	Network exposed
CUPS	631/tcp	127.0.0.1 / [::1]	Local only
DNS resolver	53/tcp/udp	127.0.0.53 / 127.0.0.54	Local only

The distinction between listening and network exposure is an important
security principle.

An open listening socket does not automatically mean that the service
is accessible from an external network.

## 13. Security Findings
Finding 1 — SSH Network Exposure

Severity: Configuration consideration

SSH was listening on all IPv4 and IPv6 interfaces.

This increases the network exposure of the SSH service compared with a
configuration that restricts SSH to specific trusted interfaces or
networks.

Finding 2 — Password Authentication Enabled

Severity: Configuration consideration

Password authentication was enabled for SSH.

Password-based authentication can increase exposure to password
guessing and brute-force attacks when SSH is reachable from untrusted
networks.

Finding 3 — Host Firewall Controls

Severity: Configuration consideration

UFW was inactive and no active nftables rules were displayed during
the investigation.

In a production environment, appropriate host-based filtering should
be considered where required by the organization's security policy.

## 14. Compromise Assessment

No evidence of active compromise was identified during this
investigation.

The investigation observed expected laboratory services and network
activity.

The presence of an open SSH port, enabled password authentication, or
an inactive firewall does not independently prove that the system has
been compromised.

Additional evidence such as suspicious processes, unauthorized
accounts, malicious files, unexpected connections, or suspicious
authentication events would be required to support a compromise
determination.

## 15. Recommended Production Considerations

For a production Linux system, an administrator or security analyst
could consider:

Restricting SSH access to trusted networks where possible.
Using strong SSH key-based authentication.
Reviewing whether password authentication is required.
Applying appropriate host firewall rules.
Keeping OpenSSH and the operating system patched.
Monitoring SSH authentication activity.
Reviewing exposed services regularly.
Removing or disabling unnecessary network services.

These recommendations should be applied according to the system's
operational requirements and security policy.

## 16. Evidence Summary

The investigation used the following evidence sources:

ip link
ip addr
ip route
ip neigh
ss -tulpn
ss -tunap
ss -ltnp
nmap 127.0.0.1
nmap 10.0.2.15
nmap -sV 10.0.2.15
sudo nmap -sU --top-ports 20 10.0.2.15
sudo sshd -T
sudo ufw status verbose
sudo nft list ruleset
resolvectl status
ssh -v vboxuser@10.0.2.15

## 17. Conclusion

The investigation demonstrated a practical workflow for assessing the
network security posture of a Linux system.

The primary network-exposed service was SSH on TCP port 22.

CUPS was identified as a locally restricted service, demonstrating the
difference between a listening service and a network-exposed service.

The system did not show evidence of active compromise during the
investigation.

However, the combination of network-wide SSH exposure, enabled
password authentication, and inactive host firewall controls represents
an important security consideration for a production environment.

The investigation demonstrates that network security assessments
should combine technical evidence with contextual analysis rather than
treating individual open ports or configuration settings as proof of
malicious activity.
