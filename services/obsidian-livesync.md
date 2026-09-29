# Obsidian LiveSync — self-hosted CouchDB

Obsidian vaults sync across devices through the **Self-hosted LiveSync** plugin, backed by CouchDB running in Docker on LXC 102.

## Components

| | |
|---|---|
| **Host** | LXC 102 (`docker`), privileged |
| **Container** | CouchDB, port `5984` |
| **Database** | `obsidian` |
| **Access** | LAN, or remote over Tailscale subnet routing |

## Notes

- CORS is enabled on CouchDB so the Obsidian client can talk to it directly.
- The LXC was converted from unprivileged to privileged to fix rlimit errors in containers.
- Remote sync only works while the client has Tailscale routes accepted (`tailscale up --accept-routes`).
- **This LXC is off-limits for attack tooling** — it holds the only copy of the synced data outside the clients.

## To do

- [ ] Put CouchDB behind Nginx Proxy Manager with a local hostname
- [ ] Scheduled backup of the CouchDB data volume
