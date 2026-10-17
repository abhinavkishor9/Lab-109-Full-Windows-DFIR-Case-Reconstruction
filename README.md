# Lab 109 — Full Windows DFIR Case Reconstruction
## Repository Description
Reconstruct Windows endpoint activity through correlated Sysmon events, Registry evidence, NTFS USN journal records, Wazuh telemetry, and SHA-256 artifact preservation. Document process execution, file-system changes, collection failures, and forensic limitations in a unified investigation timeline.

## Overview
This lab combines multiple Windows digital forensics and incident response techniques into a single controlled case. The investigation examines endpoint identity, monitoring service state, process execution, file-system activity, Registry modifications, Sysmon telemetry, and NTFS journal information.
The case uses a dedicated workspace to generate known file and Registry activity. Sysmon and PowerShell provide supporting evidence, while a SHA-256 manifest records the integrity-checking values of the collected artifacts.
The investigation also demonstrates an important forensic limitation: executing a collection command does not guarantee that valid evidence was collected. The USN journal configuration query succeeded, but the subsequent journal-read command used a literal placeholder for the starting USN and returned an invalid-parameter message instead of journal records. MFT acquisition and parsing were not demonstrated in the supplied evidence.

## Lab Environment
| Component | Configuration |
|---|---|
| Endpoint | `DESKTOP-9MMM37V` |
| Operating system reported by baseline | `Windows 10 Pro` |
| Windows version field | `2009` |
| OS build | `26200` |
| Case directory | `C:\WindowsDFIRCase109` |
| Evidence directory | `C:\WindowsDFIRCase109\Evidence` |
| Test directory | `C:\WindowsDFIRCase109\TestArea` |
| Sysmon version | `15.21` |
| Sysmon service | `Sysmon64` |
| Wazuh service | `WazuhSvc` |
| Wazuh agent ID | `001` |
| Primary telemetry | Sysmon Operational and Wazuh |
| File system artifact | NTFS USN Change Journal |

The baseline output reported `Windows 10 Pro`, version `2009`, and build `26200`. Record these as the values returned by the collection command; verify the operating-system naming separately if required.

## Investigation Scenario
A DFIR analyst needs to reconstruct a controlled sequence of endpoint activity using multiple evidence sources. The analyst creates a dedicated test directory, generates a PowerShell file, appends content, copies and renames the copy, and removes the renamed file. The analyst also creates a test Registry key and modifies a value.
Sysmon records file-creation and process-execution events during the case window. A `whoami.exe` process execution is observed, and a Registry value-set event is present in the export. However, the displayed Registry event references unrelated TrustedInstaller activity rather than the controlled case key.
The analyst queries the NTFS USN journal and records its configuration, but the journal-read command fails because the literal placeholder `0xYOUR_RECORDED_USN` was used as an argument. The saved output therefore contains command usage information instead of validated USN records. The complete MFT is not acquired or parsed in the supplied evidence.
The task is to distinguish confirmed test activity from unrelated background events, identify the collection gaps, and determine what additional evidence is required before claiming a complete timeline reconstruction.

## Investigation Objectives
- Record the endpoint baseline and monitoring service states.
- Preserve the active Sysmon configuration and Wazuh agent configuration hash.
- Reconstruct the known file-creation, append, copy, rename, and deletion sequence.
- Examine Sysmon process and file events within the case window.
- Validate the controlled Registry value and investigate the available Registry telemetry.
- Record NTFS USN journal configuration and validate journal-read output.
- Identify whether MFT metadata has been acquired and parsed.
- Compare local endpoint evidence with available Wazuh telemetry.
- Generate SHA-256 hashes for the collected artifacts.
- Document missing records, unrelated events, command failures, and remaining forensic questions.

## Controlled Activity
The case performed the following file operations in `C:\WindowsDFIRCase109\TestArea`:
1. Created `case-document.ps1`.
2. Appended additional content to the file.
3. Copied it to `case-copy.ps1`.
4. Renamed the copy to `case-renamed.ps1`.
5. Deleted `case-renamed.ps1`.

The Registry test used `HKCU:\Software\WindowsDFIRCase109` and modified the `CaseStatus` value. The final Registry export displayed `Modified`.

A separate `whoami.exe` execution was also recorded to provide a known process-execution reference point.

## Evidence Inventory
| Artifact | Purpose |
|---|---|
| `case-start.txt` | Records the case start time |
| `system-baseline.csv` | Records endpoint details returned by PowerShell |
| `monitoring-services-baseline.csv` | Captures Sysmon and Wazuh service state |
| `sysmon-config-baseline.txt` | Captures the current Sysmon configuration output |
| `wazuh-config-baseline-hash.csv` | Records the Wazuh agent configuration hash |
| `file-created-time.txt` | Records a timestamp during the file test |
| `process-test-start.txt` | Records the process-test start time |
| `registry-final-state.csv` | Captures the final test Registry value |
| `sysmon-case-events.csv` | Contains the local Sysmon event export |
| `usn-journal-case-query.txt` | Captures the USN journal configuration |
| `usn-journal-case-output.csv` | Contains the failed USN-read command output |
| `evidence-hashes.csv` | Contains SHA-256 hashes for the collected artifacts |
| `TestArea\case-document.ps1` | Surviving controlled test file |

## Preliminary Findings
- The baseline reported `Sysmon64` and `WazuhSvc` as Running with Automatic startup.
- The Sysmon baseline reported version `15.21`, active configuration `C:\Tools\Sysmon\sysmonconfig.xml`, and configuration hash `D4290CD763B7C363A7C0FEEC5873A43906018AAA00EF95A8F9AFF26D69942DDB`.
- The Sysmon event export contained 115 records: 65 Event ID 1 records, 49 Event ID 11 records, and one Event ID 13 record.
- Event ID 11 records confirmed creation of `case-document.ps1` and `case-copy.ps1`.
- An Event ID 1 record confirmed execution of `whoami.exe`.
- The sole Event ID 13 record in the supplied local export described TrustedInstaller activity, not the controlled `CaseStatus` modification.
- The final Registry export showed `CaseStatus` set to `Modified`.
- The USN journal configuration query succeeded, but the journal-read command failed because an example placeholder was used as the `startUsn` value.
- No parsed MFT metadata was included in the supplied evidence.
- A SHA-256 manifest was generated for the case files, including the surviving test file.

## Investigation Limitations
The available evidence does not establish a complete USN-to-MFT correlation. The current USN output contains an invalid-parameter message rather than journal records, so the journal must be collected again with a valid USN value.
The Sysmon export did not show the controlled Registry value in a matching Event ID 13 record. The single exported Registry event relates to TrustedInstaller activity and must not be used to attribute the controlled Registry change to that process.
Although the commands for rename and deletion were executed, the supplied Sysmon export does not contain Event IDs 23 or 26 or a confirmed event explicitly documenting the rename and deletion. Those operations are known from the controlled commands, but their telemetry correlation remains unverified.
The supplied evidence also does not establish whether any of the controlled events were successfully correlated with Wazuh records. Local event presence and Wazuh alert visibility must be evaluated separately.

## Conclusion
The lab established a controlled Windows case workspace, preserved baseline and configuration artifacts, generated file and Registry activity, collected Sysmon events, queried the USN journal configuration, and produced a SHA-256 manifest.
A preliminary activity sequence can be documented from the known operations and relevant Sysmon records. However, a full Windows DFIR reconstruction remains incomplete because the USN read failed, MFT metadata was not acquired or parsed, and the Registry change was not correlated with a matching Sysmon event.
The next stage is to rerun the USN collection with a valid starting USN, acquire and parse MFT metadata, inspect complete event records, and correlate the findings using timestamps and file references.

## Key Learning
**A successful command, an exported file, and a validated forensic artifact are three different things.** A defensible reconstruction requires examination of the actual output and explicit documentation of the evidence that remains missing.
