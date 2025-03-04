---
layout: default
title: "Setting Up Jellyfin Media Server on Fedora Atomic Desktops"
permalink: /docs/Jellyfin-on-Atomic-distro.html
---

# Setting Up Jellyfin Media Server on Fedora Atomic Desktops

<div class="intro-note">
    <p>
        Want your own personal Netflix without dealing with traditional Linux server setup?
        This guide walks you through setting up Jellyfin media server on Fedora Kinoite (or any Atomic desktop)
        using Podman containers—making it easier to maintain and less likely to break your system.
    </p>
</div>

## What You'll Need

- Fedora Kinoite, Silverblue, or another Atomic desktop (version 40+ recommended)
- Basic terminal familiarity (don't worry, we'll explain each command)
- Some media files you want to stream
- A bit of patience (but less than you'd need with a traditional distro!)

## Step 1: Setting Up Your Development Environment

<div class="frustration-meter">
    <div class="frustration-label">Frustration Level:</div>
    <div class="frustration-level">
        <div class="frustration-fill frustration-low"></div>
    </div>
</div>

<pre>
<code>
# Create a new toolbox container
toolbox create jellyfin

# Enter your new container
toolbox enter jellyfin

# Install essential tools
sudo dnf install -y podman-compose vim git curl wget
exit
</code>
</pre>

<div class="tip-box">
    <strong>What's happening:</strong> We're creating a disposable environment that won't affect your core system.
</div>

## Step 2: Creating Directory Structure

<div class="frustration-meter">
    <div class="frustration-label">Frustration Level:</div>
    <div class="frustration-level">
        <div class="frustration-fill frustration-low"></div>
    </div>
</div>

<pre>
<code>
# Create configuration directories
mkdir -p ~/jellyfin/{config,cache}
mkdir -p ~/media/{tvshows,movies,music}
</code>
</pre>

<div class="reality-check">
    <strong>Pro Tip:</strong> Keep these in your home directory to avoid permission headaches!
</div>

## Step 3: SELinux Configuration

<div class="frustration-meter">
    <div class="frustration-label">Frustration Level:</div>
    <div class="frustration-level">
        <div class="frustration-fill frustration-medium"></div>
    </div>
</div>

<pre>
<code>
# Apply SELinux contexts
sudo semanage fcontext -a -t container_file_t "/home/$USER/jellyfin(/.*)?"
sudo semanage fcontext -a -t container_file_t "/home/$USER/media(/.*)?"
sudo restorecon -Rvv ~/jellyfin ~/media
</code>
</pre>

## Step 4: Podman Compose Configuration

<div class="frustration-meter">
    <div class="frustration-label">Frustration Level:</div>
    <div class="frustration-level">
        <div class="frustration-fill frustration-medium"></div>
    </div>
</div>

<p>Create <code>~/jellyfin/podman-compose.yml</code>:</p>

<pre>
<code>
version: '3.8'
services:
  jellyfin:
    image: docker.io/jellyfin/jellyfin:latest
    container_name: jellyfin
    user: "$(id -u):$(id -g)"
    volumes:
      - ~/jellyfin/config:/config
      - ~/jellyfin/cache:/cache
      - ~/media:/media:ro
    ports:
      - 8096:8096
      - 8920:8920
    environment:
      - TZ=America/New_York
      - JELLYFIN_PublishedServerUrl=http://your-server-ip:8096
    restart: unless-stopped
</code>
</pre>

## Step 5: Deployment & Automation

<div class="frustration-meter">
    <div class="frustration-label">Frustration Level:</div>
    <div class="frustration-level">
        <div class="frustration-fill frustration-medium"></div>
    </div>
</div>

<pre>
<code>
# Start Jellyfin
podman-compose -f ~/jellyfin/podman-compose.yml up -d

# Configure firewall
sudo firewall-cmd --permanent --add-port=8096/tcp
sudo firewall-cmd --permanent --add-port=8920/tcp
sudo firewall-cmd --reload

# Create systemd service (run outside toolbox!)
podman generate systemd --new --name jellyfin > ~/.config/systemd/user/jellyfin.service
systemctl --user enable --now jellyfin.service
</code>
</pre>

<div class="reality-check">
    <strong>Common Mistake:</strong> Running systemd commands inside toolbox will fail!
</div>

## Step 6: Verification

<div class="frustration-meter">
    <div class="frustration-label">Frustration Level:</div>
    <div class="frustration-level">
        <div class="frustration-fill frustration-low"></div>
    </div>
</div>

<pre>
<code>
# Check container status
podman ps

# View logs
podman logs jellyfin

# Test access
curl http://localhost:8096
</code>
</pre>

## Step 7: Maintenance

<div class="frustration-meter">
    <div class="frustration-label">Frustration Level:</div>
    <div class="frustration-level">
        <div class="frustration-fill frustration-low"></div>
    </div>
</div>

<pre>
<code>
# Update container
podman-compose -f ~/jellyfin/podman-compose.yml pull
podman-compose -f ~/jellyfin/podman-compose.yml up -d

# Cleanup images
podman image prune -a
</code>
</pre>

## Troubleshooting Guide

<div class="tip-box">
    ### Permission Issues
    <pre>
    <code>
    podman unshare chown -R $(id -u):$(id -g) ~/jellyfin
    </code>
    </pre>
</div>

<div class="tip-box">
    ### SELinux Diagnostics
    <pre>
    <code>
    sudo ausearch -m avc -ts recent | audit2why
    </code>
    </pre>
</div>

<div class="reality-check">
    ### Nuclear Option
    <pre>
    <code>
    podman-compose -f ~/jellyfin/podman-compose.yml down
    rm -rf ~/jellyfin/config/*
    </code>
    </pre>
</div>

