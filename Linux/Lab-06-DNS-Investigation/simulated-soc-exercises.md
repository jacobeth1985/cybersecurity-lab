# Simulated SOC Process-Correlation Exercises

## Important Label

The scenarios in this file are **simulated training exercises**. They are not observations from the live McAfee Framework Host investigation documented in [windows-process-investigation.md](windows-process-investigation.md).

## Exercise: Suspicious PowerShell Launched by Microsoft Word

### Training alert

A simulated alert reports that `WINWORD.EXE` started `powershell.exe`, which then established an HTTPS connection.

### Investigation workflow

```text
Host
  -> user / session
  -> parent process: WINWORD.EXE
  -> child process: powershell.exe
  -> PID and command line
  -> destination IP and port
  -> related files, events, and reputation checks
  -> assessment and escalation decision
```

### Analyst questions

1. Which account and host were involved?
2. What was the full PowerShell command line, including any encoded content or download behavior?
3. Was Word expected to start PowerShell in this context?
4. What remote destination and port were contacted, and is the destination known or approved?
5. Are there related downloads, persistence mechanisms, security events, or endpoint detections?
6. Does the evidence support containment, escalation, or closure?

### Lesson

The process chain is a lead, not a verdict. Office applications starting scripting interpreters can be suspicious, but an analyst must validate context and collect corroborating evidence before declaring an incident.

## Contrast With the Live Endpoint Case

| Topic | Simulated alert | Live endpoint investigation |
|---|---|---|
| Initial concern | Word launching PowerShell over HTTPS | An unfamiliar service process with HTTPS traffic |
| Evidence source | Training scenario | Local Windows system queries |
| Required analysis | User context, command line, destination, related artifacts | Service, parent, path, and signature correlation |
| Outcome | Depends on collected evidence | `mc-fw-host.exe` assessed as likely legitimate |
