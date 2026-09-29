# Architecture

## Host

| | |
|---|---|
| **Hypervisor** | Proxmox VE on Debian |
| **Hostname** | `hostserver` |
| **RAM** | 15 GB |
| **Subnet** | `192.168.4.0/22` |

## Storage

| Pool | Type | Size | Use |
|---|---|---|---|
| `local` | dir | ~94 GB | ISOs, templates, `vzdump` backups |
| `local-lvm` | LVM-thin | ~338 GB | VM disks. **Snapshots supported** — the detection lab depends on this. |
| `local-2tbhd` | LVM (thick) | ~1.8 TB | Bulk storage. **No snapshots**, so VMs here are backed up with `vzdump --mode stop`. |

## Resource budget

RAM is the binding constraint, so workloads are mutually exclusive by design.

| Running | ~RAM |
|---|---|
| ollama, docker, Valheim | ~4 GB |
| Splunk (VM 105) | 8 GB |
| atomic-lab (VM 107) | 2 GB |
| Host overhead | ~1 GB |
| **Total** | **~15 GB of 15** |

Rules I follow:

- **Splunk and Wazuh never run together.** Wazuh (8 GB) is stopped while Splunk is the active SIEM.
- **Palworld (16 GB) only runs when the SIEM is down.**
- Large Ollama models are not loaded while Splunk is indexing — it pushes the host into swap.

## Remote access — Tailscale

```mermaid
flowchart LR
    Laptop[MacBook<br/>--accept-routes] -->|tailnet| HS[hostserver<br/>subnet router + exit node]
    HS --> LAN[192.168.4.0/22]
```

- `hostserver` advertises `192.168.4.0/22` and acts as an exit node.
- A systemd unit (`tailscale-up.service`) re-applies the subnet/exit-node flags on boot.
- IP forwarding is persisted in `/etc/sysctl.conf` (`net.ipv4.ip_forward=1`, `net.ipv6.conf.all.forwarding=1`).
- Tailscale SSH is enabled; no ports are forwarded to the internet for management.

## Design decisions

| Decision | Reason |
|---|---|
| atomic-lab is a **VM**, not an LXC | The Linux audit subsystem isn't namespaced. An unprivileged LXC can't load audit rules; a privileged one would audit the whole host. |
| Attack tooling never runs on LXC 102 or the host | 102 holds CouchDB data; the host runs everything. |
| Splunk runs as the `splunk` user | Splunk 10.x refuses to start as root. |
| Management only over Tailscale | No SSH or admin ports exposed to the internet. |
