# Linux-Log-Rotation-and-Retention-Investigation
## Overview
### 1. What Is Log Rotation?

Linux systems continuously generate logs containing information about system activity, authentication, services, applications, and security events.

Over time, these files can grow significantly and consume disk space.

Log rotation is the process of managing log files by moving the current log into a historical copy, creating a new log file, and optionally compressing or deleting older copies according to a retention policy.

Linux commonly uses `logrotate` to perform this task.

### 2. How Log Rotation Works

A typical log rotation process involves:

1. Checking whether a log file is due for rotation.
2. Renaming or moving the existing log file.
3. Creating a new log file when configured.
4. Optionally compressing the rotated file.
5. Retaining a defined number of historical copies.
6. Removing older copies when they exceed the retention limit.

For example, a system may maintain:

```text
messages
messages-20261108
messages-20261101
messages-20261025
```

Here, `messages` is the current log, while the date-suffixed files represent historical copies.

### 3. Understanding Logrotate Configuration

The main configuration file is:

```text
/etc/logrotate.conf
```

Application-specific policies are generally stored in:

```text
/etc/logrotate.d/
```

Important directives include:

| Directive | Purpose |
|---|---|
| `daily` | Rotate daily |
| `weekly` | Rotate weekly |
| `monthly` | Rotate monthly |
| `rotate 4` | Retain four old rotations |
| `create` | Create a new log file after rotation |
| `dateext` | Add a date suffix to rotated files |
| `compress` | Compress rotated files |
| `delaycompress` | Delay compression until the next rotation |
| `missingok` | Do not report an error if the log file is missing |
| `notifempty` | Skip rotation if the log file is empty |
| `copytruncate` | Copy the log and truncate the original |

Application-specific policies can override global defaults.

### 4. Logrotate Scheduling

Logrotate is commonly invoked through a systemd timer or another scheduled mechanism.

On a system using systemd, the following units are relevant:

```text
logrotate.timer
logrotate.service
```

The timer determines when the service is invoked.

However, the timer schedule and the rotation frequency are different concepts. Even if logrotate runs daily, a log configured for weekly rotation is not necessarily rotated every day.

### 5. Security Importance of Log Retention

Logs are important during incident response and forensic investigations.

They can help analysts reconstruct:

- Authentication activity.
- Privileged command execution.
- Service failures.
- Suspicious system activity.
- Changes to system configuration.
- Events preceding and following a security incident.

Poor retention settings can reduce the availability of historical evidence.

For example:

- Too few retained copies may remove useful investigation data.
- Premature deletion may create gaps in the timeline.
- Excessive retention may consume disk space.
- Unauthorized modification of log files may affect evidence integrity.
- Missing or incomplete logs may limit incident reconstruction.

### 6. Log Rotation and Security Monitoring

Logrotate configuration and rotated files should be examined alongside system monitoring evidence.

Wazuh can provide visibility into relevant activity through collected logs, audit events, and File Integrity Monitoring.

However, Wazuh does not automatically record every rotation operation.

Analysts should distinguish between:

- A command that inspects logrotate configuration.
- Execution of the logrotate utility.
- Actual rotation of a log file.
- Deletion or modification of historical logs.

Each activity requires appropriate supporting evidence.

### 7. Investigation Approach

A practical investigation should examine:

1. The installed logrotate utility.
2. Global and application-specific policies.
3. Rotation frequency and retention limits.
4. Existing current and historical log files.
5. Scheduled execution through systemd.
6. Relevant Wazuh and audit events.

The goal is to determine whether log retention is configured as expected and whether available evidence supports the observed behavior.

### Key Takeaway

Log rotation is a normal Linux maintenance process, but its configuration directly affects the availability of security evidence.

An analyst should understand the difference between configured policy, scheduled execution, actual file rotation, and SIEM visibility.

**Follow the evidence, not the assumption.**

This lab investigates Linux log rotation and retention on a Rocky Linux 9.8 server. It focuses on understanding how `logrotate` manages system logs, how retention policies are configured, and how rotated log files can be examined during a security investigation.

The investigation also uses Wazuh Discover to review collected administrative activity and understand the difference between command-execution events and actual log rotation evidence.

## Environment

- Operating System: Rocky Linux 9.8
- SIEM: Wazuh 4.13.1
- Log management utility: logrotate
- Log directory: `/var/log/`
- Configuration: `/etc/logrotate.conf`
- Application policies: `/etc/logrotate.d/`

## Lab Objectives 

The objective of this lab is to develop practical skills in examining Linux log management and understanding how retention policies affect security investigations.

By completing this lab, the analyst should be able to:

- Identify whether the logrotate utility is installed and available.
- Examine global and application-specific log rotation configurations.
- Interpret rotation frequency, retention count, date suffixes, and compression settings.
- Identify current log files and their historical rotated copies.
- Understand the relationship between systemd timer execution and individual log rotation policies.
- Examine Wazuh Discover events related to administrative inspection and log management.
- Distinguish configuration evidence from actual execution evidence.
- Recognize potential gaps in log retention that could affect incident response.
- Avoid classifying normal administrative activity as malicious without supporting evidence.

The final objective is to assess whether the available configuration, filesystem artifacts, and monitoring events provide sufficient evidence to understand the server's log retention behavior.

## Lab Scenario

### Scenario

You are working as a SOC analyst responsible for monitoring a Rocky Linux server that hosts security monitoring components, including Wazuh.

During a routine review, you are asked to examine the server's log management practices and determine whether historical system and authentication logs are being retained appropriately.

The server continuously generates logs under `/var/log/`, including `messages`, `secure`, `cron`, and other system-related files. Over time, these logs grow in size and require rotation to prevent excessive disk usage.

As part of the investigation, you must review the logrotate configuration, identify application-specific policies, and examine the existing rotated log files.

You also observe administrative command activity in Wazuh Discover while inspecting the log rotation configuration directory.

### Investigation Requirements

Your task is to:

- Verify that logrotate is installed.
- Review the global log rotation and retention policy.
- Identify application-specific configuration files.
- Examine current and historical log files.
- Check the scheduled execution mechanism.
- Review relevant Wazuh events.
- Determine whether the available evidence confirms normal log management activity or indicates a possible retention concern.

### Investigation Considerations

During the investigation, you must distinguish between a scheduled logrotate service and an actual rotation operation.

A daily timer does not necessarily mean that every log is rotated daily. Similarly, a Wazuh sudo event showing that an administrator inspected a configuration directory does not prove that log rotation occurred.

You should rely on configuration, filesystem artifacts, timestamps, and monitoring evidence together.

### Expected Outcome

At the end of the investigation, you should be able to explain:

- How log rotation is configured on the server.
- How many historical rotations are retained by the global policy.
- Whether compression is enabled.
- Which application-specific policies are available.
- What rotated log files currently exist.
- When logrotate is scheduled to run.
- What Wazuh actually recorded during the investigation.

The final assessment should clearly separate confirmed observations from assumptions and identify any evidence gaps that require further investigation.

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

