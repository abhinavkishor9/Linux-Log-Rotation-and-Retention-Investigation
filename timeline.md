# Investigation Timeline – Linux Log Rotation and Retention

**Date:** 15 November 2026  
**System:** Rocky Linux 9.8  
**SIEM:** Wazuh 4.13.1

| Sequence | Activity | Evidence / Observation |
|---|---|---|
| 1 | Verified logrotate installation | `/usr/sbin/logrotate` |
| 2 | Inspected global configuration | Weekly rotation, four retained rotations, date suffix enabled |
| 3 | Reviewed application-specific policies | Multiple configuration files present in `/etc/logrotate.d/` |
| 4 | Inspected `/var/log/` | Current and date-suffixed historical log files observed |
| 5 | Reviewed scheduled execution | `logrotate.timer` scheduled for midnight |
| 6 | Examined Wazuh Discover | Sudo event for inspecting `/etc/logrotate.d/` |
| 7 | Assessed evidence | Administrative inspection confirmed; actual rotation execution in Wazuh not established |

## Key Observations

- Logrotate was installed and its global configuration was available.
- The global policy specified weekly rotation and retention of four rotations.
- Compression was not enabled globally.
- Historical copies of system and authentication logs were present.
- The logrotate systemd timer was scheduled to run daily.
- Wazuh recorded an administrative command, not a confirmed log rotation event.

## Final Assessment

The investigation confirmed the configured log rotation policy, scheduled execution, and presence of rotated log files.

No evidence of malicious log deletion or deliberate tampering was established from the observations collected during this lab.
