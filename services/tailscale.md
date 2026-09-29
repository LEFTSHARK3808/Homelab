# Tailscale — remote access

`hostserver` is a Tailscale **subnet router** and **exit node**, so any device on the tailnet can reach the whole LAN without exposing ports to the internet.

## Setup

```bash
# on hostserver
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding=1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

sudo tailscale up --advertise-routes=192.168.4.0/22 --advertise-exit-node --ssh
```

Approve the route and exit node in the Tailscale admin console.

## Persisting across reboots

Subnet routing dropped after reboots, so a oneshot unit re-applies it:

```ini
# /etc/systemd/system/tailscale-up.service
[Unit]
Description=Re-apply Tailscale subnet router + exit node
After=tailscaled.service network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/bin/tailscale up --advertise-routes=192.168.4.0/22 --advertise-exit-node --ssh

[Install]
WantedBy=multi-user.target
```

## Clients

On macOS/Linux clients, routes aren't accepted by default:

```bash
sudo tailscale up --accept-routes
```

## Checks after a reboot

```bash
tailscale status
sysctl net.ipv4.ip_forward        # must be 1
```
