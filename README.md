# home-dashboard

Aggregated camera dashboard for two Yi Home Dome Guard cameras, served from a Raspberry Pi and accessible remotely via Tailscale.

## Cameras

| Name       | IP             |
|------------|----------------|
| Front Door | 192.168.1.81   |
| Bedroom    | 192.168.1.86   |

## Access

- **Local network:** `http://192.168.1.217:8080`
- **Remote (Tailscale):** `http://raspberrypi.tail80333c.ts.net`

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

```sh
mkdir -p ~/dashboard
scp index.html cyrus@192.168.1.217:~/dashboard/
```

### 5. Start the HTTP server

```sh
cd ~/dashboard && nohup python3 -m http.server 8080 &
```

To auto-start on reboot, add to crontab (`crontab -e`):

```
@reboot cd /home/cyrus/dashboard && python3 -m http.server 8080 >> /home/cyrus/dashboard/server.log 2>&1
```

### 6. Port 80 redirect (so no :8080 in the URL)

```sh
TSIP=$(tailscale ip -4)
sudo iptables -t nat -A PREROUTING -i tailscale0 -d $TSIP -p tcp --dport 80 -j REDIRECT --to-port 8080

sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```

> **Important:** scope the rule to `-d $TSIP` to avoid intercepting port 80 traffic destined for the cameras on the same subnet.

### 7. Enable MagicDNS

In the [Tailscale admin console](https://tailscale.com/admin) → DNS → enable MagicDNS.

On your phone, enable "Accept routes" and "Use Tailscale DNS" in the Tailscale app.
