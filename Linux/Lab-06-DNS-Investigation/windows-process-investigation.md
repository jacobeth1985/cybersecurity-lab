# Case Study: Windows Network-to-Process Correlation

## Scenario

During a live Windows endpoint investigation, established HTTPS traffic included an unfamiliar process. The goal was to determine whether the process required escalation by correlating multiple independent sources of evidence.

This is a real endpoint investigation, but host-specific values such as usernames, hostnames, remote IP addresses, local ports, and exact PIDs have been removed. The only retained process detail is necessary to explain the outcome.

## Methodology

1. **Start with the network evidence.** Review active established TCP connections and identify the PID in `OwningProcess`.
2. **Identify the process.** Query the process name, command line when available, and parent PID.
3. **Investigate the parent.** Parent-child relationships add execution context but do not alone determine legitimacy.
4. **Map the PID to a service.** A `services.exe` parent makes a service-ownership query especially useful.
5. **Validate the executable path.** Review whether it is in an expected, protected software location or a suspicious user-writable location.
6. **Verify the digital signature.** Confirm the Windows signature status and expected publisher.
7. **Document an evidence-based assessment.** Use calibrated language such as *likely legitimate*, *suspicious*, or *confirmed malicious*; do not overstate what the evidence proves.

## Evidence Correlation

```text
Established TCP connection on remote port 443
  -> OwningProcess PID
  -> mc-fw-host.exe
  -> parent process: services.exe
  -> service: McAfee Framework Host (running)
  -> binary: C:\Program Files\McAfee\wps\...\mc-fw-host.exe
  -> Authenticode: Valid
  -> assessment: likely legitimate
```

## Findings

| Evidence area | Finding | Why it matters |
|---|---|---|
| Network | The process owned an established HTTPS connection. | HTTPS is common for both legitimate and malicious software; the port does not determine a verdict. |
| Process | `mc-fw-host.exe` was identified from the owning PID. | A filename is a lead, not a conclusion. |
| Parent | `services.exe` was the parent process. | This is compatible with normal Windows service execution. |
| Service | The PID mapped to the running **McAfee Framework Host** service. | This links the process to a named Windows service. |
| Path | The binary was under `C:\Program Files\McAfee\wps\...`. | This is consistent with installed software and less suspicious than a temporary or profile path. |
| Signature | Windows reported `Status : Valid`. | The executable's Authenticode signature was successfully verified. |

## Assessment

`mc-fw-host.exe` was assessed as **likely legitimate** because its service registration, parent process, installed-software path, and valid digital signature formed a consistent evidence set.

This assessment is deliberately not “100% safe.” In a production SOC, an analyst would preserve timestamps and relevant telemetry, compare the file hash with an approved baseline or vendor source, assess destination reputation, and escalate if other alerts or anomalous behavior contradicted the evidence.

## SOC Interview Takeaways

An interview-ready explanation:

> I correlated an established HTTPS connection to its owning PID, identified the process, traced its parent to `services.exe`, and mapped the PID to the McAfee Framework Host service. I then validated the binary's Program Files path and Authenticode signature. Because the evidence was consistent, I assessed the process as likely legitimate while noting that further telemetry could change that assessment.

Key principles demonstrated:

- DNS answers and live network connections are different kinds of evidence.
- A process name, parent, path, or port should never be evaluated in isolation.
- Blank path or command-line fields may be caused by permissions; record the limitation and use another data source.
- “Likely legitimate” is a defensible triage outcome when evidence is consistent but not exhaustive.
