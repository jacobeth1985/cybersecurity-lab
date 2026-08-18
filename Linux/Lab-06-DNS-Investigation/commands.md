# Command Reference: DNS and Windows Endpoint Correlation

Run commands only on systems you are authorized to investigate. Commands and results vary by operating-system version, permissions, and the state of the host at collection time.

## DNS in Ubuntu / WSL

```bash
nslookup google.com
nslookup github.com
nslookup -type=mx gmail.com
```

`nslookup` maps a name to DNS data. It does not show which remote IPs a local process is actively using.

## Find established HTTPS connections in Windows PowerShell

```powershell
Get-NetTCPConnection -State Established |
  Where-Object { $_.RemotePort -eq 443 } |
  Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess
```

Record the `OwningProcess` value (the PID). Avoid publishing raw output if it includes internal IP addresses or other sensitive details.

## Identify a process and its parent

```powershell
Get-CimInstance Win32_Process |
  Where-Object { $_.ProcessId -in <PID1>, <PID2> } |
  Select-Object ProcessId, ParentProcessId, Name, CommandLine
```

For a single process:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PID>" |
  Format-List ProcessId, ParentProcessId, Name, ExecutablePath, CommandLine
```

To investigate its parent, replace `<PARENT_PID>` with the captured parent process ID:

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = <PARENT_PID>" |
  Select-Object ProcessId, ParentProcessId, Name, CommandLine
```

## Retrieve an executable path

```powershell
Get-Process -Id <PID> | Select-Object Id, ProcessName, Path
```

`ExecutablePath`, `CommandLine`, and `Path` can be blank because of access controls or process-protection behavior. A missing value is an investigation gap, not proof of malicious activity.

## Map a process to a Windows service

```powershell
Get-CimInstance Win32_Service |
  Where-Object { $_.ProcessId -eq <PID> } |
  Select-Object Name, DisplayName, State, StartName, PathName
```

Review whether the service name, account, start state, and binary path are consistent with the claimed software.

## Verify an Authenticode signature

```powershell
Get-AuthenticodeSignature "C:\Path\To\Executable.exe" |
  Select-Object Status, StatusMessage, SignerCertificate
```

`Status : Valid` supports that Windows successfully verified the file's signature. It should be considered alongside the process tree, service configuration, path, hash, and available endpoint telemetry.
