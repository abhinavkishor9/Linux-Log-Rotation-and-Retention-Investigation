# Investigation Timeline

| Sequence | Activity | Evidence / Observation |
|---|---|---|
| 1 | Verified logrotate installation | `/usr/sbin/logrotate` |
| 2 | Inspected global configuration | Weekly rotation, four retained rotations, date suffix enabled |
| 3 | Reviewed application-specific policies | Multiple configuration files present in `/etc/logrotate.d/` |
| 4 | Inspected `/var/log/` | Current and date-suffixed historical log files observed |
| 5 | Reviewed scheduled execution | `logrotate.timer` scheduled for midnight |
| 6 | Examined Wazuh Discover | Sudo event for inspecting `/etc/logrotate.d/` |
| 7 | Assessed evidence | Administrative inspection confirmed; actual rotation execution in Wazuh not established |

