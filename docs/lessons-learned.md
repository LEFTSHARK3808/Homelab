# Lessons learned

Things that broke, why, and what fixed them. Most of the learning in this lab lives here.

## Detection lab

| Problem | Cause | Fix |
|---|---|---|
| "The audit system is disabled"; auditd crash-loops | Audit isn't namespaced; an unprivileged LXC can't use it | Rebuilt the target as a VM |
| `qemu-guest-agent` hangs on enable | VM created without the agent device | `--agent enabled=1` + full `qm shutdown`/`qm start` (a guest reboot isn't enough) |
| SSH `Permission denied (publickey)` | Ubuntu cloud images are key-only | Add an ed25519 key to `authorized_keys` via the serial console |
| Universal Forwarder CLI asks for a "Splunk username" | The forwarder has its own local admin account | Use the forwarder credentials, not Splunk Web's |
| Pasted command block half-ran | An interactive login prompt consumed the following lines | Run interactive commands one at a time |
| `index=atomic` empty while forwarder is connected | `inputs.conf` was never created | Create it, restart the UF, look for `Adding watch on path` |
| Forwarder can't read `audit.log` | UF runs as `splunkfwd`, not root | Add `splunkfwd` to `adm`, set `log_group = adm` in `auditd.conf` so it survives rotation |
| `recon`/`download` keys firing with zero activity | `CONFIG_CHANGE` events from rule reloads carry the rule key | Pin key-based searches to `type=SYSCALL` |
| Snapshot missing the forwarder | Baseline taken before the UF install | Re-snapshot after each build stage |
| Events silently missing | Forwarding to an index that didn't exist | Create the index first; Splunk drops events for missing indexes without an error |

## Infrastructure

| Problem | Cause | Fix |
|---|---|---|
| Subnet routing gone after host reboot | `tailscale up` flags not persisted | systemd `tailscale-up.service` |
| Remote Mac can't reach LAN | Routes not accepted | `sudo tailscale up --accept-routes` |
| Half the VM disk unused | Ubuntu installer sizes `ubuntu-lv` at ~50% | Expand the LV to max during install |
| Docker containers hitting rlimit errors | Unprivileged LXC limits | Converted LXC 102 to privileged |
| Palworld saves not persisting | Files owned by root | `chown -R steam:steam` on the install dir |
