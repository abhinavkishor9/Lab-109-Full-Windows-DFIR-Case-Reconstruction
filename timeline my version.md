# Timeline
| Time (IST) | Event | Evidence | Assessment |
|---|---|---|---|
| 06:14:26.532 | Case initialized | `case-start.txt` | Beginning of the recorded investigation window. |
| 06:14:26 | System and monitoring baseline commands executed | `system-baseline.csv`, `monitoring-services-baseline.csv` | Endpoint details and service states were exported. |
| 06:14:27.475 | Sysmon Event ID 13 | Record ID `1387116` in `sysmon-case-events.csv` | Registry value set by `TrustedInstaller.exe` under a TrustedInstaller Winlogon notification path. This is unrelated to the controlled case Registry key. |
| 06:16:18.335 | Sysmon Event ID 11 | Record ID `1387178` | `case-document.ps1` created in the case test directory by PowerShell PID 9748. |
| 06:16:18.348 | Test timestamp recorded | `file-created-time.txt` | Timestamp written after the initial file-creation command. |
| 06:16:18.361 | Sysmon Event ID 11 | Record ID `1387179` | `case-copy.ps1` created in the case test directory by PowerShell PID 9748. |
| Not captured precisely | File content appended | Controlled PowerShell command | Additional content was written to `case-document.ps1`; no separate content-modification record was confirmed in the supplied Sysmon export. |
| Not captured precisely | Copy renamed and deleted | Controlled PowerShell commands | Commands were executed, but no matching rename or deletion event was confirmed in the supplied export. |
| Not captured precisely | Registry test value modified | `registry-final-state.csv` | The final exported value was `CaseStatus = Modified`. The local Sysmon Event ID 13 record in this export was unrelated. |
| 06:17:31.012 | Process-test start timestamp | `process-test-start.txt` | Timestamp captured before the controlled `whoami.exe` execution. |
| 06:17:31.082 | Sysmon Event ID 1 | Record ID `1387212` | `whoami.exe` executed as `DESKTOP-9MMM37V\Dell`, PID 16364, from a PowerShell parent process. |
| Not captured precisely | Sysmon baseline saved | `sysmon-config-baseline.txt` | Recorded current configuration path, hash, and settings. |
| Not captured precisely | Wazuh agent configuration hash saved | `wazuh-config-baseline-hash.csv` | Recorded the `ossec.conf` hash. |
| Not captured precisely | USN journal queried | `usn-journal-case-query.txt` | Journal configuration returned the journal ID, first/next USN values, size, and supported versions. |
| Not captured precisely | USN journal read attempted | `usn-journal-case-output.csv` | Failed because `0xYOUR_RECORDED_USN` was used as a literal placeholder. No valid USN records were collected in this output. |
| Not captured precisely | MFT acquisition/parsing | No artifact supplied | Not demonstrated; MFT correlation remains pending. |
| Not captured precisely | Evidence hashes generated | `evidence-hashes.csv` | SHA-256 values were calculated for the collected files, excluding the manifest itself. |

