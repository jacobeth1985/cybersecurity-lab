# Lab 01: Windows Event Log Investigation

## Objective

Use Windows event logs to investigate authentication activity, credential access, service-state changes, and process creation. Document findings using evidence-based SOC assessment language.

> **Scope:** This lab was completed on a personal Windows endpoint with administrative access. Endpoint-specific identifiers, event messages, usernames, hostnames, IP addresses, process IDs, and logon IDs are intentionally omitted.

## Tools and Data Sources

- Windows PowerShell (elevated)
- Windows Security event log
- Windows System event log
- `Get-WinEvent`
- `auditpol`

## Investigation Workflow

```text
Confirm log availability
  -> review recent event IDs
  -> identify authentication and credential events
  -> investigate event context and frequency
  -> review system service-state events
  -> enable and validate process-creation auditing for future activity
  -> document the assessment and limitations
```

## Log Availability

The Security log was enabled and contained 34,059 records at the time of inspection. Some protected logs cannot be queried from a standard PowerShell session; opening PowerShell as Administrator allowed access to the Security log.

```powershell
Get-WinEvent -ListLog Security |
  Select-Object LogName, IsEnabled, RecordCount, FileSize, MaximumSizeInBytes |
  Format-List
```

## Findings

| Event ID | Log | Observation | Assessment |
|---:|---|---|---|
| `4634` | Security | A logon session ended. | Normal session-lifecycle activity when viewed in isolation. |
| `5379` | Security | Credential Manager recorded a read of stored credentials. | Requires context; legitimate Windows components and applications can access saved credentials. Not proof of credential theft alone. |
| `4625` | Security | One failed logon was recorded in the previous seven days. Failure reason: unknown username or bad password. Logon type: `2` (local interactive). | Likely benign and isolated; consistent with an incorrect local sign-in attempt. No repeated-failure pattern was found. |
| `7036` | System | A `VfpExt` provider service/filter restart state was observed. | Service lifecycle activity; not suspicious without an alert, failure pattern, or related anomalous activity. |
| `4688` | Security | Process creation was successfully recorded after auditing was enabled. A Notepad launch was correlated to `explorer.exe`. | Expected desktop activity and successful validation of event collection. |

## Authentication Investigation

The following query identified failed logons:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625} -MaxEvents 5 |
  Select-Object TimeCreated, Id, ProviderName |
  Format-Table -AutoSize
```

The event detail reported a bad username or password with **Logon Type 2**. Logon Type 2 represents an interactive sign-in at the local console. A count of one failed event during the preceding seven days did not support a pattern of password spraying, brute force, or repeated access attempts.

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4625;StartTime=(Get-Date).AddDays(-7)} -MaxEvents 100 |
  Measure-Object |
  Select-Object -ExpandProperty Count
```

## Process-Creation Auditing

Initial audit policy showed **No Auditing** for Process Creation, so historical process-launch events were unavailable. Successful process-creation auditing was enabled to collect future Security event `4688` records.

```powershell
auditpol /set /subcategory:"Process Creation" /success:enable
auditpol /get /subcategory:"Process Creation"
```

This policy change only records future launches; it cannot reconstruct earlier process history. It also increases Security-log volume and should be enabled on organizational systems only with authorization.

### Validation Test

Notepad was opened through the Windows desktop interface. Event `4688` confirmed the expected parent-child relationship:

```text
explorer.exe
  -> notepad.exe
  -> Security event 4688
```

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security';Id=4688} -MaxEvents 50 |
  Where-Object { $_.Message -match '(?i)\\notepad\.exe' } |
  Select-Object -First 1 |
  Format-List TimeCreated, Message
```

## SOC Lessons

- Event IDs are investigative leads, not verdicts.
- A single `4625` failure should be assessed for frequency, logon type, and context before escalation.
- Credential-related event `5379` needs corroboration from process, user, timing, and endpoint telemetry.
- Process-creation event `4688` supports historical parent-child process correlation when auditing is enabled.
- Protected event logs require appropriate administrative permissions.
- Record evidence, limitations, and rationale; avoid asserting that normal-looking activity is “100% safe.”

## Interview-Ready Summary

> I verified that the Windows Security log was enabled, reviewed recent authentication and credential events, and investigated a failed logon. The failure was a single local interactive event caused by a bad username or password, with no repeated failures in the prior seven days, so I assessed it as likely benign. I then enabled process-creation auditing in my lab, validated event 4688 by launching Notepad, and correlated `explorer.exe` as the parent process.

## Cleanup Option

To stop collecting future successful process-creation events on a personal lab system:

```powershell
auditpol /set /subcategory:"Process Creation" /success:disable
```

Do not disable auditing when it is required by organizational policy.
