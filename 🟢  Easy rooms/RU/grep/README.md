# Grep

**Сложность:** Easy
**Статус:** разведка / в процессе
**Техники:** энумерация сервисов · OSINT по TLS-сертификату · поиск утёкшего git / API-ключа

## Разведка
```bash
nmap -sS -sV -p- -oN nmap.scan <ip>
```
```
22/tcp    open  ssh      OpenSSH 8.2p1
80/tcp    open  http     Apache 2.4.41  (дефолтная страница)
443/tcp   open  ssl/http Apache 2.4.41
51337/tcp open  http     Apache 2.4.41  → 403 Forbidden
```
TLS-сертификат на 51337 сливает хост:
```bash
curl -vk https://<ip>:51337
#   subject/issuer CN = leakchecker.grep.thm
```
`leakchecker.grep.thm` в `/etc/hosts`.

## Энумерация
```bash
gobuster dir -u http://<ip> -w .../common.txt -x js,txt,html
#   index.php, /javascript/ → /jquery/, phpmyadmin (403), server-status (403)
```

## Рабочая гипотеза
Комната про сервис «leak checker». Собранные зацепки:
- **SuperSecure Corp** — компания (первая зацепка)
- **SearchME** — название сайта/проекта, которое надо найти
- Цель: *восстановить живой API-ключ из git-репозитория* — подсказка *«Search for API Key in Dumped Repository»* указывает на выставленный/дампнутый `.git`.

Дальше: найти приложение SearchME, сдампить историю git (git-dumper), грепнуть историю по API-ключу, использовать его для регистрации/аутентификации. *Writeup в процессе — foothold в заметках не достигнут.*

## Выводы
- TLS-сертификаты сливают внутренние хосты — всегда смотреть `subject`/`issuer`.
- Выставленные `.git` — повторяющаяся тема: дампить и грепать историю на секреты (API-ключи, пароли).
