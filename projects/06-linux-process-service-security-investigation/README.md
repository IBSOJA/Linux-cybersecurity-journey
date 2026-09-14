# Project 06 — Linux Process & Service Security Investigation

## Overview

This project investigates Linux processes, process relationships, system services, and security controls on an Ubuntu Linux system.

The investigation focuses on identifying running processes, understanding parent-child process relationships, analyzing resource usage, examining important system services, reviewing enabled services, checking security controls, and identifying failed services.

The goal is to develop practical skills for determining whether processes and services are expected, properly managed, and appropriately protected from a security perspective.

## Objectives

- Identify the operating system, kernel, architecture, and current user.
- Examine running Linux processes and understand their PIDs and parent processes.
- Analyze process hierarchies using `pstree`.
- Monitor system resource usage using `top`.
- Investigate important processes individually using `ps`.
- Examine the status of critical system services using `systemctl`.
- Distinguish between running, enabled, inactive, and failed services.
- Review enabled and currently running services.
- Investigate security controls including UFW, nftables, and AppArmor.
- Identify failed system services.
- Correlate processes, services, security controls, and system behavior to identify potential security concerns.
- Document findings using evidence collected during the investigation.

## Environment

- Operating System: Ubuntu 24.04.3 LTS
- Kernel: Linux 7.0.0-30-generic
- Architecture: x86-64
- Virtualization: Oracle VirtualBox
- Investigation User: `vboxuser`

## Tools Used

### Process Investigation

- `ps` — Display current process information.
- `pstree` — Display processes in a hierarchical parent-child structure.
- `top` — Monitor processes and system resource usage in real time.

### Service Investigation

- `systemctl` — Inspect service status, running services, enabled services, and failed services.

### Security Controls

- `ufw` — Check the status of the Ubuntu firewall.
- `nft` — Inspect the nftables ruleset.
- `aa-status` — Inspect AppArmor status, loaded profiles, and enforcement modes.

### General System Investigation

- `hostname` — Identify the system hostname.
- `hostnamectl` — Display system and operating system information.
- `uname` — Display kernel and system architecture information.
- `whoami` — Identify the current user.
- `id` — Display user identity and group membership.

## Investigation Methodology

The investigation was performed in the following stages:

1. **System Identification**
   - Identified the operating system, hostname, kernel version, architecture, virtualization platform, and current user.

2. **Process Enumeration**
   - Reviewed the current terminal process and system-wide processes using `ps`.
   - Examined process IDs and parent process IDs.

3. **Process Hierarchy Analysis**
   - Used `pstree` to understand parent-child relationships between system processes, services, and the user session.

4. **Resource Analysis**
   - Used `top` to examine CPU usage, memory usage, system load, running processes, sleeping processes, and zombie processes.

5. **Individual Process Investigation**
   - Investigated important processes such as `sshd`, `cupsd`, and `gnome-shell` using their PIDs.

6. **Service Investigation**
   - Examined the status of SSH and CUPS services.
   - Reviewed currently running services and enabled service units.
   - Compared service states such as running, enabled, inactive, and failed.

7. **Security Control Investigation**
   - Checked UFW firewall status.
   - Inspected the nftables ruleset.
   - Examined AppArmor profiles and enforcement status.

8. **Failure Detection**
   - Checked whether systemd reported any failed service units.

9. **Security Assessment**
   - Correlated process ownership, PIDs, parent processes, service states, security controls, and observed system behavior.
   - Distinguished normal system activity from potential security concerns.

## System Identification

The system was identified before beginning the process and service investigation.

### Findings

- Hostname: `Ubuntu`
- Operating System: Ubuntu 24.04.3 LTS
- Kernel: Linux 7.0.0-30-generic
- Architecture: x86-64
- Virtualization: Oracle VirtualBox
- Current user: `vboxuser`
- User ID: `1000`
- Primary group: `vboxuser`
- Additional groups include `sudo` and `vboxsf`

### Security Interpretation

The current user is a member of the `sudo` group, which provides administrative privileges through `sudo`. This is a privilege-related finding and is not, by itself, evidence of compromise or malicious activity.

## Process Investigation

### Process Snapshot

The `ps` command was used to examine processes associated with the current terminal session.

The output showed:

- `bash` as the active shell process.
- `ps` as the process used to perform the process snapshot.

The `ps` command provides a snapshot of processes rather than continuous monitoring.

### System-Wide Process Enumeration

The `ps -ef` command was used to examine processes across the system.

The investigation identified expected system and desktop processes, including:

- `systemd` as PID 1.
- `systemd-journald` for system logging.
- `systemd-udevd` for device management.
- `systemd-resolved` for DNS resolution.
- `NetworkManager` for network management.
- `cron` for scheduled tasks.
- `rsyslogd` for system logging.
- `sshd` for SSH services.
- `cupsd` for printing services.
- GNOME desktop processes.
- VirtualBox guest-related processes.
- User session processes.

Kernel threads were also observed in the process listing. These were identified by their square-bracket notation and were not treated as suspicious based solely on their presence.

### Security Interpretation

The process listing showed a mixture of root-owned system processes and user-owned desktop/session processes.

Process ownership alone was not treated as evidence of malicious activity. Processes were evaluated using additional context such as their names, parent processes, purpose, and relationship to expected system services.

## Process Hierarchy Analysis

The `pstree -p` command was used to examine parent-child relationships between processes.

### Key Observations

- `systemd` (PID 1) was the top-level process and the parent of many system services.
- System services such as `sshd`, `cupsd`, `NetworkManager`, `rsyslogd`, and other daemons were associated with the system-level process hierarchy.
- A separate user-level `systemd` process managed the graphical user session.
- The GNOME desktop environment and related applications were descendants of the user session.
- The terminal process hierarchy included:
  - `gnome-terminal-server` → `bash` → investigation command
- The process tree showed expected relationships between the operating system, services, desktop session, and terminal processes.

### Security Interpretation

Process hierarchy provides useful context when investigating suspicious activity.

A process with an unexpected parent, unusual execution path, or unexpected relationship to a system service could warrant further investigation. In this investigation, the observed process relationships were consistent with the Ubuntu desktop environment and the activities being performed.

## Resource Analysis

The `top` command was used to examine real-time system resource usage and identify processes consuming significant system resources.

### Findings

At the time of the investigation:

- System uptime was approximately 7 hours and 47 minutes.
- The load averages were `0.23`, `0.12`, and `0.09`.
- There were 233 total tasks.
- 1 task was running and 232 were sleeping.
- There were 0 stopped processes.
- There were 0 zombie processes.
- CPU usage was approximately 98.6% idle.
- Approximately 2.78 GiB of memory was available.
- No swap space was configured or in use.

The most active process observed was `gnome-shell` (PID 2247), which was using approximately 4.6% CPU and 9.9% memory at the time of observation.

### Security Interpretation

The system showed low resource utilization during the investigation. CPU utilization was predominantly idle, and no zombie processes were observed.

`gnome-shell` was the highest visible resource consumer during the observation, but its resource usage was consistent with the graphical desktop environment and did not, by itself, indicate suspicious activity.

Resource usage should be interpreted in context. An unusually high CPU or memory consumer may warrant further investigation, but high resource usage alone does not prove malicious activity.

## Individual Process Investigation

Specific processes were examined using `ps -fp <PID>` to determine their ownership, parent process, and command execution details.

### SSH Daemon

The SSH daemon was identified as:

- PID: `4260`
- PPID: `1`
- User: `root`
- Command: `/usr/sbin/sshd -D [listener] 0 of 10-100 startups`

The process was associated with the system-level process hierarchy and was functioning as the SSH listener.

### CUPS Scheduler

The CUPS printing daemon was identified as:

- PID: `5187`
- PPID: `1`
- User: `root`
- Command: `/usr/sbin/cupsd -l`

The process was associated with the CUPS printing service and was running under the system-level process hierarchy.

### GNOME Shell

The GNOME Shell process was identified as:

- PID: `2247`
- PPID: `1989`
- User: `vboxuser`
- Command: `/usr/bin/gnome-shell`

The process was associated with the user's graphical desktop session and was a child of the user-level systemd process.

### Security Interpretation

The investigated processes had expected executable paths, ownership, and parent-child relationships for their respective functions.

The SSH and CUPS daemons were root-owned and associated with PID 1, consistent with system-managed services. GNOME Shell was user-owned and associated with the user's graphical session.

No suspicious process behavior was identified from these process-level checks. However, process inspection alone cannot establish that a system is completely free from compromise; additional evidence such as service configuration, network activity, and logs should also be considered.

## Service Investigation

System services were investigated using `systemctl` to determine their current state, startup configuration, associated processes, and service-management behavior.

### SSH Service

The SSH service was examined using:

```bash
systemctl status ssh --no-pager
```
Findings:

- Service: ssh.service
- Description: OpenBSD Secure Shell server
- State: inactive (dead)
- Service unit: disabled
- Triggered by: ssh.socket

The SSH socket was then examined using:

systemctl status ssh.socket --no-pager

Findings:

- Socket: ssh.socket
- State: active (listening)
- Socket unit: enabled
- Listening address: 0.0.0.0:22
- Listening address: [::]:22

The socket configuration was examined using:

systemctl cat ssh.socket

The configuration included:

ListenStream=0.0.0.0:22
ListenStream=[::]:22
Accept=no

This demonstrated that SSH was configured using systemd socket activation. The socket remains active and listening for incoming connections while the ssh.service process is started when triggered by the socket.

The listening socket was correlated with network state using:

sudo ss -lntp | grep ':22'

The output showed that TCP port 22 was listening on both IPv4 and IPv6 and was associated with systemd PID 1.

The earlier process investigation identified an sshd process associated with the SSH service. The difference in observed process/service state demonstrates that system state can change over the course of an investigation and that service state should be correlated with current process and socket information.

### CUPS Service

The CUPS service was examined using:

```bash
systemctl status cups --no-pager
```

Findings:

- Service: `cups.service`
- Description: CUPS Scheduler
- State: `active (running)`
- Main PID: `5187`
- Triggered by: `cups.path` and `cups.socket`
- Service unit: `enabled`
- Status indicated that the scheduler was running.

The service's main process, PID `5187`, matched the `cupsd` process identified during individual process investigation.

### Security Interpretation

Both SSH and CUPS were actively running and had corresponding processes identified during the process investigation.

The SSH service demonstrated an important distinction between service state and startup configuration: a service can be currently running even when its traditional service unit is marked as disabled, particularly when socket activation is involved.

Service state should therefore be interpreted together with activation mechanisms, associated processes, and configuration rather than relying on the `enabled` or `disabled` label alone.

## Running and Enabled Services

The investigation compared currently running services with services configured as enabled using `systemctl`.

### Currently Running Services

The following command was used to identify services currently in the `running` state:

```bash
systemctl list-units --type=service --state=running --no-pager
```

The system reported 30 running service units.

Examples of running services included:

- `NetworkManager.service` — network management.
- `systemd-journald.service` — system logging.
- `systemd-resolved.service` — DNS resolution.
- `rsyslog.service` — system logging.
- `cron.service` — scheduled task management.
- `cups.service` — printing service.
- `cups-browsed.service` — printer discovery and browsing.
- `snapd.service` — Snap package management.
- `unattended-upgrades.service` — automatic security updates.
- `vboxadd-service.service` — VirtualBox guest integration.
- `user@1000.service` — user session management.

Other services were also running as part of the Ubuntu desktop environment and system configuration.

### Enabled Services

The following command was used to identify service units configured as enabled:

```bash
systemctl list-unit-files --type=service --state=enabled --no-pager
```

The enabled service list contained system and application services including:

- `apparmor.service`
- `avahi-daemon.service`
- `bluetooth.service`
- `cron.service`
- `cups.service`
- `gnome-remote-desktop.service`
- `NetworkManager.service`
- `openvpn.service`
- `rsyslog.service`
- `snapd.service`
- `sssd.service`
- `systemd-resolved.service`
- `ufw.service`
- `unattended-upgrades.service`
- `vboxadd.service`
- `vboxadd-service.service`
- `wpa_supplicant.service`

### Security Interpretation

The running-service list and enabled-service list were not identical.

This demonstrates that an enabled service is not necessarily running at the time of investigation. Conversely, a service can be active through mechanisms such as socket activation even when its traditional service unit is marked as disabled.

Service investigation therefore requires examining both current runtime state and startup configuration.

The presence of a service in the enabled list does not by itself indicate that the service is currently active or that it is malicious. Each service should be evaluated according to its purpose, current state, configuration, and relationship to other system components.

## Security Control Investigation

Security controls were investigated to determine whether firewall filtering and mandatory access control mechanisms were active and enforcing protection.

### UFW Firewall

The UFW service unit was first examined using:

```bash
systemctl status ufw --no-pager
```

The service unit was shown as:

- Loaded: enabled
- State: `active (exited)`

The actual UFW firewall status was then checked using:

```bash
sudo ufw status verbose
```

The result was:

```text
Status: inactive
```

### Security Interpretation

Although the UFW service unit was enabled and had an `active (exited)` state, UFW itself reported that the firewall was inactive.

This demonstrates that the state of a systemd service unit should not automatically be interpreted as proof that a security control is actively enforcing protection.

The UFW result indicates that UFW was not actively filtering traffic at the time of investigation.

### nftables

The nftables ruleset was examined using:

```bash
sudo nft list ruleset
```

No rules were displayed during the investigation.

### Security Interpretation

No active nftables rules were displayed by the command at the time of investigation.

This result was considered together with the UFW status rather than being interpreted in isolation. The absence of displayed nftables rules does not, by itself, prove that the entire system is without network security controls.

### AppArmor

AppArmor status was examined using:

```bash
sudo aa-status
```

The investigation found:

- AppArmor kernel module: loaded
- Profiles loaded: `159`
- Profiles in enforce mode: `62`
- Profiles in complain mode: `5`
- Profiles in unconfined mode: `92`
- Processes with enforced AppArmor profiles included `cups-browsed`, `cupsd`, and `rsyslogd`

Five profiles were in complain mode, including the SSSD profile. The SSSD service itself was found to be inactive during the service investigation.

### Security Interpretation

AppArmor was loaded and actively enforcing a number of security profiles.

The presence of enforced profiles for services such as `cupsd`, `cups-browsed`, and `rsyslogd` provides an additional security control by restricting the behavior of those applications according to their AppArmor profiles.

Profiles in complain mode record policy violations without enforcing the restrictions in the same way as enforce mode. Therefore, complain-mode profiles should not be treated as providing the same level of active restriction as enforcing profiles.

The AppArmor findings represent an active security control on the system, even though UFW was inactive and no nftables rules were displayed.

## Failed Services

The system was checked for services that systemd considered to be in a failed state.

The following command was used:

```bash
systemctl --failed --no-pager
```

The command reported:

```text
0 loaded units listed.
```

A second check was performed using:

```bash
systemctl list-units --type=service --state=failed --no-pager
```

This command also reported:

```text
0 loaded units listed.
```

### Security Interpretation

Systemd reported no failed service units at the time of the investigation.

The absence of failed services is a positive system-health indicator because no service failures were being reported by systemd during the investigation.

However, the absence of failed systemd units does not prove that every application or security control on the system is functioning correctly. Service status should be considered together with process information, configuration, logs, network activity, and other security evidence.

## Overall Security Assessment

The process, service, and security-control investigations were correlated to assess the overall security posture of the Ubuntu system.

### Positive Security Findings

The investigation identified several positive security indicators:

- Important system processes had expected ownership, executable paths, and parent-child relationships.
- The SSH and CUPS processes were associated with their expected system-managed services.
- The system showed low CPU utilization and available memory during the investigation.
- No zombie processes were observed.
- Systemd reported no failed service units.
- AppArmor was loaded with `62` profiles in enforce mode.
- AppArmor enforcement was observed for processes including `cupsd`, `cups-browsed`, and `rsyslogd`.

### Security Concerns and Configuration Observations

Several configuration conditions were identified that may increase security risk or require further review:

- UFW reported `Status: inactive`.
- No nftables rules were displayed during the investigation.
- SSH was configured for systemd socket activation, with `ssh.socket` actively listening on TCP port 22 on all IPv4 and IPv6 interfaces.
- UFW was inactive during the investigation.
- The VM was using a `10.0.2.0/24` network consistent with a VirtualBox NAT environment.
- SSH authentication methods were not the primary focus of this project and were not used as the basis for the service-state assessment.
- Multiple services were configured as enabled. Services that are not required should be reviewed and disabled where appropriate.
- Several AppArmor profiles were in complain mode rather than enforce mode.

These findings represent security considerations and configuration observations. They do not, by themselves, demonstrate that the system has been compromised.

### Risk Assessment

The most significant observation from this investigation was the lack of active UFW filtering combined with the absence of displayed nftables rules. SSH was listening on TCP port 22 on all IPv4 and IPv6 interfaces through systemd socket activation, so SSH access should receive particular attention during a security hardening review. The VM's VirtualBox NAT configuration provides an additional network boundary, but actual reachability should be assessed separately.

The presence of AppArmor enforcement provides an additional layer of protection for supported applications and services.

Overall, the investigation did not identify direct evidence of malicious processes, failed system services, or abnormal resource consumption. However, the system would benefit from reviewing unnecessary listening services, firewall configuration, SSH authentication methods, and AppArmor profiles in complain mode.

### Investigation Limitations

This investigation was based on a point-in-time examination of the system.

The checks performed cannot prove that the system is completely free from compromise. A more comprehensive investigation would also examine authentication logs, persistent mechanisms, scheduled tasks, file integrity, network connections over time, installed software, and other host-based security evidence.

## Conclusion

This investigation provided a practical examination of Linux processes, process hierarchies, system services, resource usage, and host security controls.

The observed processes and service relationships were consistent with the expected Ubuntu desktop environment and the services intentionally configured on the system. No failed systemd services, abnormal resource consumption, or clearly suspicious process relationships were identified during the investigation.

The investigation also identified security configuration considerations. UFW was inactive, no nftables rules were displayed, and SSH was listening on TCP port 22 on all IPv4 and IPv6 interfaces through systemd socket activation. The VM's VirtualBox NAT configuration provides an additional network boundary, but actual external reachability was not established during this investigation. AppArmor provided an additional layer of protection through enforced profiles, although several profiles remained in complain mode.

The investigation demonstrates that Linux security assessment requires more than identifying individual processes or services. Effective analysis requires correlating process information, service state, startup configuration, security controls, network exposure, and other available evidence.

No direct evidence of compromise was identified during this investigation. However, the findings provide areas for further hardening and investigation, particularly firewall configuration, SSH authentication, unnecessary services, and AppArmor enforcement.
