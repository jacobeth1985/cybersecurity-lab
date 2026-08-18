# Lab 06: DNS and Windows Endpoint Investigation

## Objective

Investigate how DNS translates names into IP addresses, then demonstrate how a SOC analyst can correlate a live Windows network connection to its owning process, service, executable path, and digital signature.

> **Scope note:** The DNS exercises were completed in Ubuntu on WSL. The endpoint investigation was performed on a live Windows workstation. Simulated alert exercises are documented separately and are not evidence from the live endpoint.

## Tool Used

- Ubuntu on WSL
- `nslookup` from the `dnsutils` package
- Windows PowerShell
- Windows networking, process, service, and signature inspection cmdlets

## DNS Lookup: google.com

Command used:

    nslookup google.com

## What I Learned

DNS stands for Domain Name System. It translates a human-readable domain name, such as `google.com`, into IP addresses that computers use to communicate.

The result was marked **Non-authoritative answer**. This means the DNS server provided a cached response rather than answering directly as the authoritative DNS server for Google.

## DNS Lookup: github.com

Command used:

    nslookup github.com

## Result

GitHub resolved to this IPv4 address:

    20.87.245.0

## Observation

A single domain can resolve to one or more IP addresses. These addresses can change over time because large services use distributed infrastructure.

DNS resolution alone does not prove that a workstation is communicating with an IP address. A current connection must be observed and then correlated to an owning process.

## MX Record Lookup: gmail.com

Command used:

    nslookup -type=mx gmail.com

## Result

Gmail returned these mail exchangers:

- Priority 5: `gmail-smtp-in.l.google.com`
- Priority 10: `alt1.gmail-smtp-in.l.google.com`
- Priority 20: `alt2.gmail-smtp-in.l.google.com`
- Priority 30: `alt3.gmail-smtp-in.l.google.com`
- Priority 40: `alt4.gmail-smtp-in.l.google.com`

## What I Learned

MX means **Mail Exchanger**. MX records tell other mail systems which servers should receive email for a domain.

The priority number matters: lower numbers have higher priority. Gmail’s primary mail server has priority 5, while the `alt` servers provide backup options.

---

## Windows SOC Investigation Extension

This extension uses the following evidence chain:

```text
DNS name
  -> IP address
  -> active network connection
  -> OwningProcess / PID
  -> process and parent process
  -> Windows service
  -> executable path
  -> Authenticode digital signature
  -> assessment
```

The key lesson is to corroborate evidence rather than trust a filename, a port, or a single tool result.

### Live endpoint case: McAfee Framework Host

An established HTTPS connection was correlated to `mc-fw-host.exe` (PID values are intentionally omitted from this portfolio write-up). Investigation established that the process:

- Had `services.exe` as its parent, consistent with a service-hosted process.
- Was registered as the running **McAfee Framework Host** Windows service.
- Ran from an expected `C:\Program Files\McAfee\...` location rather than a user-writable temporary directory.
- Had a **Valid** Authenticode signature.

**Assessment: likely legitimate.** This is an evidence-based assessment, not a claim that any process is permanently safe. A full enterprise investigation would also consider host baselines, hashes, telemetry history, vendor reputation, and organizational policy.

Read the full case study in [windows-process-investigation.md](windows-process-investigation.md). A reusable command reference is in [commands.md](commands.md), and simulated SOC process-correlation exercises are clearly separated in [simulated-soc-exercises.md](simulated-soc-exercises.md).

## Skills Demonstrated

- DNS and MX record lookup with `nslookup`
- Difference between DNS resolution and observed network communication
- Correlation of TCP connections to `OwningProcess` / PID
- Process and parent-process investigation
- Windows service ownership and executable-path review
- Authenticode signature verification
- Evidence-based SOC triage and documentation

## Evidence and Privacy

Live screenshots are not included because they may disclose usernames, hostnames, internal addresses, process IDs, or other endpoint details. See [screenshots/README.md](screenshots/README.md) for a safe approach to adding sanitized evidence later.
