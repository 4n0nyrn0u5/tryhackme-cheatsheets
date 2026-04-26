# 🐧 Linux Command Line — Cheatsheet

> **Manba:** TryHackMe — Pre Security + Cyber Security 101  
> **Mavzular:** Navigatsiya · Fayllar · Permissions · Processes · Network · Pentest toollar

---

## 1. Navigatsiya va Fayl Tizimi

### Fayl tizimi tuzilishi
```
/                   ← Root (hamma narsa shu yerdan boshlanadi)
├── bin/            ← Asosiy buyruqlar (ls, cat, cp...)
├── etc/            ← Konfiguratsiya fayllari ⭐
├── home/           ← Foydalanuvchi papkalari
│   └── user/       ← Sizning home (~)
├── var/            ← Log, cache, database fayllari ⭐
│   └── log/        ← Log fayllar
├── tmp/            ← Vaqtinchalik fayllar (world-writable!)
├── usr/            ← O'rnatilgan dasturlar
├── opt/            ← Qo'shimcha dasturlar
├── root/           ← Root foydalanuvchi home
├── proc/           ← Jarayonlar (virtual)
└── dev/            ← Qurilmalar
```

### Navigatsiya buyruqlari
```bash
pwd                  # Hozirgi papka
ls                   # Papka tarkibi
ls -la               # Batafsil + yashirin fayllar
ls -lah              # + o'lcham (human-readable)
cd /etc              # Papkaga o'tish
cd ..                # Yuqoriga
cd ~                 # Home papkaga
cd -                 # Oldingi papkaga
```

---

## 2. Fayl Operatsiyalari

```bash
# Ko'rish:
cat file.txt              # Faylni chiqarish
less file.txt             # Sahifalab o'qish (q = chiqish)
head -n 20 file.txt       # Birinchi 20 qator
tail -n 20 file.txt       # Oxirgi 20 qator
tail -f /var/log/auth.log # Real-time monitoring ⭐

# Yaratish va tahrirlash:
touch file.txt            # Bo'sh fayl
nano file.txt             # Sodda editor
vim file.txt              # Kuchli editor

# Nusxa va ko'chirish:
cp file.txt /tmp/         # Nusxa
cp -r folder/ /tmp/       # Papka nusxasi
mv file.txt newname.txt   # Ko'chirish / Nomini o'zgartirish
rm file.txt               # O'chirish
rm -rf folder/            # Papkani o'chirish (EHTIYOT!)

# Yaratish:
mkdir myfolder            # Papka
mkdir -p a/b/c            # Ichma-ich papkalar

# Qidirish:
find / -name "flag.txt" 2>/dev/null     # Fayl qidirish ⭐
find / -perm -4000 2>/dev/null          # SUID fayllar ⭐
find /home -type f -name "*.txt"        # .txt fayllar
locate passwords.txt                    # locate DB dan qidirish

# Matn qidirish:
grep "password" file.txt                # Faylda qidirish
grep -r "password" /etc/               # Rekursiv
grep -i "pass" file.txt                # Case-insensitive
grep -n "error" file.txt               # Qator raqami bilan
grep -v "comment" file.txt             # Teskari (mos bo'lmaganlar)
grep -oP 'picoCTF\{[^}]+\}' file.txt  # Regex bilan

# Saralash:
sort file.txt                          # Saralash
sort -r file.txt                       # Teskari
uniq                                   # Takrorlanganlarni olib tashlash
sort file.txt | uniq -c | sort -rn    # Eng ko'p takroran ⭐
```

---

## 3. File Permissions

### Ruxsat tuzilishi
```
-  rwx  rwx  rwx
│   │    │    └── Others (boshqalar)
│   │    └─────── Group (guruh)
│   └──────────── Owner (egasi)
└──────────────── Fayl turi (- = fayl, d = papka, l = link)

r = Read    (o'qish)   = 4
w = Write   (yozish)   = 2
x = Execute (bajarish) = 1
```

### Misol: `chmod`
```bash
chmod 755 file.sh     # rwxr-xr-x
chmod 644 file.txt    # rw-r--r--
chmod +x script.sh    # Bajarish ruxsati qo'shish
chmod -w file.txt     # Yozish ruxsatini olish
chmod u+s binary      # SUID bit (privesc!)

# Hisoblash:
# 7 = 4+2+1 = rwx
# 5 = 4+0+1 = r-x
# 4 = 4+0+0 = r--
# 6 = 4+2+0 = rw-
```

### Egasini o'zgartirish
```bash
chown user:group file.txt
chown -R user:group folder/
```

### SUID/SGID — Privilege Escalation uchun ⭐
```bash
# SUID (s bit) — faylni egasi huquqida bajarish
find / -perm -4000 -type f 2>/dev/null

# SGID — guruh huquqida bajarish
find / -perm -2000 -type f 2>/dev/null

# World-writable fayllar (xavfli!)
find / -perm -0002 -type f 2>/dev/null
```

---

## 4. Users va Groups

```bash
# Foydalanuvchi haqida:
whoami                # Hozirgi user
id                    # UID, GID, groups
who                   # Tizimga kirganlar
w                     # Kimlar nima qilmoqda

# Foydalanuvchilar ro'yxati:
cat /etc/passwd       # Barcha userlar ⭐
cat /etc/shadow       # Parol hashlari (root kerak)
cat /etc/group        # Guruhlar

# Switch user:
su username           # User o'zgartirish
su -                  # Root bo'lish (parol kerak)
sudo command          # Root huquqida bajarish
sudo -l               # Nimani sudo bilan bajarish mumkin ⭐

# User yaratish (root):
useradd -m newuser
passwd newuser
usermod -aG sudo newuser  # Sudo guruhiga qo'shish
```

### /etc/passwd formati
```
root    : x  : 0    : 0    : root    : /root   : /bin/bash
username: pw : UID  : GID  : Comment : Home    : Shell
```
- `x` — parol `/etc/shadow` da
- UID 0 = root huquqlari ⭐

---

## 5. Processes (Jarayonlar)

```bash
# Ko'rish:
ps aux                    # Barcha jarayonlar ⭐
ps aux | grep apache      # Muayyan jarayon
top                       # Real-time monitoring
htop                      # Interaktiv (ko'proq yaxshi)
pgrep nginx               # PID topish

# Boshqarish:
kill 1234                 # Jarayonni o'ldirish
kill -9 1234              # Majburan o'ldirish
killall apache2           # Nom bilan o'ldirish
pkill firefox             # Pattern bilan

# Background/Foreground:
command &                 # Background'da ishga tushirish
jobs                      # Background jarayonlar
fg %1                     # Foreground'ga qaytarish
Ctrl+Z                    # Pause (background'ga)
Ctrl+C                    # To'xtatish

# Cron jobs (scheduled tasks):
crontab -l                # Hozirgi user cron ⭐
cat /etc/crontab          # Tizim cron ⭐
ls /etc/cron.d/           # Qo'shimcha cron fayllar

# Cron format:
# * * * * * command
# │ │ │ │ └── Hafta kuni (0-7)
# │ │ │ └──── Oy (1-12)
# │ │ └────── Kun (1-31)
# │ └──────── Soat (0-23)
# └────────── Daqiqa (0-59)
```

---

## 6. Network Buyruqlari

```bash
# IP manzillar:
ip addr show              # Barcha interfacelar (ifconfig o'rniga)
ip addr show eth0         # Muayyan interface
hostname -I               # Faqat IP'lar

# Ulanishlar:
netstat -tulnp            # Tinglayotgan portlar ⭐
ss -tulnp                 # netstat ning yangi versiyasi ⭐
netstat -an | grep ESTABLISHED  # Joriy ulanishlar

# DNS:
cat /etc/resolv.conf      # DNS server
cat /etc/hosts            # Local DNS ⭐ (poisoning uchun)

# Fayl yuklash:
wget http://10.10.10.10/file.txt
curl -O http://10.10.10.10/file.txt
curl http://10.10.10.10/file.txt -o savename.txt

# HTTP so'rov yuborish:
curl http://site.com                        # GET
curl -X POST http://site.com/login \
     -d "user=admin&pass=password"          # POST
curl -H "Cookie: session=abc" http://site/  # Header
curl -v http://site.com                     # Verbose (headerlar ko'rish)
curl -I http://site.com                     # Faqat headerlar (HEAD)
curl -k https://site.com                    # SSL tekshiruvsiz
```

---

## 7. Pipes va Redirection

```bash
# Pipe (|) — bir buyruq chiqishini ikkinchisiga:
ps aux | grep python
cat /etc/passwd | grep bash
ls -la | sort -k5 -n     # o'lcham bo'yicha saralash

# Redirection:
command > file.txt        # Faylga yozish (ustiga)
command >> file.txt       # Faylga qo'shish
command 2> errors.txt     # Xatoliklarni faylga
command 2>/dev/null       # Xatoliklarni o'chirish ⭐
command &> all.txt        # Hammani faylga

# tee — ekranga ham, faylga ham:
nmap -sV target | tee scan_results.txt
```

---

## 8. Foydali Shortcutlar va Triklar

```bash
# History:
history                   # Buyruqlar tarixi
history | grep nmap       # Qidirish
!!                        # Oldingi buyruqni qayta
!nmap                     # "nmap" bilan boshlanganini qayta
Ctrl+R                    # Tarixi qidirish (interaktiv)

# Terminal:
Ctrl+C                    # To'xtatish
Ctrl+Z                    # Pause
Ctrl+D                    # Chiqish (exit)
Ctrl+L                    # Tozalash (clear)
Tab                       # Avtomatik to'ldirish ⭐
Tab Tab                   # Variantlarni ko'rish

# Environment:
echo $PATH                # PATH o'zgaruvchi
echo $HOME                # Home papka
env                       # Barcha environment variables ⭐
export MYVAR="value"      # O'zgaruvchi yaratish

# Fayl mazmuni tez ko'rish:
file suspicious            # Fayl turi aniqlash ⭐
strings binary_file        # Binary'dan matn ⭐
xxd file.bin | head        # Hex ko'rish
base64 -d encoded.txt      # Base64 decode
```

---

## 9. Pentest uchun Muhim Fayllar ⭐

```bash
# Konfiguratsiya va sirlar:
cat /etc/passwd            # Userlar
cat /etc/shadow            # Parol hashlari (root)
cat /etc/crontab           # Scheduled tasks
cat /etc/hosts             # Local DNS
cat ~/.bash_history        # Buyruqlar tarixi
cat ~/.ssh/id_rsa          # Private SSH key
find / -name "*.conf" 2>/dev/null
find / -name "*.config" 2>/dev/null
find / -name "config.php" 2>/dev/null
find / -name ".env" 2>/dev/null        # Environment vars ⭐

# Log fayllar:
cat /var/log/auth.log      # Login'lar
cat /var/log/apache2/access.log   # Web server
cat /var/log/syslog        # Tizim loglari
ls /var/log/               # Barchasi

# Web server fayllar:
ls /var/www/html/          # Web root
cat /var/www/html/config.php
```

---

## 10. Bash Scripting — Asoslar

```bash
#!/bin/bash
# Birinchi qator — shebang

# O'zgaruvchilar:
name="Ali"
echo "Salom $name"

# Argumentlar:
echo "1-argument: $1"
echo "Barcha: $@"

# If-else:
if [ $1 -eq 0 ]; then
    echo "Nol"
elif [ $1 -gt 0 ]; then
    echo "Musbat"
else
    echo "Manfiy"
fi

# Loop:
for i in {1..10}; do
    echo "Raqam: $i"
done

# While:
while true; do
    echo "Ishlayapti..."
    sleep 1
done

# Funksiya:
greet() {
    echo "Salom, $1!"
}
greet "World"

# Buyruq chiqishini variable'ga:
result=$(whoami)
echo "User: $result"

# Pentest script misoli — port scanner:
for port in 22 80 443 8080 3306; do
    (echo >/dev/tcp/10.10.10.1/$port) 2>/dev/null && \
    echo "Port $port: OPEN"
done
```

---

*TryHackMe Pre Security | Linux Fundamentals*
