# Project 07 — Linux Security Hardening Lab

## Overview

This project focuses on identifying and reducing security risks on an Ubuntu Linux system through a controlled security hardening process.

The project follows a **Before → Harden → After** methodology:

1. Establish a security baseline.
2. Identify hardening opportunities.
3. Apply carefully selected security controls.
4. Verify that the system remains functional.
5. Compare the post-hardening state with the baseline.
6. Document the changes and security improvements.

No hardening changes were made during the baseline assessment.

---

## Objectives

- Assess local users and privileges.
- Identify accounts with administrative privileges.
- Review sensitive file permissions.
- Assess SSH configuration and exposure.
- Review running and enabled services.
- Identify unnecessary or potentially unnecessary services.
- Assess firewall configuration.
- Review available system updates.
- Assess AppArmor protection.
- Review kernel security controls.
- Apply appropriate Linux security hardening measures.
- Verify the system after hardening.
- Maintain evidence of the before-and-after security state.

---

## Environment

### Baseline Environment

- **OS:** Ubuntu 24.04.3 LTS
- **Kernel:** Linux 7.0.0-30-generic
- **Architecture:** x86-64
- **Virtualization:** Oracle VirtualBox
- **Primary User:** vboxuser (UID 1000)
- **Primary Group:** vboxuser
- **Administrative Group:** sudo

The baseline environment represents the system state before security hardening was performed.

### Post-Update Environment

Following the system update and reboot:

- **OS:** Ubuntu 24.04.5 LTS
- **Kernel:** Linux 7.0.0-31-generic
- **Architecture:** x86-64
- **Virtualization:** Oracle VirtualBox
- **Primary User:** vboxuser (UID 1000)

The newer kernel was installed as part of the system update phase before the remaining security-hardening activities.

---

# Phase 1 — Security Baseline

The baseline was collected before making any security configuration changes.

This provides a reference point for measuring the effect of later hardening actions.

## 1. User and Privilege Assessment

Only one account was identified with UID 0:

```text
root:0
```

The `vboxuser` account is a member of the `sudo` group and has unrestricted sudo capability on the system.

The following accounts were identified:

- vboxuser
- bob
- A
- Alice

`bob`, `A`, and `Alice` have locked passwords and have never logged in according to `lastlog`.

No accounts were removed or modified during the baseline assessment.

Evidence:

`evidence/baseline-users.txt`

---

## 2. SSH Baseline

The SSH service used systemd socket activation.

### Baseline Service State

- `ssh.socket` was active and enabled.
- `ssh.service` was triggered by the SSH socket.
- SSH listened on:
  - `0.0.0.0:22`
  - `[::]:22`
- `/run/sshd` was initially absent while `ssh.service` was inactive.
- Starting `ssh.service` created the required runtime directory.
- `sshd -t` and `sshd -T` then completed successfully.

### Baseline Effective SSH Configuration

The effective SSH configuration before hardening included:

| Setting | Baseline Value |
| ------- | -------------- |
| Port | 22 |
| MaxAuthTries | 6 |
| MaxSessions | 10 |
| PermitRootLogin | without-password |
| PubkeyAuthentication | yes |
| PasswordAuthentication | yes |
| X11Forwarding | yes |

No SSH configuration changes were made during the baseline assessment.


---

## 3. File Permission Assessment

The main account database files were assessed.

| File           | Permissions | Owner       |
| -------------- | ----------: | ----------- |
| `/etc/passwd`  |         644 | root:root   |
| `/etc/group`   |         644 | root:root   |
| `/etc/shadow`  |         640 | root:shadow |
| `/etc/gshadow` |         640 | root:shadow |

The sensitive files `/etc/shadow` and `/etc/gshadow` were not world-readable.

Existing home directories were configured with restrictive permissions:

- `/home/vboxuser`: 750
- `/home/bob`: 750

The accounts `A` and `Alice` have home directory paths recorded in `/etc/passwd`, but those directories do not currently exist.

No permission changes were made during the baseline.

Evidence:

`evidence/baseline-permissions.txt`

---

## 4. Service Assessment

The baseline identified:

- 30 running service units
- 57 enabled service unit files
- 0 failed systemd services

AppArmor was active and enabled.

Several services were identified for review during the hardening phase, including:

- GNOME Remote Desktop
- Avahi
- CUPS
- CUPS Browsed
- ModemManager

These services were not disabled during the baseline because service removal or disabling should only occur after determining whether the functionality is required.

VirtualBox integration services were retained because the system is running inside VirtualBox.

Evidence:

`evidence/baseline-services.txt`

---

## 5. Firewall Assessment

The UFW firewall was found to be:

```text
Status: inactive
```

The UFW systemd unit was:

```text
active (exited)
enabled
```

However, the active firewall policy was not enforcing restrictive filtering.

The observed IPv4 and IPv6 firewall chains had permissive ACCEPT policies, and no restrictive nftables filtering rules were observed.

The nftables filter tables were empty:

```text
table ip filter {
}

table ip6 filter {
}
```

This represents a significant hardening opportunity.

No firewall rules were changed during the baseline.

Evidence:

`evidence/baseline-firewall.txt`

---

## 6. System Update Assessment

The configured Ubuntu repositories were reachable successfully.

The following repositories were available:

- noble
- noble-updates
- noble-security
- noble-backports

At the time of the baseline:

**151 package upgrades were available.**

The available updates included security-relevant components such as:

- Linux kernel packages
- AppArmor
- Linux firmware
- networking components
- other core system packages

The running kernel was:

```text
7.0.0-30-generic
```

A newer kernel package was available:

```text
7.0.0-31-generic
```

The `unattended-upgrades` service was active and enabled, and its configuration file was present.

No package upgrades were installed during the baseline.

Evidence:

`evidence/baseline-updates.txt`

---

## 7. AppArmor Assessment

AppArmor was loaded and enabled.

Baseline results:

| AppArmor State  | Count |
| --------------- | ----: |
| Profiles loaded |   159 |
| Enforce         |    62 |
| Complain        |     5 |
| Prompt          |     0 |
| Kill            |     0 |
| Unconfined      |    92 |

Three observed processes were running under enforcing AppArmor profiles:

- cupsd
- cups-browsed
- rsyslogd

No AppArmor profiles were modified during the baseline.

The presence of unconfined profiles was not treated as a security failure by itself because profile enforcement should be evaluated according to the application and its requirements.

Evidence:

`evidence/baseline-apparmor.txt`

---

## 8. Kernel Security Assessment

The following security-related kernel controls were inspected:

| Control                     | Value | Assessment                         |
| --------------------------- | ----: | ---------------------------------- |
| `kernel.randomize_va_space` |     2 | ASLR enabled                       |
| `kernel.dmesg_restrict`     |     1 | dmesg access restricted            |
| `kernel.kptr_restrict`      |     1 | Kernel pointer exposure restricted |
| `fs.protected_hardlinks`    |     1 | Protection enabled                 |
| `fs.protected_symlinks`     |     1 | Protection enabled                 |

These controls already had protective values.

No changes were required during the baseline.

Evidence:

`evidence/baseline-kernel-security.txt`

---

# Baseline Findings Summary

| Area            | Baseline Finding                         | Hardening Consideration                 |
| --------------- | ---------------------------------------- | --------------------------------------- |
| Users           | Only root has UID 0                      | Review unnecessary accounts             |
| Sudo            | vboxuser has unrestricted sudo           | Preserve required administrative access |
| SSH             | Socket active on port 22                 | Review SSH configuration safely         |
| Permissions     | Sensitive files appropriately restricted | Maintain secure permissions             |
| Services        | 30 running, 57 enabled                   | Review unnecessary services             |
| Failed Services | 0                                        | Maintain healthy service state          |
| Firewall        | UFW inactive                             | Configure restrictive firewall policy   |
| Updates         | 151 packages pending                     | Apply available updates                 |
| AppArmor        | 62 profiles enforcing                    | Preserve working profiles               |
| Kernel Controls | Protective values present                | Retain secure settings                  |

---

# Evidence Directory

Baseline evidence is stored in the `evidence/` directory:

```text
evidence/
├── baseline-apparmor.txt
├── baseline-firewall.txt
├── baseline-kernel-security.txt
├── baseline-permissions.txt
├── baseline-services.txt
├── baseline-ssh.txt
├── baseline-updates.txt
└── baseline-users.txt
```

---

# Hardening Plan

The hardening phase will be performed carefully and incrementally.

Planned areas of investigation include:

1. Apply available system updates.
2. Review SSH configuration before changing authentication settings.
3. Harden SSH without causing loss of access.
4. Configure a restrictive firewall policy.
5. Review unnecessary network-facing services.
6. Disable services only when their functionality is not required.
7. Preserve required VirtualBox functionality.
8. Re-check AppArmor after hardening.
9. Verify kernel security controls remain protected.
10. Re-run the baseline checks after changes.

Hardening decisions will be based on evidence rather than blindly applying generic security configurations.

---

# Before-and-After Methodology

For each major hardening area, the project will record:

```text
BEFORE
  ↓
Security assessment
  ↓
Hardening change
  ↓
Verification
  ↓
AFTER
```

The final report will document:

- What was changed
- Why it was changed
- The security benefit
- Any operational impact
- How the change was verified
- Whether the system remained functional

---

# Safety and Rollback

Security hardening can accidentally prevent legitimate access or break required functionality.

Therefore:

- SSH changes will be validated before being applied.
- Firewall rules will be configured before enabling enforcement.
- Required services will not be disabled blindly.
- Existing configurations will be inspected before modification.
- Changes will be verified after each major stage.
- The VirtualBox console provides a local recovery path if network access is interrupted.

No destructive changes will be made simply for the purpose of demonstrating hardening.

---

# Project Status

## Completed

- [x] Project directory created
- [x] Baseline system assessment
- [x] User and privilege assessment
- [x] SSH assessment
- [x] Permission assessment
- [x] Service assessment
- [x] Firewall assessment
- [x] Update assessment
- [x] AppArmor assessment
- [x] Kernel security assessment
- [x] Baseline evidence documentation

## Project Status

### Completed

- [x] Baseline system assessment
- [x] User and privilege assessment
- [x] SSH assessment and hardening
- [x] Permission assessment
- [x] Service assessment
- [x] Firewall hardening
- [x] System updates and reboot
- [x] AppArmor assessment
- [x] Kernel security assessment
- [x] Avahi and CUPS Browsed hardening
- [x] GNOME Remote Desktop hardening
- [x] ModemManager hardening
- [x] Kerneloops hardening
- [x] CUPS hardening
- [x] Post-hardening service verification
- [x] Post-hardening network verification
- [x] Evidence documentation
- [x] Before-and-after comparison
- [x] Final README review

### Remaining

- [ ] Final Git commit
- [ ] GitHub push

---

# Phase 2 — Security Hardening

The hardening phase applied selected security controls based on the baseline findings. Each change was tested after implementation to verify that required system functionality remained available.

## 1. System Updates

The system was updated before the main hardening activities.

- Installed kernel: `7.0.0-31-generic`
- System was rebooted after the kernel update.
- The system subsequently reported Ubuntu 24.04.5 LTS.
- Remaining package updates were reviewed and documented.
- Eleven packages remained upgradable and were deferred/phased at the time of verification.

Evidence:

- `evidence/after-updates.txt`

## 2. Firewall Hardening

UFW was initially inactive with permissive default policies.

The firewall was configured as follows:

- Default incoming traffic: **deny**
- Default outgoing traffic: **allow**
- Default routed traffic: **deny**
- SSH: **allowed on TCP port 22**
- UFW logging: **low**
- UFW: **enabled**

The firewall was verified after enabling it.

Evidence:

- `evidence/baseline-firewall.txt`
- `evidence/firewall-hardening.txt`

## 3. SSH Hardening

SSH authentication was hardened using public-key authentication.

An Ed25519 key pair was generated and the public key was added to the user's `authorized_keys` file.

A dedicated SSH hardening drop-in was created:

`/etc/ssh/sshd_config.d/99-hardening.conf`

The following settings were applied:

- `PasswordAuthentication no`
- `PermitRootLogin no`
- `X11Forwarding no`
- `MaxAuthTries 3`

Configuration syntax was tested with `sshd -t`, and the effective configuration was verified with `sshd -T`.

Key-based authentication succeeded, while a password-only authentication test was rejected.

Evidence:

- `evidence/ssh-baseline.txt`
- `evidence/ssh-hardening.txt`

## 4. Avahi and CUPS Browsed Hardening

Avahi and CUPS Browsed were reviewed because they provided network discovery and remote-printer functionality that was not required for this system.

`cups-browsed` was stopped and disabled first because it could keep Avahi active through its dependency relationship.

The following services were then disabled:

- `cups-browsed.service`
- `avahi-daemon.service`
- `avahi-daemon.socket`

After the change:

- Avahi was inactive.
- CUPS Browsed was inactive.
- UDP port 5353 was no longer listening.
- DNS resolution remained functional.
- Internet connectivity remained functional.
- SSH remained available.

Evidence:

- `evidence/avahi-baseline.txt`
- `evidence/avahi-hardening.txt`

## 5. GNOME Remote Desktop Hardening

GNOME Remote Desktop was reviewed as a potential remote-access service.

The baseline service was active and enabled, but no listeners were found on the standard RDP or VNC ports:

- TCP 3389 — no listener
- TCP 5900 — no listener

The system configuration showed that RDP was disabled, and the service had no configured TLS credentials or remote-access users.

The service was temporarily stopped to verify that normal system operation was unaffected. The following remained functional:

- graphical.target
- GDM
- SSH
- CUPS
- DNS resolution
- Internet connectivity

GNOME Remote Desktop was then permanently disabled.

After hardening:

- `gnome-remote-desktop.service` was inactive.
- The service was disabled.
- No RDP or VNC listeners were present.

Evidence:

- `evidence/gnome-remote-desktop-hardening.txt`

## 6. ModemManager Hardening

ModemManager was reviewed because the system did not have any detected cellular modem hardware.

Baseline checks showed:

- `ModemManager.service` was active and enabled.
- `mmcli -L` reported no modems.
- No TCP, UDP, or Unix socket listeners were associated with ModemManager.
- NetworkManager and systemd-resolved were operating normally.

The service was temporarily stopped to verify that networking remained functional.

After the test:

- NetworkManager remained active.
- systemd-resolved remained active.
- The Ethernet interface remained available.
- DNS resolution and Internet connectivity remained functional.

Because no modem hardware was present or required, ModemManager was permanently disabled.

After hardening:

- `ModemManager.service` was inactive.
- The service was disabled.
- Its related startup dependencies were removed.

Evidence:

- `evidence/modemmanager-hardening.txt`

## 7. Kerneloops Hardening

Kerneloops was reviewed as a kernel crash signature collection service.

Baseline checks showed:

- `kerneloops.service` was active and enabled.
- The service ran under the dedicated `kernoops` user and `adm` group.
- No TCP, UDP, or Unix socket listeners were associated with the service.
- Searches of `/var/log/kern.log` and the kernel journal for common kernel fault indicators returned no matching output during the investigation.

The service was temporarily stopped and the system was monitored for functional impact.

After the test:

- NetworkManager remained active.
- systemd-resolved remained active.
- The graphical target remained active.
- SSH remained available.
- Internet connectivity remained functional.

The service was then permanently disabled because it was not required for the intended system role.

After hardening:

- `kerneloops.service` was inactive.
- The service was disabled.
- Its multi-user startup link was removed.

Evidence:

- `evidence/kerneloops-hardening.txt`

## 8. CUPS Hardening

CUPS was reviewed because it provides local and network printing services that were not required for this system.

Baseline checks showed:

- `cups.service` was active and enabled.
- CUPS was listening only on:
  - `127.0.0.1:631`
  - `[::1]:631`
- No configured printers or default printer were present.
- `lpstat -p -d` reported no destinations.
- The absence of output from `lpstat -a` indicated that no printer destinations were configured.

The CUPS service was temporarily stopped to verify that essential system functions remained available.

After the test:

- SSH remained active.
- NetworkManager remained active.
- systemd-resolved remained active.
- DNS resolution and Internet connectivity remained functional.
- Port 631 was no longer listening.

CUPS was then permanently disabled along with its associated socket and path activation units:

- `cups.service`
- `cups.socket`
- `cups.path`

After hardening:

- CUPS was inactive.
- CUPS service, socket, and path units were disabled.
- Port 631 was no longer listening.

Evidence:

- `evidence/cups-hardening.txt`

# Phase 3 — Post-Hardening Verification

Post-hardening checks were performed to confirm that the security controls were active and that essential system functions remained operational.

## 1. Service Verification

The number of running service units decreased from the baseline count of 30 to 26 after the hardening activities.

The following services were confirmed inactive or disabled after hardening:

- Avahi Daemon
- CUPS Browsed
- GNOME Remote Desktop
- ModemManager
- Kerneloops
- CUPS

The remaining required services continued operating normally.

## 2. Network Listener Verification

The final listening sockets were reviewed after the hardening activities.

Expected system listeners included:

- `127.0.0.53:53` — systemd-resolved
- `127.0.0.54:53` — systemd-resolved
- `0.0.0.0:22` — SSH
- `[::]:22` — SSH

The following previously observed service ports were no longer listening:

| Service | Port | Post-Hardening Result |
| ------- | ---- | --------------------- |
| Avahi | UDP 5353 | No listener |
| CUPS | TCP 631 | No listener |
| GNOME Remote Desktop | TCP 3389 | No listener |
| VNC | TCP 5900 | No listener |
| SSH | TCP 22 | Remained available |

## 3. SSH Verification

The final effective SSH configuration was verified with `sshd -T`.

The hardened values included:

| Setting | Baseline | Hardened |
| ------- | -------- | -------- |
| MaxAuthTries | 6 | 3 |
| PermitRootLogin | without-password | no |
| PasswordAuthentication | yes | no |
| X11Forwarding | yes | no |

Public-key authentication was tested successfully.

A password-only authentication test was rejected with:

`Permission denied (publickey)`

## 4. Firewall Verification

The final UFW configuration was verified as active and enabled.

| Firewall Setting | Final State |
| ---------------- | ----------- |
| Status | Active |
| Startup | Enabled |
| Incoming | Deny |
| Outgoing | Allow |
| Routed | Deny |
| SSH TCP 22 | Allowed |
| Logging | Low |

## 5. Connectivity Verification

After service hardening, essential connectivity checks remained successful.

The following were verified:

- NetworkManager active
- systemd-resolved active
- Ethernet interface operational
- DNS resolution functional
- Internet connectivity functional
- SSH service available

Connectivity tests to external IP and hostname targets completed successfully during the hardening process.

# Phase 4 — Before-and-After Comparison

The following comparison summarizes the principal security changes made during the project.

| Security Area | Baseline | Post-Hardening |
| ------------- | -------- | -------------- |
| Kernel | 7.0.0-30-generic | 7.0.0-31-generic |
| UFW | Inactive | Active and enabled |
| UFW incoming policy | ACCEPT | DENY |
| UFW outgoing policy | ACCEPT | ALLOW |
| SSH password authentication | Enabled | Disabled |
| SSH root login | without-password | Disabled |
| SSH MaxAuthTries | 6 | 3 |
| SSH X11 forwarding | Enabled | Disabled |
| Avahi | Active/enabled | Disabled |
| CUPS Browsed | Active/enabled | Disabled |
| GNOME Remote Desktop | Active/enabled | Disabled |
| ModemManager | Active/enabled | Disabled |
| Kerneloops | Active/enabled | Disabled |
| CUPS | Active/enabled | Disabled |
| Running service units | 30 | 26 |
| SSH | Listening on TCP 22 | Remained available |
| CUPS | Listening on TCP 631 | No listener |
| Avahi | UDP 5353 listener | No listener |
| RDP | No listener | No listener |
| VNC | No listener | No listener |

The comparison demonstrates that the hardening work reduced unnecessary services and network exposure while preserving required connectivity and SSH access.

# Conclusion

This project followed a baseline-to-hardening-to-verification methodology to assess and improve the security posture of an Ubuntu 24.04 system.

The baseline assessment identified several areas for hardening, including firewall configuration, SSH authentication, system updates, and unnecessary services.

The hardening phase then:

- enabled a restrictive UFW firewall policy,
- applied available system updates and rebooted into the updated kernel,
- strengthened SSH authentication and access controls,
- disabled unnecessary network discovery and remote-access services,
- disabled services that were not required by the system's intended role,
- and verified that essential networking, DNS, graphical operation, and SSH access remained functional.

Post-hardening verification showed that the number of running service units decreased from 30 to 26. Previously observed listeners for Avahi, CUPS, and other disabled services were no longer present, while SSH remained available on TCP port 22.

The project demonstrates the importance of establishing a measurable baseline, making controlled security changes, and validating both the security improvements and the continued availability of required services.

All major findings, actions, and verification results are supported by evidence files stored in the project `evidence/` directory.
