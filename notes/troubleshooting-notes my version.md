# Troubleshooting Notes
## 1. USN Journal Read Returned an Invalid Parameter
### Observed behavior
The file `usn-journal-case-output.csv` contains:
`startUsn=0xYOUR_RECORDED_USN is an invalid parameter.`
The output then displays the syntax for `fsutil usn readJournal`.
### Cause
The example placeholder `0xYOUR_RECORDED_USN` was entered literally instead of a valid USN value.
### Corrected collection
The query output recorded:
`First Usn : 0x00000018dd000000`
Use that actual hexadecimal value as the starting point:
```powershell
$CaseRoot = "C:\WindowsDFIRCase109"
$EvidencePath = Join-Path $CaseRoot "Evidence"
$UsnStart = "0x00000018dd000000"
$UsnOutput = Join-Path $EvidencePath "usn-journal-case-output-corrected.csv"

& fsutil.exe usn readjournal C: "startUsn=$UsnStart" csv |
    Out-File -LiteralPath $UsnOutput -Encoding utf8

Get-Item -LiteralPath $UsnOutput |
    Select-Object FullName, Length, LastWriteTime

Get-Content -LiteralPath $UsnOutput -TotalCount 20
```
This reads currently available journal records from the recorded first USN. The journal may be large. Validate that the output contains actual records rather than usage text before treating it as evidence.
The starting USN was not captured immediately before the controlled file operations, so the corrected read can be used to search available records, but the investigation should not claim that it has a precisely isolated pre-test-to-post-test delta.
Do not overwrite the original failed output. Retain it as evidence of the collection error and use the new filename for the corrected collection.
## 2. MFT Metadata Has Not Been Acquired
### Observed behavior
The supplied evidence contains a USN journal configuration query, a failed journal output, and normal file artifacts. It does not contain a raw MFT acquisition or parsed MFT export.
### Recommended action
Use a suitable, trusted NTFS forensic acquisition or parser and document:
- Tool name and version.
- Source volume and acquisition method.
- Collection time and timezone.
- Output file and format.
- Errors, access restrictions, and relevant limitations.
Do not claim MFT file-reference or parent-reference correlations until the records have been acquired and parsed.
## 3. Sysmon Registry Event Does Not Match the Test Registry Key
### Observed behavior
The case Registry export shows `CaseStatus = Modified`. The sole Event ID 13 record in `sysmon-case-events.csv` targets a TrustedInstaller Registry path instead of the case Registry key.
### Interpretation
The exported event documents unrelated Registry activity. It cannot be used as direct evidence of the controlled value change.
### Troubleshooting
Search the local log for events referencing the controlled Registry path:
```powershell
Get-WinEvent -FilterHashtable @{
    LogName   = "Microsoft-Windows-Sysmon/Operational"
    Id        = 12, 13, 14
    StartTime = [datetime]::Parse(
        (Get-Content "C:\WindowsDFIRCase109\Evidence\case-start.txt" -Raw)
    )
} -ErrorAction SilentlyContinue |
    Where-Object {
        $_.Message -match "WindowsDFIRCase109|CaseStatus"
    } |
    Select-Object TimeCreated, Id, RecordId, Message |
    Format-List
```
If no matching events are found, inspect the active Sysmon Registry event filtering configuration and document the absence. Do not alter the configuration merely to make the previous activity appear retroactively.
## 4. Rename and Delete Events Were Not Confirmed
The controlled commands renamed `case-copy.ps1` to `case-renamed.ps1` and deleted the renamed copy. However, the supplied Sysmon export contained Event IDs 1, 11, and 13 only. It did not provide a confirmed event for the test file's rename or deletion.
### Troubleshooting
Inspect the full Sysmon event export for the filenames and check any other relevant telemetry collected during the same window. Event ID 11 records file creation; it does not by itself establish a rename or deletion.
If the relevant events are absent, report the operations as actions performed in the controlled script, not as independently corroborated Sysmon events.
## 5. System Baseline Reports Windows 10 Pro
The recorded `system-baseline.csv` reports:
- `WindowsProductName`: `Windows 10 Pro`
- `WindowsVersion`: `2009`
- `OsBuildNumber`: `26200`
Preserve these values as captured from `Get-ComputerInfo`. If the report must describe the endpoint as Windows 11, verify that identification using a separate source rather than silently changing the collected fields.
## 6. File Timeline and Timezone Handling
The case timestamps were captured in ISO 8601 format with `+05:30`, while Sysmon event messages include UTC timestamps.
For example, the local Sysmon record for `case-document.ps1` shows local time around `06:16:18` and UTC time around `00:46:18`.
Convert all timestamps to a common timezone before merging the timeline. Do not use screenshot order as a substitute for event timestamps.
## 7. Wazuh Evidence Is Not Equivalent to Local Evidence
The supplied evidence establishes local Sysmon activity, but it does not provide a complete Wazuh query result correlated to this case window.
If checking Wazuh:
1. Use the existing `wazuh-alerts-*` data view.
2. Set the time window to include the case activity.
3. Search for the controlled paths and process name using confirmed indexed fields.
4. Inspect complete event JSON when a query fails or returns no results.
5. Distinguish absence from an alerts index from absence in the underlying endpoint log.
Do not claim that Wazuh received or alerted on a specific case event unless the matching Wazuh document is available.
## 8. Evidence Hash Manifest
The `evidence-hashes.csv` manifest was generated across the case root, excluding itself. It includes evidence files and the surviving controlled test script.
If any evidence file is corrected or added, preserve the original manifest separately when historical integrity comparison is important, then generate an updated manifest with a new name.
The manifest for a failed USN command remains useful as an integrity value for that failed-output artifact. It does not establish successful USN collection.
