# 🦈 Network Sniffing — Wireshark

> Projet réalisé dans le cadre de la formation **La Plateforme** — Mai 2026  
> Analyse de trafic réseau avec Wireshark et tshark

---

## 📋 Présentation

Ce projet est une initiation concrète au **modèle OSI** et aux principaux protocoles réseau.  
L'objectif est de capturer, filtrer et analyser des trames réseau à l'aide de **Wireshark** (interface graphique) et **tshark** (ligne de commande), sur un réseau local virtuel composé de deux VMs sous VMware :

| Machine | OS | IP | Rôle |
|---|---|---|---|
| VM Serveur | Kali Linux | 172.16.228.134 | Héberge les services |
| VM Client | Debian | 172.16.228.133 | Génère le trafic |

---

## 📁 Structure du dépôt

```
wireshark/
│
├── docs/                         # Fichiers de capture réseau
│   ├── capture2.pcapng
│   ├── capture_dhcp.pcapng
│   ├── capture_dns.pcapng
│   ├── capture_ftp.pcapng
│   ├── capture_http.pcapng
│   ├── capture_https.pcapng
│   ├── capture_mdns.pcapng
│   └── capture_smb2.pcapng
│
├── images/                       # Captures d'écran Wireshark & tshark
│   ├── capture_dhcp.png
│   ├── capture_dns_serveur_client.png
│   ├── capture_ftp_kali.png
│   ├── capture_http.png
│   ├── capture_https_kali.png
│   ├── capture_mdns.png
│   ├── capture_smb.png
│   ├── capture_smb2.png
│   ├── capture_tcp
│   ├── capture_udp.png
│   ├── ftp_machine_client_debian.png
│   ├── paquet_arp.png
│   ├── tcp_flag_syn&ack.png
│   ├── tshark_capture_paquets.png
│   ├── tshark_dns.png
│   ├── tshark_http.png
│   ├── tshark_install.png
│   ├── tshark_port53.png
│   └── tshark_scan.png
│
├── wireshark.pdf                 # Présentation de soutenance
└── README.md
```

---

## 🔍 Protocoles analysés

### Partie 1 — Couches réseau

| Protocole | Couche OSI | Port | Description |
|---|---|---|---|
| **ARP** | L2/L3 | — | Résolution IP → MAC, type 0x0806 |
| **UDP** | L4 | 53 / 67 / 5353 | Sans connexion, rapide, temps réel |
| **TCP** | L4 | variable | Connexion fiable, 3-Way Handshake |

### Partie 2 — Protocoles applicatifs

| Protocole | Port | Chiffrement | Observations |
|---|---|---|---|
| **DHCP** | UDP 67/68 | ❌ | Séquence DORA — attribution IP automatique |
| **DNS** | UDP 53 | ❌ | Requêtes en clair — tous les sites consultés visibles |
| **mDNS** | UDP 5353 | ❌ | DNS multicast local, sans serveur central (Bonjour/Avahi) |
| **FTP** | TCP 21 | ❌ | Login et mot de passe **en clair** — dangereux |
| **HTTP** | TCP 80 | ❌ | Contenu HTML, headers, cookies lisibles |
| **HTTPS** | TCP 443 | ✅ TLS 1.3 | Données chiffrées, impossible à lire |
| **SMB2** | TCP 445 | ⚠️ Partiel | Partage de fichiers réseau (Samba) |

---

## 🖥️ Partie 3 — tshark

tshark est la version CLI de Wireshark, utilisable dans des scripts bash.

### Installation
```bash
sudo apt install tshark
```

### Commandes utilisées

```bash
# Lister les interfaces disponibles
tshark -D

# Capturer 20 paquets sur wlp2s0
tshark -i wlp2s0 -c 20

# Filtre de CAPTURE BPF (avant) — seuls les paquets DNS sont enregistrés
tshark -i wlp2s0 -f "port 53"

# Filtre d'AFFICHAGE (après) — tous les paquets capturés, seul le DNS s'affiche
tshark -i wlp2s0 -Y "dns"

# Filtrer les requêtes HTTP
tshark -i wlp2s0 -Y "http"

# Commande combinée — capture DNS filtrée et sauvegardée
tshark -i wlp2s0 -f "port 53" -Y "dns" -c 50 -w capture_dns.pcapng
```

### Options principales

| Option | Description |
|---|---|
| `-i eth0` | Interface réseau à écouter |
| `-f "port 53"` | Filtre de capture BPF — appliqué **avant** |
| `-Y "dns"` | Filtre d'affichage — appliqué **après** |
| `-c 20` | Limiter à N paquets |
| `-w fichier.pcapng` | Sauvegarder la capture |
| `-D` | Lister les interfaces |
| `-r fichier.pcapng` | Lire un fichier existant |

---

## 🔒 Sécurité — FTP vs TLS

| Critère | FTP (sans TLS) | HTTPS / TLS |
|---|---|---|
| Chiffrement | ❌ Aucun | ✅ TLS 1.3 |
| Login visible | ✅ En clair | ❌ Chiffré |
| Mot de passe | ✅ En clair | ❌ Chiffré |
| Risque | 🔴 Très élevé | 🟢 Faible |
| Alternative | SFTP / FTPS | Déjà sécurisé |

> En écoutant des échanges FTP sans TLS dans Wireshark, le login et le mot de passe
> transitent en texte brut et sont interceptables par n'importe qui sur le réseau.

---

## 🛠️ Compétences techniques acquises

| Domaine | Compétences |
|---|---|
| **Modèle OSI / TCP-IP** | Identifier les couches L2 à L7 dans une trame, lire les en-têtes Ethernet, ARP, IP, TCP et UDP |
| **Analyse de trafic** | Capturer, filtrer et inspecter des trames avec Wireshark (filtres de capture BPF et filtres d'affichage) |
| **tshark (CLI)** | Capturer, filtrer, sauvegarder (`.pcapng`) et relire du trafic en ligne de commande, utilisable dans des scripts bash |
| **Protocoles applicatifs** | Décoder DHCP (séquence DORA), DNS, mDNS, FTP, HTTP, HTTPS/TLS 1.3 et SMB2 |
| **TCP** | Observer le 3-Way Handshake et les flags SYN / ACK |
| **Sécurité réseau** | Repérer les données transmises en clair (login/mot de passe FTP, requêtes DNS, HTTP) et justifier le passage à SFTP/FTPS et HTTPS |
| **Virtualisation & Linux** | Monter un réseau local virtuel sous VMware avec deux VMs (Kali Linux, Debian) |
| **Documentation** | Documenter une démarche technique et la présenter en soutenance |

---

## 🎯 Compétences visées

- Administrer et sécuriser les infrastructures réseaux
- Participer à la mesure et à l'analyse du niveau de sécurité
- Participer à la détection et au traitement des incidents de sécurité
- Participer à l'élaboration et à la mise en œuvre de la politique de sécurité

---

## 🔗 Ressources

- [Wireshark](https://www.wireshark.org/)
- [tshark filtering](https://www.wireshark.org/docs/man-pages/tshark.html)
- [Hex Packet Decoder](https://hpd.gasmi.net/)
- [Base64 En/Decoder](https://www.base64decode.org/)

---

*La Plateforme — Mai 2026*
