# home-dashboard

Aggregated camera dashboard for two Yi Home Dome Guard cameras, served from a Raspberry Pi and accessible remotely via Tailscale.

## Cameras

| Name       | IP             |
|------------|----------------|
| Front Door | 192.168.1.81   |
| Bedroom    | 192.168.1.86   |

## Access

- **Local network:** `http://192.168.1.217:8080`
- **Remote (Tailscale):** `http://raspberrypi.tail80333c.ts.net:8080`

The Pi launcher at `http://192.168.1.217` (port 80) also links to this dashboard.

## Pi Setup

### 1. Install Tailscale

```sh
curl -fsSL https://tailscale.com/install.sh | sh
```

### 2. Enable IP forwarding

```sh
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### 3. Start Tailscale with subnet routing

```sh
sudo tailscale up --advertise-routes=192.168.1.0/24 --accept-routes
```

Authenticate via the printed URL, then approve the subnet in the [Tailscale admin console](https://tailscale.com/admin) under Machines → the Pi → Subnets.

### 4. Deploy the dashboard

The `home-server` repo (`cyrusxyl/home-server`) deploys this dashboard. It clones this repo to `~/workspace/home-dashboard` and runs it as the systemd unit `home-dashboard` on port 8080.

Push changes to this repo, then run on the Pi:

```sh
~/workspace/home-server/deploy.sh
```

Do not start the server with crontab or `nohup`. Do not add an iptables redirect for port 80. The `home-server` launcher uses port 80.

### 5. Enable MagicDNS

In the [Tailscale admin console](https://tailscale.com/admin) → DNS → enable MagicDNS.

On your phone, enable "Accept routes" and "Use Tailscale DNS" in the Tailscale app.
