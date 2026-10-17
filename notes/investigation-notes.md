# Lab 109 — Full Windows DFIR Case Reconstruction Notes

## 1. Case Summary
This investigation combines endpoint baseline collection, Sysmon event review, controlled file operations, Registry changes, NTFS journal examination, and evidence hashing.
The case began on `DESKTOP-9MMM37V` on 17 October 2026. The collected artifacts support a controlled file-activity sequence and a `whoami.exe` process execution. The Registry export records the final test value, while the current Sysmon export does not contain a matching event for that Registry key.
The USN journal configuration was queried successfully, but the journal-read command did not collect valid records. MFT acquisition and parsing were not shown in the supplied evidence.

## 2. Case Initialization
The case start timestamp recorded in `case-start.txt` was:
`2026-10-17T06:14:26.5327718+05:30`

The file activity marker in `file-created-time.txt` was:
`2026-10-17T06:16:18.3487820+05:30`

The process-test marker in `process-test-start.txt` was:
`2026-10-17T06:17:31.0127429+05:30`

The evidence and test directories were:
- `C:\WindowsDFIRCase109\Evidence`
- `C:\WindowsDFIRCase109\TestArea`

## 3. System and Monitoring Baseline
The system baseline output contained:
| Field | Value |
|---|---|
| Computer name | `DESKTOP-9MMM37V` |
| Windows product name reported | `Windows 10 Pro` |
| Windows version field | `2009` |
| OS build | `26200` |

The monitoring service baseline contained:
| Service | State | Startup |
|---|---|---|
| `Sysmon64` | Running | Automatic |
| `WazuhSvc` | Running | Automatic |

The reported product-name field should be preserved as captured. Its naming can be verified separately if required; the source data should not be silently replaced with an inferred value.

## 4. Sysmon Configuration
The Sysmon configuration output reported:
- Sysmon version: `15.21`
- Service name: `Sysmon64`
- Driver name: `SysmonDrv`
- Configuration path: `C:\Tools\Sysmon\sysmonconfig.xml`
- Configuration hash: `SHA256=D4290CD763B7C363A7C0FEEC5873A43906018AAA00EF95A8F9AFF26D69942DDB`
- Hashing algorithms: `MD5,SHA256,IMPHASH`
- Network connection logging: enabled
- Image loading: disabled
- CRL checking: enabled
- DNS lookup: enabled

The configuration report was preserved as `sysmon-config-baseline.txt`. The Wazuh agent configuration hash was exported separately to `wazuh-config-baseline-hash.csv`.

## 5. Controlled File Activity
The controlled file sequence created and modified `case-document.ps1`, copied it to `case-copy.ps1`, renamed the copy to `case-renamed.ps1`, and deleted the renamed copy.

The Sysmon export confirmed the following Event ID 11 records:
| Local time (IST) | Record ID | Target file | Process |
|---|---:|---|---|
| 2026-10-17 06:16:18.335 | 1387178 | `C:\WindowsDFIRCase109\TestArea\case-document.ps1` | PowerShell 7, PID 9748 |
| 2026-10-17 06:16:18.361 | 1387179 | `C:\WindowsDFIRCase109\TestArea\case-copy.ps1` | PowerShell 7, PID 9748 |

These records support the creation of the original file and its copy. They do not independently prove that the copy was renamed or deleted; the supplied event export contains no confirmed rename/deletion record for the test file.

The final evidence manifest includes `TestArea\case-document.ps1`, the file expected to remain after the controlled sequence. The absence of the other temporary file from the manifest is consistent with the command sequence, but the manifest itself does not prove a deletion event.

## 6. Process Execution
The process-test timestamp recorded in `process-test-start.txt` was:
`2026-10-17T06:17:31.0127429+05:30`

Sysmon Event ID 1 record `1387212` recorded `whoami.exe` at `2026-10-17 06:17:31` local time.

| Field | Observed value |
|---|---|
| Event ID | 1 |
| Record ID | 1387212 |
| Process ID | 16364 |
| Image | `C:\Windows\System32\whoami.exe` |
| User | `DESKTOP-9MMM37V\Dell` |
| Integrity level | High |
| Parent process ID | 9748 |
| Parent image | PowerShell 7 executable |
| SHA-256 | `23240EF9F8B0A9A324110B1C2331DE31DC1B0E08F5359CB707E51A939AF56CD3` |

The event aligns closely with the recorded process-test timestamp. It establishes process execution, not malicious behavior.

## 7. Registry Investigation
The controlled Registry location was:
`HKCU:\Software\WindowsDFIRCase109`

The test created a string value named `CaseStatus`, set it to `Created`, and then changed it to `Modified`.

The final export in `registry-final-state.csv` showed:
| Value | Final data |
|---|---|
| `CaseStatus` | `Modified` |

The supplied `sysmon-case-events.csv` contained one Event ID 13 record, but it did not reference the controlled Registry key. Its details were:
| Field | Value |
|---|---|
| Event ID | 13 |
| Record ID | 1387116 |
| Process | `C:\WINDOWS\servicing\TrustedInstaller.exe` |
| Target object | `HKLM\System\CurrentControlSet\Control\Winlogon\Notifications\Components\TrustedInstaller\Events` |
| Details | `CreateSession` |
| User | `NT AUTHORITY\SYSTEM` |

This event documents unrelated TrustedInstaller Registry activity. It cannot be used as direct evidence of the controlled `CaseStatus` change.

A separate supplied Wazuh screenshot showed an Event ID 13 dated 16 October 2026. That record predates the current case start on 17 October and must not be merged into this case timeline without additional correlation evidence.

## 8. Sysmon Event Collection
The supplied `sysmon-case-events.csv` contained 115 records:
| Event ID | Count |
|---|---:|
| 1 | 65 |
| 11 | 49 |
| 13 | 1 |
| Total | 115 |

The export demonstrates that Sysmon was recording process-creation and file-creation events during the case window. Most records are unrelated background activity and should not be treated as part of the controlled case.
The one Registry value-set event in the CSV did not target `HKCU\Software\WindowsDFIRCase109`. The supplied export did not contain Events 12, 14, 23, or 26 for the controlled test activity.

## 9. USN Journal Collection
The USN journal configuration query reported:
| Field | Value |
|---|---|
| Journal ID | `0x01da57b6418ad7a2` |
| First USN | `0x00000018dd000000` |
| Next USN | `0x00000018df20d4d8` |
| Lowest valid USN | `0x0000000000000000` |
| Maximum size | 32 MB |
| Allocation delta | 8 MB |
| Supported record versions | 2 through 4 |
| Write range tracking | Disabled |

The query output was saved to `usn-journal-case-query.txt`.

The following command used a literal placeholder:
`fsutil usn readjournal C: startUsn=0xYOUR_RECORDED_USN csv`

The resulting `usn-journal-case-output.csv` contains the error that the argument was invalid, followed by usage information. It does not contain validated USN records.

Therefore, no USN reason-flag analysis or correlation to the case files can be claimed from the current output.

## 10. MFT Status
The supplied evidence does not include a raw MFT acquisition or parsed MFT export. Consequently, file-reference numbers, parent references, and MFT timestamp correlations remain unavailable.
A suitable forensic acquisition or parsing method is required before USN-to-MFT correlation can be completed. Record the tool and version, source volume, acquisition method, output format, and collection limitations.

## 11. SHA-256 Evidence Manifest
The evidence manifest was generated for files under the case directory, excluding the manifest itself. It includes the case timestamps, baseline files, Sysmon event export, configuration captures, Registry export, USN artifacts, and the surviving test script.

The presence of a hash in the manifest supports later integrity comparison of that artifact. It does not prove that the artifact is complete or that its contents establish an event by themselves.

The manifest also includes `usn-journal-case-output.csv`, which is the output of a failed command. Its hash preserves that failure artifact; it does not convert it into valid USN evidence.

## 12. Confirmed Findings and Gaps
### Confirmed
- The case start and activity marker timestamps were recorded.
- Sysmon and Wazuh services were running in the baseline.
- The active Sysmon configuration was captured.
- Sysmon Event ID 11 confirmed creation of `case-document.ps1` and `case-copy.ps1`.
- Sysmon Event ID 1 recorded `whoami.exe` execution.
- The Registry final-state export showed `CaseStatus = Modified`.
- The NTFS USN journal configuration was queried.
- A SHA-256 manifest was generated.

### Not Established
- Valid USN journal records associated with the controlled activity.
- MFT file-reference and parent-reference information.
- A Sysmon Event ID 13 record for the controlled `CaseStatus` change.
- Confirmed Sysmon event evidence for the rename and deletion operations.
- Wazuh correlation for the controlled case events.
- A complete reconstruction of every file-system operation.

## 13. Conclusion
The evidence supports a preliminary reconstruction of the controlled process, file, and Registry activity. Sysmon records confirm the two file creations and the `whoami.exe` execution, while the Registry export confirms the final test value.
The USN collection is invalid because a placeholder was used as the starting USN, and MFT parsing has not been demonstrated. The final case report must retain these as explicit evidence gaps.
