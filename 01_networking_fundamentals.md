# 🌐 Networking Fundamentals — Cheatsheet

> **Manba:** TryHackMe — Pre Security  
> **Mavzular:** OSI Model · TCP/IP · DNS · HTTP · Ports · Subnetting

---

## 1. OSI Model (7 qavat)

OSI — tarmoq aloqasini 7 ta mantiqiy qavatga bo'ladi. **"Please Do Not Throw Sausage Pizza Away"** — yodlash uchun.

| # | Qavat | Inglizcha | Protokollar / Qurilmalar | Vazifa |
|---|-------|-----------|--------------------------|--------|
| 7 | **Ilova** | Application | HTTP, HTTPS, FTP, DNS, SMTP | Foydalanuvchi bilan ishlash |
| 6 | **Taqdimot** | Presentation | SSL/TLS, JPEG, ASCII | Ma'lumotni format, encrypt |
| 5 | **Sessiya** | Session | NetBIOS, RPC | Ulanishni boshqarish |
| 4 | **Transport** | Transport | **TCP, UDP** | Segmentlash, port |
| 3 | **Tarmoq** | Network | **IP**, ICMP, ARP | IP manzil, routing |
| 2 | **Kanal** | Data Link | Ethernet, MAC, Switch | MAC manzil, frame |
| 1 | **Fizik** | Physical | Kabel, Hub, Wi-Fi | Bit uzatish |

### 🔴 Red Team uchun nima muhim?
- **L7** — Web hujumlar (XSS, SQLi, SSRF) shu qavat
- **L4** — Port scanning, TCP/UDP exploit
- **L3** — IP spoofing, network recon
- **L2** — ARP spoofing, MITM

---

## 2. TCP vs UDP

### TCP (Transmission Control Protocol) — Ishonchli
```
Client          Server
  |──── SYN ────→|     1. Aloqa boshlash so'rovi
  |←── SYN-ACK ──|     2. Tasdiqlash
  |──── ACK ────→|     3. Aloqa ochildi
  |              |
  |──── DATA ───→|     Ma'lumot yuborish
  |←─── ACK ────|     Tasdiqlash
  |              |
  |──── FIN ────→|     Aloqani yopish
  |←── FIN-ACK ──|
```

**Xususiyatlari:** Ordered, reliable, connection-oriented  
**Ishlatiladi:** HTTP/S, FTP, SSH, SMTP

### UDP (User Datagram Protocol) — Tez
```
Client          Server
  |──── DATA ───→|     Tasdiqlashsiz yuborish
  |──── DATA ───→|     Ba'zisi yo'qolishi mumkin
```

**Xususiyatlari:** No guarantee, connectionless, fast  
**Ishlatiladi:** DNS, DHCP, Video streaming, VoIP, Gaming

---

## 3. Muhim Portlar 🔑

> **Yodlash:** Port < 1024 = well-known, 1024-49151 = registered, > 49151 = dynamic

| Port | Protokol | Xizmat | Hujumchi uchun ahamiyat |
|------|----------|--------|------------------------|
| 21 | TCP | FTP | Anonymous login, brute force |
| 22 | TCP | SSH | Brute force, key theft |
| 23 | TCP | Telnet | Cleartext, MITM |
| 25 | TCP | SMTP | Email relay, spam |
| 53 | TCP/UDP | DNS | Zone transfer, cache poison |
| 80 | TCP | HTTP | Web attacks |
| 110 | TCP | POP3 | Email interception |
| 139/445 | TCP | SMB | EternalBlue, ransomware |
| 443 | TCP | HTTPS | Web attacks (encrypted) |
| 3306 | TCP | MySQL | SQLi, credential theft |
| 3389 | TCP | RDP | BlueKeep, brute force |
| 8080 | TCP | HTTP-alt | Web proxy, dev servers |

---

## 4. IP Manzillar

### IPv4 tuzilishi
```
192  .  168  .   1  .   1
 8bit    8bit   8bit   8bit  =  32bit
Oktet1  Oktet2 Oktet3 Oktet4
```

### Private IP diapazonlar (Internet'da yo'q)
```
10.0.0.0     – 10.255.255.255    (Class A — katta tarmoqlar)
172.16.0.0   – 172.31.255.255    (Class B — o'rta tarmoqlar)
192.168.0.0  – 192.168.255.255   (Class C — uy/ofis tarmoqlari)
127.0.0.1                         (Loopback — o'zingiz)
```

### CIDR Notation
```
192.168.1.0/24  →  192.168.1.0 – 192.168.1.255  (256 ta IP)
192.168.1.0/25  →  192.168.1.0 – 192.168.1.127  (128 ta IP)
192.168.1.0/16  →  192.168.0.0 – 192.168.255.255 (65536 ta IP)
```

---

## 5. DNS (Domain Name System)

### DNS qanday ishlaydi?
```
Siz: "google.com ga kir"
    ↓
1. Browser cache tekshiradi  →  topilsa, tugadi
2. OS /etc/hosts tekshiradi  →  topilsa, tugadi
3. Local DNS (router) so'raydi
4. ISP DNS so'raydi
5. Root DNS server  (.com boshqaruvchisini ko'rsatadi)
6. TLD DNS (.com server)  →  google.com NS ni ko'rsatadi
7. Authoritative DNS  →  google.com = 142.250.x.x
    ↓
Javob keshlanadi va brauzerga beriladi
```

### DNS Record turlari
| Record | Vazifa | Misol |
|--------|--------|-------|
| **A** | Domain → IPv4 | `google.com → 142.250.1.1` |
| **AAAA** | Domain → IPv6 | `google.com → 2607:f8b0::1` |
| **CNAME** | Domain → boshqa domain | `www → google.com` |
| **MX** | Mail server | `mail.google.com` |
| **TXT** | Matn ma'lumoti | SPF, DKIM, verify tokens |
| **NS** | Name server | DNS server manzili |
| **PTR** | IP → Domain (reverse) | `1.1.250.142 → google.com` |

### DNS Buyruqlari
```bash
# Domain IP ni topish:
nslookup google.com
dig google.com A

# Mail server:
dig google.com MX

# Name server:
dig google.com NS

# Barcha recordlar:
dig google.com ANY

# Reverse lookup (IP → domain):
dig -x 8.8.8.8

# Zone transfer urinish (recon):
dig axfr @ns1.target.com target.com
```

---

## 6. HTTP/HTTPS Asoslari

### HTTP Request tuzilishi
```http
GET /login HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Cookie: session=abc123
Accept: text/html

(body — GET'da bo'lmaydi)
```

### HTTP Response tuzilishi
```http
HTTP/1.1 200 OK
Content-Type: text/html
Set-Cookie: session=xyz789; HttpOnly
Server: nginx/1.18

<html>...</html>
```

### HTTP Status kodlar
| Kod | Ma'nosi | Ahamiyat |
|-----|---------|---------|
| 200 | OK | Muvaffaqiyat |
| 301 | Moved Permanently | Redirect |
| 302 | Found (temp redirect) | Login redirect |
| 400 | Bad Request | Noto'g'ri so'rov |
| 401 | Unauthorized | Login kerak |
| 403 | Forbidden | Ruxsat yo'q |
| 404 | Not Found | Sahifa yo'q |
| 500 | Internal Server Error | Server xatosi |
| 502 | Bad Gateway | Proxy xatosi |

### HTTP Metodlar
| Metod | Vazifa | Xavf |
|-------|--------|------|
| GET | Ma'lumot olish | Low |
| POST | Ma'lumot yuborish | Medium |
| PUT | Ma'lumot yangilash | High |
| DELETE | O'chirish | High |
| HEAD | Faqat headerlar | Low |
| OPTIONS | Ruxsat etilgan metodlar | Info leak |
| PATCH | Qisman yangilash | Medium |

---

## 7. Subnetting — Tez hisoblash

```
/24  →  255.255.255.0   →  254 host
/25  →  255.255.255.128 →  126 host
/26  →  255.255.255.192 →   62 host
/27  →  255.255.255.224 →   30 host
/28  →  255.255.255.240 →   14 host
/29  →  255.255.255.248 →    6 host
/30  →  255.255.255.252 →    2 host
```

---

## ⚡ Tez Referans Buyruqlar

```bash
# O'z IP manzilim:
ip addr show          # Linux
ipconfig              # Windows

# Tarmoqdagi qurilmalar:
arp -a

# Route table:
ip route show         # Linux
route print           # Windows

# Ping (ICMP):
ping -c 4 8.8.8.8     # Linux (-c = count)
ping -n 4 8.8.8.8     # Windows (-n = count)

# Traceroute:
traceroute 8.8.8.8    # Linux
tracert 8.8.8.8       # Windows

# Ochiq portlar:
netstat -tulnp         # Linux
netstat -ano           # Windows

# DNS manual:
cat /etc/resolv.conf   # DNS server manzili (Linux)
```

---

*TryHackMe Pre Security | Networking Fundamentals*
