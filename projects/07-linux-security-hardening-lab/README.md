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

| Item                 | Value              |
| -------------------- | ------------------ |
| Operating System     | Ubuntu 24.04.3 LTS |
| Kernel               | 7.0.0-30-generic   |
| Architecture         | x86-64             |
| Virtualization       | Oracle VirtualBox  |
| Investigation User   | vboxuser           |
| User UID             | 1000               |
| Primary Group        | vboxuser           |
| Administrative Group | sudo               |

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

## 2. SSH Assessment

The system uses **systemd socket activation** for SSH.

Baseline state:

- `ssh.service`: inactive (dead)
- `ssh.service`: disabled
- `ssh.socket`: active and listening
- `ssh.socket`: enabled
- IPv4 listener: `0.0.0.0:22`
- IPv6 listener: `[::]:22`

Port 22 was observed listening and owned by systemd (PID 1).

The SSH service defines:

```text
RuntimeDirectory=sshd
RuntimeDirectoryMode=0755
```

The `/run/sshd` directory was not present while `ssh.service` was inactive.

Attempts to run:

```bash
sudo sshd -T
sudo sshd -t
```

returned:

```text
Missing privilege separation directory: /run/sshd
```

This is consistent with the service being inactive because systemd is configured to create the runtime directory when the SSH service starts.

No SSH configuration was changed during the baseline.

Evidence:

`evidence/baseline-ssh.txt`

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

## In Progress

- [ ] Baseline README review
- [ ] Baseline Git commit
- [ ] System hardening
- [ ] Post-hardening verification
- [ ] Before-and-after comparison
- [ ] Final security report
- [ ] Final Git commit and GitHub push

---

# Conclusion

The baseline assessment provides a documented security snapshot of the Ubuntu system before hardening.

The most significant identified hardening opportunities are:

- enabling and configuring a restrictive host firewall,
- applying pending system updates,
- reviewing SSH configuration,
- and evaluating unnecessary services that may increase the system's attack surface.

Other inspected controls, including sensitive file permissions and several kernel security settings, were already configured with protective values.

The next phase will apply selected hardening measures and verify their effect using the same evidence-driven methodology.
