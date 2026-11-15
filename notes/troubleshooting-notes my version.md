# Troubleshooting Notes

## 1. Understanding the Difference Between Timer and Rotation Frequency

Observation:

The logrotate timer was scheduled to execute daily, while `/etc/logrotate.conf` specified:

```text
weekly
```

Explanation:

The systemd timer controls when the logrotate service is invoked. The rotation policy determines whether an individual log file is due for rotation.

A daily timer does not mean every log file is rotated daily.

## 2. Global Settings Versus Application Policies

Observation:

The `/etc/logrotate.d/` directory contained several application-specific configuration files.

Explanation:

The global configuration provides default settings, but application-specific policies may override them.

To investigate a particular log, inspect its corresponding configuration file instead of relying only on `/etc/logrotate.conf`.

Example:

```bash
sudo cat /etc/logrotate.d/rsyslog
```

## 3. Understanding Date-Suffixed Files

Observation:

Rotated files such as `messages-20261108` and `secure-20261108` were present.

Explanation:

The `dateext` option enables date-based suffixes for rotated files.

A date suffix identifies the naming convention used for the rotated copy. It should not automatically be treated as proof of the exact time when the rotation operation occurred.

## 4. Compression Was Not Enabled Globally

Observation:

The global configuration contained:

```text
#compress
```

Explanation:

The compression directive was commented out.

Therefore, compression is not enabled by the global configuration. An application-specific policy may still enable compression independently.

## 5. Interpreting Wazuh Discover Events

Observation:

Wazuh displayed rule ID `5402` for the command:

```text
/bin/ls -l /etc/logrotate.d/
```

Explanation:

This is a sudo-related event recording administrative command execution.

It does not prove that logrotate ran.

Analysts should distinguish between:

- Inspecting logrotate configuration.
- Executing the logrotate utility.
- Rotating an individual log file.
- Deleting or compressing historical logs.

These are separate activities and require separate evidence.

## 6. Wazuh Visibility Limitations

Wazuh does not automatically generate a dedicated alert for every log rotation operation.

Visibility depends on the available log sources, audit configuration, and File Integrity Monitoring coverage.

A missing Wazuh alert does not prove that rotation did not occur.

Similarly, a sudo event does not prove that a log file was modified.

