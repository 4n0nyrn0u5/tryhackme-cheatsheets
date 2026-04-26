# 🔍 Recon & OSINT — Cheatsheet

> **Manba:** TryHackMe — Cyber Security 101  
> **Mavzular:** Passive Recon · Active Recon · Nmap · Directory Enum · Google Dorks · OSINT toollar

---

## 1. Recon metodologiyasi

```
RECON (Reconnaissance)
├── Passive Recon (Sust) — Targetga tegmasdan
│   ├── Google Dorks
│   ├── WHOIS lookup
│   ├── DNS lookup
│   ├── Shodan, Censys
│   ├── Archive.org (Wayback Machine)
│   └── LinkedIn, GitHub, Pastebin
│
└── Active Recon (Faol) — Target bilan to'g'ridan-to'g'ri
    ├── Port scanning (nmap)
    ├── Service enumeration
    ├── Directory fuzzing (gobuster)
    ├── Web scraping
    └── Banner grabbing
```

> ⚠️ **Eslatma:** Active recon izlar qoldiradi va ruxsatsiz qilinsa noqonuniy!

---

## 2. Passive Recon Toollar

### WHOIS — Domain ma'lumoti
```bash
whois example.com
# Natija: Egasi, registrar, sana, nameserver, manzil
```

### DNS Lookup
```bash
# A record (IP):
dig example.com A
nslookup example.com

# Mail server:
dig example.com MX

# Name server:
dig example.com NS

# Barcha recordlar:
dig example.com ANY

# Reverse lookup (IP → domain):
dig -x 93.184.216.34

# Zone transfer (agar ruxsat berilsa — katta info):
dig axfr @ns1.example.com example.com

# Subdomain topish (brute force):
dnsenum example.com
fierce --domain example.com
```

### Subdomainlarni topish
```bash
# subfinder (passiv):
subfinder -d example.com

# amass (kuchli):
amass enum -passive -d example.com
amass enum -active -d example.com

# assetfinder:
assetfinder example.com

# crt.sh (SSL sertifikatlardan):
curl "https://crt.sh/?q=%.example.com&output=json" | \
  python3 -m json.tool | grep name_value
# Yoki brauzerda: https://crt.sh/?q=%.example.com
```

### Shodan — Internet'dagi qurilmalar ⭐
```bash
# Saytda: shodan.io
# Qidirish misollari:
hostname:example.com          # Hostname bo'yicha
org:"Company Name"            # Tashkilot bo'yicha
port:22 country:UZ            # SSH O'zbekistonda
product:Apache version:2.4    # Muayyan Apache versiya
ssl:"example.com"             # SSL sertifikatida
vuln:CVE-2021-44228           # Log4j zaifligiga ega

# CLI (API key kerak):
shodan search hostname:example.com
shodan host 93.184.216.34
```

### Google Dorks ⭐
```bash
# Asosiy operatorlar:
site:example.com              # Faqat shu sayt
filetype:pdf                  # Fayl turi
inurl:admin                   # URL'da "admin"
intitle:"index of"            # Directory listing
intext:"password"             # Matnda "password"

# Amaliy dorklar:
site:example.com filetype:pdf           # PDF fayllar
site:example.com inurl:admin            # Admin panel
site:example.com inurl:login            # Login sahifasi
site:example.com filetype:sql           # SQL dump!
site:example.com filetype:env           # .env fayl!
site:example.com filetype:log           # Log fayllar
intitle:"index of" "config.php"        # Config fayllar
inurl:".git" site:example.com          # Git repo leak
"example.com" site:pastebin.com        # Pastebin leak
"@example.com" filetype:xls            # Email listlar

# GitHub dorks:
# github.com'da qidirish:
"example.com" password
"example.com" api_key
"example.com" secret_key
org:company-name password
```

### Wayback Machine — Eski versiya
```bash
# Brauzerda:
https://web.archive.org/web/*/example.com

# CLI:
waybackurls example.com > old_urls.txt
cat old_urls.txt | grep -E "\.(php|asp|aspx|jsp)" | head -50
```

### theHarvester — Email va subdomain yig'ish
```bash
theHarvester -d example.com -b google
theHarvester -d example.com -b linkedin
theHarvester -d example.com -b all
# Natija: emaillar, subdomainlar, IP'lar
```

---

## 3. Nmap — Port Scanner ⭐

### Asosiy scan turlari
```bash
# Tez scan (top 1000 port):
nmap 10.10.10.10

# Barcha portlar (sekin):
nmap -p- 10.10.10.10

# Muayyan portlar:
nmap -p 22,80,443,8080 10.10.10.10

# Servis versiya aniqlash:
nmap -sV 10.10.10.10

# OS aniqlash (root kerak):
nmap -O 10.10.10.10

# Aggressive scan (OS + version + script + traceroute):
nmap -A 10.10.10.10

# UDP scan (sekin!):
nmap -sU 10.10.10.10
```

### Scan texnikaları
```bash
# SYN scan (default, root):
nmap -sS 10.10.10.10

# Connect scan (root shart emas):
nmap -sT 10.10.10.10

# Ping scan (host discovery):
nmap -sn 10.10.10.0/24

# Firewall bypass:
nmap -sF 10.10.10.10    # FIN scan
nmap -sX 10.10.10.10    # Xmas scan
nmap -sN 10.10.10.10    # Null scan
nmap -f 10.10.10.10     # Fragment packets
```

### Nmap Scripts (NSE) ⭐
```bash
# Script kategoriyalari:
# auth, broadcast, brute, default, discovery, exploit, fuzzer, intrusive, safe, vuln

# Default scriptlar:
nmap -sC 10.10.10.10

# Muayyan script:
nmap --script=http-title 10.10.10.10
nmap --script=ssh-brute 10.10.10.10

# Zaiflik scan:
nmap --script=vuln 10.10.10.10

# Web server info:
nmap --script=http-headers,http-methods,http-robots.txt 10.10.10.10

# SMB:
nmap --script=smb-vuln* 10.10.10.10
nmap --script=smb-enum-shares 10.10.10.10

# FTP:
nmap --script=ftp-anon 10.10.10.10    # Anonymous login

# DNS zone transfer:
nmap --script=dns-zone-transfer --script-args dns-zone-transfer.domain=example.com 10.10.10.10
```

### Nmap Output
```bash
# Normal output:
nmap -oN scan.txt 10.10.10.10

# XML output (Metasploit uchun):
nmap -oX scan.xml 10.10.10.10

# Greppable:
nmap -oG scan.gnmap 10.10.10.10

# Hammasi:
nmap -oA scan 10.10.10.10
# → scan.nmap, scan.xml, scan.gnmap

# Tez professional scan:
nmap -sV -sC -p- --min-rate 5000 -oA full_scan 10.10.10.10
```

### Nmap Timing
```bash
-T0   # Paranoid (juda sekin, IDS bypass)
-T1   # Sneaky
-T2   # Polite
-T3   # Normal (default)
-T4   # Aggressive (CTF uchun) ⭐
-T5   # Insane (juda tez, noto'g'ri natija berishi mumkin)
```

---

## 4. Directory va File Enumeration

### Gobuster ⭐
```bash
# Directory brute force:
gobuster dir \
  -u http://10.10.10.10 \
  -w /usr/share/wordlists/dirb/common.txt

# Ko'proq wordlist (kuchliroq):
gobuster dir \
  -u http://10.10.10.10 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,html,txt,js,json \
  -t 50

# Subdomain brute force:
gobuster dns \
  -d example.com \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

# Foydali flaglar:
# -x → fayl kengaytmalari
# -t → threadlar soni (tezlik)
# -b → skip status kodlar (e.g., -b 404,403)
# -k → SSL tekshiruvsiz
# -H → custom header
# -c → cookie
```

### Feroxbuster (gobuster alternativ, rekursiv)
```bash
feroxbuster -u http://10.10.10.10 \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,html,txt \
  --depth 3
```

### ffuf (Fuzz Faster U Fool) ⭐
```bash
# Directory:
ffuf -u http://10.10.10.10/FUZZ \
  -w /usr/share/wordlists/dirb/common.txt

# Subdomain:
ffuf -u http://FUZZ.example.com \
  -w subdomains.txt \
  -H "Host: FUZZ.example.com"

# Parameter fuzzing:
ffuf -u http://10.10.10.10/page.php?param=FUZZ \
  -w payloads.txt

# POST body:
ffuf -u http://10.10.10.10/login \
  -X POST \
  -d "username=FUZZ&password=password" \
  -w usernames.txt \
  -fc 401  # 401 javoblarni filter
```

### Foydali Wordlistlar
```bash
/usr/share/wordlists/
├── dirb/
│   ├── common.txt          ← Kichik, tez
│   └── big.txt             ← Katta
├── dirbuster/
│   └── directory-list-2.3-medium.txt  ← Eng yaxshi ⭐
└── rockyou.txt             ← Parollar uchun

# SecLists (alohida o'rnatish):
apt install seclists
ls /usr/share/seclists/
```

---

## 5. Service Enumeration

### FTP (port 21)
```bash
# Anonymous login tekshirish:
ftp 10.10.10.10
# Username: anonymous
# Password: (bo'sh yoki email)

# nmap bilan:
nmap --script ftp-anon -p 21 10.10.10.10

# Fayllarni olish:
ftp> ls
ftp> get filename
ftp> mget *
```

### SSH (port 22)
```bash
# Ulanish:
ssh user@10.10.10.10
ssh -i id_rsa user@10.10.10.10    # Private key bilan
ssh -p 2222 user@10.10.10.10      # Boshqa port

# Brute force:
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  ssh://10.10.10.10

# Banner grabbing:
nc 10.10.10.10 22
```

### SMB (port 139/445)
```bash
# Share'larni ro'yxatlash:
smbclient -L //10.10.10.10 -N    # -N = no password

# Share'ga kirish:
smbclient //10.10.10.10/sharename -N

# enum4linux (Windows enumeration):
enum4linux -a 10.10.10.10

# nmap scripts:
nmap --script smb-enum-shares,smb-vuln* -p 445 10.10.10.10
```

### HTTP (port 80/443)
```bash
# Banner grabbing:
curl -I http://10.10.10.10
nc 10.10.10.10 80
# → GET / HTTP/1.0
# → [Enter x2]

# Nikto (web vuln scanner):
nikto -h http://10.10.10.10

# whatweb (texnologiya aniqlash):
whatweb http://10.10.10.10
```

---

## 6. OSINT — Real Hayot

### Email topish
```bash
# hunter.io — email pattern aniqlash
# phonebook.cz — email qidirish

# theHarvester:
theHarvester -d example.com -b all

# LinkedIn: company employees → email pattern
# Format: firstname.lastname@company.com
```

### Parol leaklari
```bash
# haveibeenpwned.com — email leaked?
# dehashed.com — credential search
# leakcheck.io

# GitHub'da leak qidirish:
# github.com/search?q="company.com"+password&type=code
```

### Metadata
```bash
# Rasm metadata (EXIF):
exiftool image.jpg
# → GPS koordinatalar, kamera, muallif, sana

# PDF metadata:
exiftool document.pdf
# → Muallif, dastur, sana

# Metadata tozalash:
mat2 document.pdf
```

---

## ⚡ Recon Workflow (CTF/Pentest)

```bash
# 1. Asosiy scan:
nmap -sV -sC -p- --min-rate 5000 -oA initial_scan TARGET_IP

# 2. Web server bo'lsa:
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -x php,html,txt -o dirs.txt
nikto -h http://TARGET
whatweb http://TARGET

# 3. Domain bo'lsa:
whois domain.com
dig domain.com ANY
subfinder -d domain.com
theHarvester -d domain.com -b all

# 4. Topilgan portlarga qarab:
# Port 21 → ftp TARGET
# Port 22 → ssh, banner grab
# Port 139/445 → smbclient, enum4linux
# Port 3306 → mysql -h TARGET -u root
# Port 27017 → mongo TARGET (MongoDB)
```

---

*TryHackMe Cyber Security 101 | Recon & OSINT*
