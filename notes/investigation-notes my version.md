# Investigation Notes

## 1. Investigation Summary

The objective was to examine how the Rocky Linux server manages log rotation and retention, identify existing rotated logs, and review whether Wazuh provided relevant visibility into administrative activity.

The investigation focused on configuration and filesystem evidence without manually forcing log rotation or modifying retention settings.

## 2. Logrotate Installation

Command:

```bash
command -v logrotate
```

Observed output:

```text
/usr/sbin/logrotate
```

Finding:

The logrotate utility is installed and available at `/usr/sbin/logrotate`.

## 3. Global Configuration Review

Command:

```bash
sudo cat /etc/logrotate.conf
```

Relevant configuration:

```text
weekly
rotate 4
create
dateext
#compress
include /etc/logrotate.d
```

Analysis:

- `weekly`: Configures weekly rotation by default.
- `rotate 4`: Retains four old rotations.
- `create`: Creates a new log file after rotation.
- `dateext`: Uses dates as suffixes for rotated files.
- `compress`: Commented out, so compression is not enabled globally.
- `include`: Loads additional policies from `/etc/logrotate.d/`.

The global settings provide the default policy. Individual application configurations may override these settings.

## 4. Application-Specific Configuration

Command:

```bash
sudo ls -l /etc/logrotate.d/
```

Observed configuration files included:

```text
bootlog
btmp
chrony
dnf
firewalld
iscsiuiolog
kvm_stat
psacct
rsyslog
samba
sssd
wpa_supplicant
wtmp
```

Finding:

Application-specific log rotation policies are present.

The filenames alone do not establish their individual rotation frequency or retention settings. Each file must be inspected to determine its actual policy.

## 5. Existing Log Files

Command:

```bash
sudo ls -lh /var/log/
```

Important observations:

- `messages` had a current size of approximately 3.2 MB.
- `secure` had a current size of approximately 36 KB.
- Multiple date-suffixed rotated copies of both logs were present.
- Rotated `cron`, `maillog`, `spooler`, and `boot.log` files were also present.
- The observed rotated files were not compressed.

Examples:

```text
messages-20261025
messages-20261101
messages-20261108
messages-20261115

secure-20261025
secure-20261101
secure-20261108
secure-20261115
```

Finding:

Historical log copies are present in `/var/log/`.

The existence of these files supports the observation that log rotation has occurred. Their timestamps and names should be correlated with the corresponding configuration and rotation state before drawing conclusions about exact execution times.

## 6. Scheduled Execution

Command:

```bash
systemctl list-timers --all | grep logrotate
```

Observed output:

```text
Mon 2026-11-16 00:00:00 IST
Sun 2026-11-15 07:11:15 IST
logrotate.timer
logrotate.service
```

Finding:

The logrotate timer is scheduled for daily execution at midnight.

Important distinction:

The timer schedule is daily, while the global logrotate policy specifies weekly rotation. Logrotate can run daily and still rotate a particular log only when its policy and rotation state indicate that rotation is due.

## 7. Wazuh Discover Review

A Wazuh event was observed for the command:

```text
/bin/ls -l /etc/logrotate.d/
```

Event details:

| Field | Value |
|---|---|
| Agent ID | 000 |
| Agent name | localhost.localdomain |
| Rule ID | 5402 |
| Rule level | 3 |
| Rule description | Successful sudo to ROOT executed. |
| Decoder | sudo |
| Event source | journald |
| Command | `/bin/ls -l /etc/logrotate.d/` |

Analysis:

The event records administrative command execution while inspecting the logrotate configuration directory.

It is not a logrotate execution event and does not indicate that a log file was rotated, deleted, or compressed.

## 8. Investigation Assessment

| Area | Finding |
|---|---|
| Logrotate installation | Confirmed |
| Global configuration | Weekly, four rotations |
| Compression | Disabled globally |
| Application policies | Present |
| Rotated log files | Observed |
| Scheduled timer | Daily |
| Wazuh administrative event | Observed |
| Actual rotation execution in Wazuh | Not established |

