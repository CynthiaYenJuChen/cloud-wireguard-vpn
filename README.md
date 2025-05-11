📡 GCP VM: WireGuard VPN Setup Guide
This is a step-by-step guide to setting up a lightweight WireGuard VPN server on Google Cloud Platform (GCP).
Perfect if you want your phone or laptop to connect securely and tunnel traffic through a GCP IP (e.g., for faster international access or secure browsing).

✅ Who is this for?
You have a GCP account (Free Tier works)

You're using Ubuntu/Debian (default GCP image)

Your goal: use your GCP IP securely from other devices (phone, laptop)

🔧 0. Before You Begin
GCP account activated with billing enabled (Free Tier eligible)

Project created

Compute Engine API enabled

🚀 1. Create a VM on GCP
Go to GCP Console → Compute Engine → VM instances → Create Instance

Suggested low-cost setup:

Name: vpn

Region: e.g., us-central1

Machine type: e2-micro (Free Tier)

Boot disk: Standard persistent disk, 10GB

Disable Logging, Monitoring, Backups

Leave the IP as Ephemeral (default)

Create the VM

🔐 2. SSH into the VM and Install WireGuard
Click SSH next to your new instance to open the terminal, then run:

```bash
curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
chmod +x wireguard-install.sh
sudo ./wireguard-install.sh
```

Follow the prompts:

- Keep default interface name wg0

- Use port 51820 or a random one (just remember it)

- DNS: choose 1.1.1.1, 1.0.0.1

- Pick a username for your first VPN client (e.g., laptop)

Once it finishes, you'll have a .conf file and QR code ready.

👤 To Add More VPN Clients Later
Run the script again:

```bash
sudo ./wireguard-install.sh
```
Choose "Add a new client", and it will generate a new config and QR code.

🌐 3. Open Firewall Port on GCP
Go to VPC network → Firewall → Create Firewall Rule:

Name: allow-wireguard

Network: default

Direction: Ingress

Source IP ranges: 0.0.0.0/0

Protocols and ports: check UDP, input the port you used (e.g., 51820)

📲 4. Import the VPN Config to Your Devices
📱 On Mobile (iOS / Android)
Install the WireGuard app

Tap + → Scan QR code

Import → Enable the tunnel

💻 On Desktop (macOS / Windows)
Install the WireGuard app

Click Import tunnel from file, load your .conf file

Enable it

🧪 5. Test the VPN
Visit whatismyip.com after connecting.
If you see your GCP public IP (e.g., 35.xx.xx.xx), it’s working ✅

📌 Optional but Recommended
Auto-start on boot:

```bash
sudo systemctl enable wg-quick@wg0
```

Start / Stop manually:

```bash
sudo systemctl start wg-quick@wg0
sudo systemctl stop wg-quick@wg0
```

🎯 Further Plan
Planning to migrate this setup to Oracle Cloud Free Tier for better speed (they offer always-free VMs).
You can also use DuckDNS to bind a domain name and auto-update IP for convenience.

Enjoy your personal cloud VPN – fast, private, and free to run! 🚀
