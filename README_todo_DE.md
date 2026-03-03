# AI Worker Node – Ubuntu + OpenClaw Intranet Deployment

## Overview

Dieses Projekt beschreibt den Aufbau eines dedizierten KI-Worker-Nodes auf älterer Hardware innerhalb eines Intranets.

Ziel ist es, einen stabilen, isolierten und remote steuerbaren AI-Agent-Server bereitzustellen, der über externe LLM-APIs (z. B. OpenRouter) angebunden wird und mehrere virtuelle „Mitarbeiter“ simulieren kann.

Technologische Basis:

- Ubuntu 24.04 LTS Desktop
- OpenClaw
- OpenSSH
- xRDP
- OpenRouter (oder alternativer LLM Provider)
- Optional Docker Deployment

---

# 1. System Architecture

```
Windows Workstation
 ├─ RDP → Ubuntu GUI (xRDP)
 ├─ SSH → OpenClaw CLI
 └─ Browser → WebUI

Ubuntu AI Node
 ├─ openclaw user (isolated)
 ├─ OpenClaw CLI
 ├─ WebUI
 ├─ LLM API Integration
 └─ Messenger APIs
```

---

# 2. Base Installation – Ubuntu 24.04 LTS

## 2.1 Download

ISO von:
[https://ubuntu.com/download](https://ubuntu.com/download)

## 2.2 USB erstellen (Windows)

Tool:
Rufus

- GPT + UEFI
- ISO Mode

## 2.3 Installation

- Normale Installation
- Full disk
- Admin-User erstellen (z. B. sysadmin)
- Nach Installation:

```bash
sudo apt update
sudo apt upgrade -y
sudo reboot
```

---

# 3. Netzwerk Setup

## 3.1 Feste IP konfigurieren

Empfohlen:

- Router DHCP-Reservierung

Beispiel:

```
192.168.0.50
```

---

# 4. Remote Access Setup

## 4.1 SSH

```bash
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo ufw allow 22
```

Client Optionen:

- PuTTY
- Windows PowerShell
- Visual Studio Code (Remote SSH Extension)

---

## 4.2 RDP (GUI Zugriff)

```bash
sudo apt install xrdp -y
sudo systemctl enable xrdp
sudo ufw allow 3389
```

Windows Client:
Microsoft Remote Desktop

---

# 5. Benutzer- und Rechtekonzept

## 5.1 OpenClaw Service-User

```bash
sudo adduser openclaw
```

Wichtig:

- Kein sudo
- Keine Admin-Gruppen
- Home-Directory isoliert
- Optional: AppArmor Profil

---

# 6. OpenClaw Installation (Bare Metal)

Als openclaw User:

```bash
su - openclaw
git clone <repo-url>
cd openclaw
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

.env Datei:

```
LLM_PROVIDER=openrouter
LLM_API_KEY=YOUR_KEY
MODEL=gpt-4.1
```

---

# 7. WebUI & CLI Start

CLI:

```bash
python main.py
```

WebUI:

```bash
python webui.py
```

---

# 8. Security Hardening Guide (Production-Grade)

## 8.1 SSH Hardening

Datei:

```bash
sudo nano /etc/ssh/sshd_config
```

Empfohlene Änderungen:

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers sysadmin openclaw
```

SSH neu starten:

```bash
sudo systemctl restart ssh
```

---

## 8.2 Firewall Policy

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22
sudo ufw allow 3389
sudo ufw enable
```

Optional: IP Range Restriction

---

## 8.3 Fail2Ban

```bash
sudo apt install fail2ban -y
```

Aktiviert Schutz gegen Brute Force Angriffe.

---

## 8.4 System Updates automatisieren

```bash
sudo apt install unattended-upgrades
```

---

## 8.5 Prozess-Isolierung

Empfohlen:

- systemd Service Unit
- NoNewPrivileges
- PrivateTmp
- MemoryLimit

---

## 8.6 AppArmor aktivieren

Ubuntu bringt AppArmor standardmäßig mit.

Status prüfen:

```bash
sudo aa-status
```

---

# 9. Docker-basierte Alternative (Empfohlen für Isolation)

## 9.1 Docker installieren

```bash
sudo apt install docker.io -y
sudo systemctl enable docker
```

User zur Docker-Gruppe hinzufügen:

```bash
sudo usermod -aG docker openclaw
```

---

## 9.2 Beispiel Dockerfile

```Dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY . .
RUN pip install --no-cache-dir -r requirements.txt

ENV LLM_PROVIDER=openrouter
ENV MODEL=gpt-4.1

CMD ["python", "main.py"]
```

---

## 9.3 Build & Run

```bash
docker build -t openclaw-node .
docker run -d \
  -p 8080:8080 \
  --env-file .env \
  --name openclaw \
  --restart unless-stopped \
  openclaw-node
```

---

## 9.4 Vorteile Docker

- Prozess-Isolation
- Port-Management
- Schnellere Updates
- Reproduzierbare Deployments
- Snapshot-Strategie möglich

---

# 10. LLM API Integration

Empfohlene Provider:

- OpenRouter
- OpenAI
- Anthropic

OpenRouter erlaubt Zugriff auf mehrere Modelle über eine API.

---

# 11. Messenger Integration (Optional)

- Telegram Bot API
- WhatsApp Business API

API Tokens werden ausschließlich in .env gespeichert.

---

# 12. Operational Best Practices

- Keine Root-Ausführung
- Keine globalen API-Keys
- Logs regelmäßig prüfen
- Separate Agent-Profile
- Backup-Strategie definieren
- Ressourcen-Monitoring (htop, glances)

---

# 13. Performance auf älterer Hardware

Empfehlungen:

- Leichtgewichtige Desktop-Umgebung (z. B. Xfce)
- Swappiness anpassen
- Kein unnötiger Dienststart
- Docker Resource Limits setzen

---

# 14. Ergebnis

Ein älterer PC wird transformiert zu:

- Intranet AI Worker Node
- Multi-Agent System
- API-gekoppelter KI-Assistent
- Remote administrierbarer Server
- Sichere isolierte Umgebung

---
