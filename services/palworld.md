# Palworld dedicated server

Self-hosted Palworld 1.0 server running as a boot-persistent systemd service with a nightly restart. **Currently stopped** — it's provisioned for 16 GB and can't share the host with the SIEM.

## Overview

| | |
|---|---|
| **OS** | Ubuntu Server 24.04 LTS |
| **Run-as user** | `steam` |
| **Install path** | `/home/steam/palworld-server` |
| **Steam app ID** | 2394010 (dedicated server, anonymous) |
| **Resources** | 4 vCPU · 16 GB RAM (no ballooning) · 8 GB swap · 40 GB SSD |
| **Ports** | `8211/udp` game (forwarded) · `27015/udp` query · `25575/tcp` RCON (**LAN only**) |

## Gameplay settings

Tuned for a small group of friends:

```ini
ServerPlayerMaxNum=10
ExpRate=1.5
PalCaptureRate=1.2
CollectionDropRate=1.5
WorkSpeedRate=1.5
DeathPenalty=Item
bIsPvP=False
AutoSaveSpan=15
```

## Service management

```bash
sudo systemctl start|stop|restart palworld
journalctl -u palworld -f
sudo ss -ulnp | grep 8211
```

The unit runs a SteamCMD update on every start, so a restart is also an update.

### Nightly restart

Palworld leaks memory over long runtimes. A systemd timer (`palworld-restart.timer`) restarts it daily at 05:00 UTC.

## Backups

`backup-palworld.sh` tars `Pal/Saved/SaveGames/` every 6 hours via cron and keeps the last 14 archives.

```cron
0 */6 * * * /home/steam/backup-palworld.sh >> /home/steam/backups/backup.log 2>&1
```

## Troubleshooting

| Symptom | Cause / fix |
|---|---|
| Server exits instantly | `steamclient.so` missing from `~/.steam/sdk64/` |
| `[S_API FAIL] ... before SteamAPI_Init` | Harmless startup noise |
| Outside players can't join, LAN works | Port forward must be UDP, or the public IP changed |
| Saves not persisting | Files owned by root → `chown -R steam:steam` |
| `steamcmd: command not found` | Use `/usr/games/steamcmd` |

## Security notes

- Only the game port is forwarded; RCON and SSH are LAN/tailnet only.
- Join and admin passwords live in a password manager, not in this repo.
