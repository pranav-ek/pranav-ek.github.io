---
layout: post
title: Pihole on Raspberry Pi Zero 2W
date: '2025-12-26 17:10:00'
---

## 1. Overview

This guide provides a high-level summary of setting up Pi-hole on a Raspberry Pi Zero 2W using DietPi, with Tailscale for remote ad-blocking.

## 2. Prerequisites

- Raspberry Pi Zero 2W (or any other Raspberry Pi)
- SD card (at least 32GB)
- Micro USB power supply
- WiFi connectivity

## 3. Install DietPi on Raspberry Pi Zero 2W

DietPi is a lightweight OS perfect for the Pi Zero.

- **Download & Flash**: Download the DietPi image for Raspberry Pi and flash it to your SD card using [balenaEtcher](https://etcher.balena.io/).
- **WiFi Setup (Crucial)**: Before ejecting the SD card, open `dietpi.txt` to enable `AUTO_SETUP_NET_WIFI_ENABLED=1`, and enter your credentials in `dietpi-wifi.txt`. This allows a headless boot.
- **Link**: [DietPi Installation Docs](https://dietpi.com/docs/install/)

## 4. Install Pi-hole

Once DietPi is running and you have SSH access:

- **Install Command**: Run `curl -sSL https://install.pi-hole.net | bash`.
- **Configuration**: During installation, assign a **static IP** to your Pi to ensure consistent DNS routing.

### DNS Configuration

It is critical to set a **static IP address** for the Raspberry Pi Zero 2W so its address doesn't change. You must also configure your network to use Pi-hole for DNS.

If your router (e.g., provided by AT&T) prevents you from changing DNS settings, you must use one of the following methods. **Methods 1, 2, and 3 are only required if you cannot modify the DNS settings directly on your router.**

#### Method 1: Configure DNS on Each Device (Easiest)
This is best for a small number of devices.

Manually update the network settings on each device to point to the Pi-hole's static IP address. This typically involves navigating to the Wi-Fi or Ethernet settings, locating the DNS server configuration (often under "Advanced" or "Manual" settings), and replacing the default addresses with your Pi-hole's IP.

#### Method 2: Use Your Own Router (Recommended for Full Control)
This involves putting the ISP gateway in bridge mode (IP Passthrough) and letting your own router handle everything.

1. **Set up IP Passthrough**: Configure your ISP gateway to pass the public IP to your personal router.
2. **Connect your Router**: Plug your router's WAN port into a LAN port on the gateway.
3. **Set DNS on Your Router**: Log into your router's admin page (e.g., `192.168.1.1`) and set your custom DNS servers in the WAN/Internet settings.

#### Method 3: Disable ISP DHCP (Advanced)
If you don't want a second router but want automatic settings for all devices, you can run your own DHCP server via Pi-hole.

1. **Disable DHCP on your Router**: Log into your router's web interface, locate the DHCP server settings, and disable it. This prevents the router from assigning IP addresses, allowing the Pi-hole to take over as the primary DHCP server on your network.

2. **Enable DHCP in the Admin Console:**
    1. Open your browser and go to your Pi-hole Admin Dashboard (usually `http://pi.hole/admin` or `http://<IP_ADDRESS>/admin`).
    2. Navigate to **Settings** > **DHCP**.
    3. Check the box for **DHCP server enabled**.

3. **Configure DHCP Settings:**
    - **Range of IP addresses**: Define a range that excludes your gateway IP. (e.g., `192.168.1.10` to `192.168.1.250`).
    - **Router (gateway) IP address**: Enter your router's IP (e.g., `192.168.1.254`).

- **Link**: [Pi-hole Basic Install](https://docs.pi-hole.net/main/basic-install/)

## 5. Remote Access with Tailscale

To block ads on your mobile devices while away from home:

- **Install Tailscale**: Run `curl -fsSL https://tailscale.com/install.sh | sh`.
- **Configure**: Run `sudo tailscale up --accept-dns=false`. The `--accept-dns=false` flag is important so the Pi doesn't use itself for DNS during startup loops.
- **Admin Console**: In the Tailscale admin console:
    1. Add your Pi's Tailscale IP as a **Global Nameserver**.
    2. Enable **"Override local DNS"**.
- **Link**: [Tailscale & Pi-hole Guide](https://tailscale.com/kb/1114/pi-hole)

## 6. Conclusion

Setting up Pi-hole on a Raspberry Pi Zero 2W is a cost-effective way to improve your network's privacy and speed by blocking ads and trackers at the DNS level. By leveraging DietPi's lightweight footprint, you maximize the performance of the Pi Zero's hardware. Integrating Tailscale further extends these benefits, providing a secure, encrypted tunnel for your mobile devices to access your home DNS from anywhere in the world. This combination creates a robust, portable, and low-maintenance network security solution that enhances your browsing experience across all your devices.
