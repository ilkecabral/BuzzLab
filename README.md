# -BuzzLab<div align="center">

[![Typing SVG](https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=700&size=32&pause=1000&color=E94560&center=true&vCenter=true&width=600&lines=BuzzLab+%F0%9F%A7%AA;Self-Hosted+Home+Lab;Built+on+Proxmox+VE)](https://git.io/typing-svg)

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:1a1a2e,50:16213e,100:e94560&height=120&section=header&text=&animation=fadeIn" width="100%"/>

![Proxmox](https://img.shields.io/badge/Proxmox-E57000?style=for-the-badge&logo=proxmox&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu_24.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Jellyfin](https://img.shields.io/badge/Jellyfin-00A4DC?style=for-the-badge&logo=jellyfin&logoColor=white)
![GitHub Stats](https://img.shields.io/badge/Home_Lab-Active-e94560?style=for-the-badge)

</div>

---

## `> whoami`

Self-hosted home lab running on a single bare-metal machine.  
Built around media automation, containerised services, and (soon) security monitoring.  
Everything runs on **Proxmox VE** with a Docker stack inside an LXC container.

---

## `> lshw` — Hardware

| Component | Spec |
|-----------|------|
| 🖥️ CPU | Intel Core i7 |
| 🧠 RAM | 16 GB |
| ⚡ Boot | NVMe SSD — 238.5 GB *(Proxmox OS)* |
| 💾 Data | SATA HDD — 465.8 GB *(media & appdata)* |
| 🌐 Network | TP-Link TL-WR841N · `192.168.0.0/24` |

---

## `> docker ps` — Services

| Service | Port | Role |
|---------|------|------|
| 🎬 **Jellyfin** | `:8096` | Media server — Movies & TV |
| 📺 **Sonarr** | `:8989` | TV show automation |
| 🎥 **Radarr** | `:7878` | Movie automation |
| 🔍 **Prowlarr** | `:9696` | Indexer manager |
| ⬇️ **qBittorrent** | `:8080` | Download client |
| 💬 **Bazarr** | `:6767` | Subtitle management |

All services run via **Docker Compose** inside an Ubuntu 24.04 LXC (`mediaserver`, ID: 100) on Proxmox.  
Compose file lives at `/mnt/media/appdata/docker-compose.yml`.

---

## `> tree /mnt/media` — Storage Layout

```
/mnt/media/                     ← SATA HDD (465.8 GB)
├── appdata/
│   ├── jellyfin/               ← Jellyfin config & metadata
│   ├── sonarr/                 ← Sonarr config & DB
│   ├── radarr/                 ← Radarr config & DB
│   ├── prowlarr/               ← Indexer config
│   ├── qbittorrent/            ← Download client config
│   └── bazarr/                 ← Subtitle config
├── media/
│   ├── movies/                 ← Movie library
│   └── tv/                     ← TV show library
└── downloads/                  ← Download output
```

---

## `> upcoming` — Planned

```diff
+ [ PLANNED ] Wazuh LXC — SIEM & Intrusion Detection
+             RAM: 6144 MB
+             Purpose: Security monitoring across the entire stack
```

---

## `> stats`

<div align="center">

![GitHub Stats](https://github-readme-stats.vercel.app/api?username=ilkecabral&show_icons=true&theme=radical&bg_color=1a1a2e&title_color=e94560&icon_color=e94560&text_color=d0d0e0&border_color=16213e&hide_border=false)

![Top Languages](https://github-readme-stats.vercel.app/api/top-langs/?username=YOUR_USERNAME&layout=compact&theme=radical&bg_color=1a1a2e&title_color=e94560&text_color=d0d0e0&border_color=16213e)

</div>

---

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:e94560,50:16213e,100:1a1a2e&height=100&section=footer" width="100%"/>

*BuzzLab · March 2026*

</div>
