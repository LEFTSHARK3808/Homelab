# Detection Engineering Lab

A purple-team loop: run a MITRE ATT&CK technique with **Atomic Red Team**, see whether **Splunk** catches it, and write or tune a detection for the gap.

## Architecture

```mermaid
flowchart LR
    subgraph T["VM 107 — atomic-lab (Ubuntu 24.04)"]
        ART[Atomic Red Team<br/>PowerShell 7] -->|executes| K[Linux kernel]
        K -->|audit events| AD[auditd<br/>16 rules]
        AD --> LOG[/var/log/audit/audit.log/]
        UF[Splunk Universal Forwarder<br/>runs as splunkfwd]
        LOG --> UF
        AUTH[/auth.log · syslog/] --> UF
    end
    UF -->|TCP 9997| IDX
    subgraph S["VM 105 — Splunk Enterprise 10.4.3"]
        IDX[(index=atomic)] --> TA[Splunk_TA_nix<br/>field extraction]
        TA --> DET[Saved searches<br/>& alerts]
        ESCU[ES Content Update] --> DET
    end
```

**Why a VM?** The Linux audit subsystem isn't namespaced, so auditd can't run in an unprivileged container. A small VM gets its own kernel.

**Snapshots:** `clean-baseline` (auditd + forwarder) and `art-ready` (+ PowerShell + ART). Every test ends in `-Cleanup` or a rollback to `art-ready`.

## The loop

```powershell
Invoke-AtomicTest T1087.001 -ShowDetailsBrief   # read what it does first
Invoke-AtomicTest T1087.001 -CheckPrereqs
Invoke-AtomicTest T1087.001
# → search Splunk: did ESCU fire? did my detection fire?
Invoke-AtomicTest T1087.001 -Cleanup
```

For every run I record: **technique → did existing content alert → what I searched → detection written → what I tuned**.

## Coverage

| Technique | Name | Telemetry | Status |
|---|---|---|---|
| [T1087.001](detections/T1087.001.md) | Local Account Discovery | `EXECVE` args | ✅ Detection written & validated |
| T1053.003 | Cron | `persistence` key writes | 🔜 Next |
| T1003.008 | /etc/passwd & /etc/shadow | `identity` key read of `/etc/shadow` | Planned |
| T1136.001 | Create Account | `useradd` execve + `identity` writes | Planned |
| T1059.004 | Unix Shell | `sh -c` execve chains | Planned |
| T1548.001 | Setuid and Setgid | `chmod u+s` | Planned |
| T1070.002 | Clear Linux Logs | truncation in `/var/log` | Planned |
| T1046 | Network Service Discovery | `nmap` execution | Planned |
| T1105 | Ingress Tool Transfer | `curl` / `wget` execution | Planned |
| T1057 | Process Discovery | `ps`, `top` enumeration | Planned |

## Config in this folder

| File | What it is |
|---|---|
| [`config/atomic.rules`](config/atomic.rules) | auditd rules — the telemetry every detection depends on |
| [`config/inputs.conf`](config/inputs.conf) | Universal Forwarder inputs |
| [`config/savedsearches.conf`](config/savedsearches.conf) | Detections as Splunk saved searches |

## Working notes on the data

- One `execve` produces several linked records: `SYSCALL` (who ran what binary), `EXECVE` (full arguments), `PATH` (files touched).
- `auid` survives `sudo`, so it attributes activity to the human who logged in. `auid=1000 uid=0` = a user running as root through sudo.
- Rule reloads log `CONFIG_CHANGE` events carrying each rule's key. **Always pin key-based searches to `type=SYSCALL`**, or every `augenrules --load` looks like an attack.
- `/etc/shadow` gets a read watch (`-p rwa`) because legitimate reads are rare. `/etc/passwd` doesn't — it's read constantly, so passwd enumeration is detected from command lines instead.
