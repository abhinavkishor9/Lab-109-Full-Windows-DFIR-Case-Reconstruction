# Lab-109-Full-Windows-DFIR-Case-Reconstruction
## Overview
Full Windows Digital Forensics and Incident Response (DFIR) case reconstruction brings together multiple evidence sources to determine what happened, when it happened, which processes or files were involved, and what the evidence can actually prove.

### Evidence sources for the case
#### 1. Windows event logs and Sysmon
Examine process creation, file activity, Registry events, and monitoring-service changes.

#### 2. NTFS artifacts
Use USN journal records and parsed MFT metadata to investigate file creation, modification, renaming, and deletion.

#### 3. Registry and configuration evidence
Examine controlled Registry changes, Sysmon configuration versions, and Wazuh agent configuration integrity.

#### 4. Wazuh telemetry
Check whether locally generated events reached the manager, were processed by rules, and became searchable alerts.

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

## Lab Scenario

A Windows endpoint investigation requires more than identifying a single suspicious process or reviewing an isolated security event. Investigators must establish the sequence of activity, preserve supporting artifacts, and determine whether the available evidence is sufficient to connect individual events into a defensible timeline.

In this lab, a controlled investigation is performed on a Windows system using Sysmon telemetry, Registry state, NTFS metadata, and available endpoint monitoring information. The objective is to reconstruct a sequence of test activities involving file creation, copying, renaming, deletion, Registry modification, and process execution while preserving the collected evidence for later validation.

### Investigation Activities

The investigation follows a controlled sequence of activities:

- **Baseline collection:** Record system information, monitoring service states, and relevant configuration hashes before examining the test activity.
- **File-system activity:** Create a test PowerShell script, modify its contents, create a copy, rename the copy, and subsequently delete the renamed file.
- **Process execution:** Execute a benign identity-discovery command using `whoami.exe` and examine its associated Sysmon process-creation record.
- **Registry modification:** Change a controlled Registry value and inspect the final state alongside available Registry-related telemetry.
- **NTFS artifact collection:** Examine the USN Journal configuration and attempt to retrieve journal records that may help reconstruct file-system changes.
- **Evidence preservation:** Export relevant event records, maintain an inventory of collected artifacts, and generate SHA-256 hashes to support integrity verification.

### Investigation Challenge

The collected evidence does not automatically provide a complete reconstruction. Sysmon records can establish that particular events were observed, but they do not necessarily prove every action performed during the test. Similarly, a Registry event occurring near the investigation start time cannot be attributed to the controlled Registry modification without supporting evidence.

The initial USN Journal collection also contains an invalid placeholder parameter rather than usable journal records. Consequently, the available output cannot establish a complete file-system timeline. MFT parsing and a successful extraction of relevant USN Journal records remain additional investigative requirements.

### Expected Investigation Approach

The investigator must correlate timestamps, process identifiers, file paths, Registry observations, and available system information without treating temporal proximity as proof of causation. Each finding should be classified according to the strength of its supporting evidence, with collection errors and unresolved questions documented explicitly.

The final reconstruction should explain what the artifacts confirm, what remains uncertain, and which additional collection or analysis steps are necessary to strengthen the investigation. The emphasis is on evidence-driven Windows DFIR rather than forcing every artifact into a single narrative.


## Lab Objectives

The objective of this lab is to reconstruct a controlled Windows endpoint activity sequence by collecting and examining host artifacts, process execution records, file-system activity, Registry state, and available NTFS metadata. The investigation focuses on establishing a defensible timeline while distinguishing directly observed evidence from events that cannot be reliably correlated.

By completing this lab, the investigator should be able to:

- **Establish a forensic baseline:** Record relevant system information, monitoring service states, and configuration hashes before analyzing the test activity.
- **Investigate process execution:** Examine Sysmon Event ID 1 records to identify executed processes, their parent processes, associated users, timestamps, and available executable hashes.
- **Reconstruct file-system activity:** Analyze Sysmon Event ID 11 records and supporting file-system artifacts to identify file creation and copying activity within the controlled test directory.
- **Validate Registry changes:** Examine the final state of the test Registry key and determine whether the available telemetry directly supports attribution of the change to the controlled activity.
- **Assess NTFS USN Journal evidence:** Inspect the journal configuration and attempt to retrieve relevant records, documenting command errors and collection limitations where applicable.
- **Develop a unified investigation timeline:** Align recorded timestamps from the test markers, process execution records, file activity, and other available artifacts without assuming that temporal proximity proves causation.
- **Preserve evidence integrity:** Generate SHA-256 hashes for collected artifacts and maintain an inventory that supports subsequent validation.
- **Evaluate telemetry coverage:** Determine which investigation objectives are supported by the collected Sysmon records and identify areas where Windows event logs, Wazuh telemetry, or additional artifacts are missing.
- **Recognize correlation limitations:** Avoid attributing unrelated Registry events to the test activity and distinguish confirmed observations from unresolved findings.
- **Identify further forensic requirements:** Document additional collection or parsing needed to strengthen the reconstruction, including successful USN Journal record extraction and MFT analysis.

### Expected Learning Outcome

The investigator should be able to produce an evidence-based reconstruction of a controlled Windows activity sequence, explain how individual artifacts support or limit each finding, and document unresolved gaps without presenting incomplete evidence as a confirmed conclusion.

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
