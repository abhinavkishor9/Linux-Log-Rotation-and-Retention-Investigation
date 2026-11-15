# Linux Log Rotation and Retention Investigation

## Overview

This lab investigates Linux log rotation and retention on a Rocky Linux 9.8 server. It focuses on understanding how `logrotate` manages system logs, how retention policies are configured, and how rotated log files can be examined during a security investigation.

The investigation also uses Wazuh Discover to review collected administrative activity and understand the difference between command-execution events and actual log rotation evidence.

## Environment

- Operating System: Rocky Linux 9.8
- SIEM: Wazuh 4.13.1
- Log management utility: logrotate
- Log directory: `/var/log/`
- Configuration: `/etc/logrotate.conf`
- Application policies: `/etc/logrotate.d/`

## Investigation Objectives

- Review the global logrotate configuration.
- Identify application-specific rotation policies.
- Examine rotation frequency and retention settings.
- Inspect existing rotated and compressed log files.
- Review the logrotate systemd timer.
- Correlate available administrative activity with Wazuh events.
- Identify potential log retention gaps without assuming malicious activity.

## Key Findings

The global configuration contained the following settings:

| Setting | Observed value |
|---|---|
| Rotation frequency | Weekly |
| Retention | 4 rotations |
| Create new log file | Enabled |
| Date-based suffix | Enabled |
| Compression | Disabled |
| Application policies | `/etc/logrotate.d/` |

The logrotate timer was scheduled to run daily at midnight. However, the actual rotation decision depends on the individual log's policy and rotation state.

Existing rotated files were identified for system and authentication logs, including:

- `messages-20261025`
- `messages-20261101`
- `messages-20261108`
- `messages-20261115`
- `secure-20261025`
- `secure-20261101`
- `secure-20261108`
- `secure-20261115`

## Wazuh Findings

Wazuh Discover displayed a successful sudo event associated with:

`/bin/ls -l /etc/logrotate.d/`

Event details:

- Rule ID: `5402`
- Rule level: `3`
- Rule description: Successful sudo to ROOT executed.
- Agent: `000` (local Wazuh server)
- Event source: `journald`

This event confirms administrative command execution. It does not establish that logrotate executed or that any log files were rotated.

## Conclusion

The investigation confirmed that logrotate is installed, global retention settings are configured, application-specific policies are present, and rotated log files exist.

The Wazuh event provided supporting evidence of administrative inspection activity, but no direct logrotate execution event was established during this investigation.

The findings demonstrate why analysts should correlate configuration, filesystem evidence, scheduled execution, and SIEM events rather than treating a single administrative event as proof of log rotation.

## Evidence-Based Approach

- Configuration indicates intended rotation behavior.
- Existing rotated files demonstrate that rotated log copies are present.
- The systemd timer indicates scheduled execution.
- Wazuh events provide visibility into collected activity.
- None of these observations alone proves that every configured log was rotated successfully.
