# Project 04 — Linux Log Analysis

## Project Overview

This project demonstrates the investigation and analysis of Linux
authentication logs using command-line security tools.

The laboratory focuses on SSH authentication activity and demonstrates
how security analysts can identify authentication failures, successful
logins, repeated attempts, false positives, and suspicious patterns.

## Objectives

- Understand Linux authentication logs.
- Investigate SSH authentication events.
- Identify failed and successful authentication attempts.
- Extract usernames, timestamps, source addresses, and ports.
- Count repeated authentication failures.
- Identify and eliminate false positives.
- Correlate related authentication events.
- Document security findings professionally.

## Environment

- Ubuntu 24.04.3 LTS
- OpenSSH Server
- Linux command line
- `/var/log/auth.log`

## Tools Used

- `grep`
- `awk`
- `sort`
- `uniq`
- `systemctl`
- `ss`
- `ps`
- SSH

## Project Structure

```text
04-linux-log-analysis/
├── analysis/
│   └── authentication-analysis.txt
├── logs/
├── reports/
│   └── security-investigation.md
└── README.md
```

## Investigation Methodology

The investigation followed these stages:

1. Examine the authentication log.
2. Assess the system's SSH environment.
3. Install and verify the OpenSSH server.
4. Generate controlled SSH authentication events.
5. Identify successful and failed authentication events.
6. Filter false positives.
7. Extract security-relevant fields.
8. Count repeated authentication failures.
9. Correlate related events.
10. Document and assess the findings.

## Key Findings

Six failed SSH authentication events were identified during the
controlled laboratory exercise.

All six events:

- Targeted the `vboxuser` account.
- Originated from the IPv6 loopback address `::1`.
- Occurred between `22:51:04` and `23:48:14`.
- Were intentionally generated as part of the investigation.

The activity was therefore classified as benign laboratory activity.

## False Positive Handling

Broad searches initially returned false positives because search
terms appeared inside previously executed commands that were recorded
in the authentication log.

A more precise regular expression was used to isolate genuine SSH
failed-password events:

`sshd\[[0-9]+\]: Failed password`

## Skills Demonstrated

- Linux log analysis
- SSH authentication investigation
- Regular expressions
- `grep`
- `awk`
- `sort`
- `uniq`
- Event correlation
- False-positive identification
- Security documentation
- Evidence-based assessment

## Conclusion

This project demonstrates the basic workflow used when investigating
Linux authentication activity.

The exercise emphasizes that security analysts must validate log
entries, distinguish evidence from assumptions, and consider the
context surrounding an event before declaring malicious activity.
