---
title: "Building a Raspberry Pi 4 Wireless Home Surveillance System"
date: 2026-04-22
categories:
  - Blog
tags:
  - Raspberry Pi
  - Linux
  - Camera
---

I recently turned a Raspberry Pi 4B with an camera module into a wireless network camera that I can monitor from my phone. The goal was simple:

* Boot automatically
* Connect to Wi-Fi
* Start streaming on power-up
* Watch from a phone over the local network
* Keep latency as low as possible

This post documents the full process, the mistakes I made, what actually worked, and the final stable configuration.

<figure>
  <img src="/images/Pi1.jpg" alt="Raspberry Pi Photo" width="700">
  <figcaption>This is the assembled Raspberry Pi.</figcaption>
</figure>

---

# Hardware Used

* Raspberry Pi 4 Model B
* Raspberry Pi Camera Module (Always disconnect the power before connecting the camera.)
* microSD card
* Stable 5V / 3A power supply
* Wi-Fi network
* Phone with VLC installed

---

# Initial Goal

I wanted a lightweight wireless camera for:

* door viewer monitoring & room monitoring
* pet monitoring
* quick remote viewing over Wi-Fi

---

# Step 1: Install Raspberry Pi OS

I used **Raspberry Pi Imager** and selected:

* Raspberry Pi 4
* Raspberry Pi OS (64-bit)

During advanced setup, I preconfigured:

* Hostname: `Picam`
* Username: `Picam`
* Wi-Fi SSID + password
* Enabled SSH

This makes headless setup much easier.

---

# Step 2: First Boot and SSH Access

After flashing the SD card:

1. Insert SD card
2. Connect camera module
3. Power on the Pi
4. Wait 1–2 minutes
5. Connect from Windows:

```bash
ssh Picam@Picam.local
```

If prompted for host authenticity:

```text
yes
```

Then enter your password.

```bash
sudo apt update
sudo apt full-upgrade -y
sudo reboot
```
Upgrade the system. Then restart and reconnect via SSH.

---

# Step 3: Verify the Camera

Run:

```bash
rpicam-hello --list-cameras
```

My output showed:

```text
ov5647
```

That confirmed:

* camera cable installed correctly
* CSI interface working
* drivers loaded properly

---

# Step 4: First Working Network Stream

The first working command was:

```bash
rpicam-vid -t 0 -n --width 1280 --height 720 --framerate 30 \
--codec libav --libav-format mpegts \
-o tcp://0.0.0.0:8888?listen=1
```

Then on my phone (VLC):

```text
tcp://Picam.local:8888
```

---

# New Problem: Latency Was High

The stream was usable, but latency was noticeable.

Reasons:

* 720p resolution
* 30 fps
* TCP buffering
* VLC network cache
* MPEG transport stream overhead

---

# Final Low-Latency Configuration

I reduced latency significantly by switching to:

```bash
rpicam-vid -t 0 -n --width 640 --height 480 --framerate 20 \
--codec libav --libav-format mpegts \
--bitrate 1000000 \
-o tcp://0.0.0.0:8888?listen=1
```

### Why This Helped

* Lower resolution = less data
* Lower FPS = easier encoding
* Lower bitrate = faster buffering
* More stable over Wi-Fi

---

# VLC Mobile Settings (Important)

Inside VLC on mobile:

Set **Network caching** to:

```text
100 ms
```

or even:

```text
50 ms
```

This had a major impact on responsiveness.

---

# Step 5: Auto Start on Boot

I wanted the camera to start automatically after power-on.

Create a systemd service:

```bash
sudo nano /etc/systemd/system/Picam.service
```

Paste:

```ini
[Unit]
Description=Picam Low Latency Camera Stream
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=Picam
ExecStart=/usr/bin/rpicam-vid -t 0 -n --width 640 --height 480 --framerate 20 --codec libav --libav-format mpegts --bitrate 1000000 -o tcp://0.0.0.0:8888?listen=1
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

Enter:

```bash
Ctrl + O
Enter
Ctrl + X
```

Enable it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable Picam.service
sudo systemctl start Picam.service
```

Check:

```bash
systemctl status Picam.service
```

You want:

```text
active (running)
```

<figure>
  <img src="/images/Pi2.jpg" alt="The image from the camera." width="700">
  <figcaption>This worked. (Although the quality is a bit...) </figcaption>
</figure>

---

# Mistakes I Made (and Lessons Learned)

## 1. Mixing Command Syntax

I mixed raw H264 mode with MPEG-TS mode.

Use the right syntax for each streaming method.

## 2. Assuming `.local` Always Works

Windows mDNS resolution can be inconsistent.

When in doubt:

```bash
hostname -I
```

Then use the IP directly:

```text
tcp://192.168.x.x:8888
```

## 3. SSH Host Key Warnings After Reinstall

After reflashing, Windows reported:

```text
REMOTE HOST IDENTIFICATION HAS CHANGED
```

Fix:

```bash
ssh-keygen -R Picam.local
```

Then reconnect.

## 4. Do Not Unplug Power Directly.

The best method is:

```bash
sudo poweroff
```

Then wait a few dozen seconds before disconnecting the power.

---

# Reliability Tips

## Use a Fixed IP

Reserve the Raspberry Pi in your router DHCP table.

This avoids changing addresses after reboot.

## Use Good Wi-Fi Signal

High ping times directly affect stream responsiveness.

## Keep the System Updated

```bash
sudo apt update
sudo apt full-upgrade -y
```

## Last Resort

If the problem still cannot be resolved, you can try reinstalling the system.

---

# Final Result

My Raspberry Pi now functions as a lightweight wireless camera:

* boots automatically
* joins Wi-Fi automatically
* starts streaming automatically
* viewable from phone instantly
* low latency

---

# Possible Future Upgrades

* Motion detection recording
* Night vision camera module
* Remote access outside home
* AI object detection

---

# Final Thoughts

Raspberry Pi is a very fun platform that can be used for many interesting projects. 

However, it is relatively expensive and can sometimes be a bit unstable. 

For a beginner and hobbyist like me, it is more than sufficient. Especially now, with powerful AI tools available, the barrier to using it has been greatly reduced.

