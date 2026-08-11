# Linux Incident Response Report

## 1. Incident Overview

### Incident Type

Simulated malicious cron persistence.

### Environment

- Operating System: Ubuntu Linux
- Environment: VirtualBox
- User: vboxuser
- Incident Type: Simulated cybersecurity incident

### Executive Summary

A suspicious cron job named `security-update` was identified
under `/etc/cron.d/`.

The job was configured to execute every five minutes as the
`root` user and write `SIMULATED_SECURITY_EVENT` to
`/var/log/security-lab.log`.

The activity was investigated using system configuration,
filesystem metadata, cron logs, and system journal evidence.

The cron job was preserved as evidence and subsequently removed
from the live system.

Post-containment verification confirmed that the simulated
activity was no longer executing.

---

## 2. Detection

The suspicious artifact identified during the investigation was:

```text
/etc/cron.d/security-update
```

---

## 3. Investigation Findings

### Cron Configuration

The suspicious file was owned by:
`root:root`

with permissions:
`0644`

The cron service was confirmed to be active.

### Execution Evidence

Cron journal entries confirmed execution of the command:
`(root) CMD (echo "SIMULATED_SECURITY_EVENT" >> /var/log/security-lab.log)`

### Log Evidence

The file:
`/var/log/security-lab.log`

contained repeated:
`SIMULATED_SECURITY_EVENT`

entries.

### File Metadata

The suspicious cron file was examined using stat.

The artifact was created and modified on:
`2026-08-09`

The exact timestamps are preserved in the investigation evidence.

---

## 4. Evidence Collected

The following evidence was preserved:

`evidence/security-update.evidence`

Additional evidence reviewed included:
- `/etc/cron.d/security-update`
- `/var/log/security-lab.log`
- cron journal entries
- file metadata

---

## 5. Incident Timeline

### Initial Discovery

An unexpected cron job named `security-update` was identified
under `/etc/cron.d/`.

### Investigation

The cron configuration was examined and the scheduled command
was identified.

### Execution Confirmed

Cron journal entries confirmed that the command was executing
every five minutes as `root`.

### Artifact Identified

The command wrote:
`SIMULATED_SECURITY_EVENT`

to:
`/var/log/security-lab.log`

### Evidence Preservation

A copy of the suspicious cron file was preserved as:

`evidence/security-update.evidence`

### Containment

The suspicious cron job was removed from the live system.

### Eradication

Additional searches confirmed that no remaining references to
`security-update` or the `SIMULATED_SECURITY_EVENT` existed in the
cron configuration.

### Recovery

The cron service remained active and operational.

Post-containment verification showed no new simulated event
executions.

---

## 6. Containment

The suspicious cron job was removed after evidence preservation.

The legitimate cron entries were not modified.

The evidence copy was retained for investigation:

`evidence/security-update.evidence`

Containment was successfully verified.

## 7. Eradication

The malicious persistence mechanism was removed from the live
system.

Searches of `/etc/cron*` found no remaining references to:

`security-update`

or to:

`SIMULATED_SECURITY_EVENT`

The historical log containing the simulated events was retained
as evidence.

## 8. Recovery

The cron service was verified after containment:
`Active: active (running)`

No new simulated event executions were observed in the cron
journal after the suspicious job was removed.

The system was therefore considered recovered from the simulated
persistence mechanism.

---

## 9. Impact Assessment

This was a controlled simulated incident.

The observed activity demonstrated how a malicious actor could
use a scheduled task to establish persistence and execute a
command with elevated privileges.

No real compromise or malicious activity is being claimed.

---

## 10. Lessons Learned

This investigation demonstrated the importance of:

- Reviewing scheduled tasks during incident response.
- Investigating unexpected files in `/etc/cron.d`.
- Checking file ownership and permissions.
- Reviewing cron journal activity.
- Preserving evidence before removing suspicious artifacts.
- Comparing current system state against an established baseline.
- Verifying that suspicious activity stops after containment.
- Avoiding conclusions without supporting evidence.

---

## 11. Final Assessment

The simulated incident involved a cron-based persistence
mechanism executing a command as root every five minutes.

The artifact was identified, investigated, preserved as evidence,
removed from the live system, and subsequently verified as no
longer executing.

The incident response process was completed through:

Detection → Investigation → Evidence Preservation → Containment
→ Eradication → Recovery

### Final Status

**SIMULATED INCIDENT — CONTAINED AND RECOVERED**
