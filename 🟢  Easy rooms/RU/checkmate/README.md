# Checkmate

**Сложность:** Easy
**Техники:** энумерация множества сервисов · вывод пароля из контента соседнего сайта (OSINT → пароль)

## Разведка
```bash
nmap -sS -Pn -p- -T4 -A -oN portscan.nmap <ip>
```
Пять Flask-сервисов (Werkzeug) на одном хосте:
```
22/tcp   ssh
5000/tcp http  "Operation Checkmate"  (API-пазл /check + /state)
5001/tcp http  "FirewallOS — Sign in" (firewall.thm:5001)
5002/tcp http  "Engineering Careers"  (jobs.thm:5002)
5003/tcp http  "social.thm — Log in"  (social.thm:5003)
```
Vhost'ы в `/etc/hosts`.

## Энумерация
```bash
gobuster dir -u http://<ip>:5000 -w .../common.txt
#   /state → {"progress":1} ; /check (POST) → {"message":"Incorrect ❌"}
gobuster dir -u http://<ip>:5002 -w .../common.txt -x html,txt,js
#   /login  /profile (→ /login)
```
- **5002 (jobs.thm)** — сайт вакансий. Плейсхолдер логина `marco`. Контент полон тематических слов (*innovation, excellence, security, cloud, digital, future*).
- **5003 (social.thm)** — явная подсказка: *«Use the details from jobs.thm to generate Marco's password»*.

## Подход — вывести пароль из контента
Идея: собрать словарь jobs.thm через CeWL и построить пароль Marco из лексики сайта, затем войти как `marco` на social.thm (и перейти по связанным сервисам).
```bash
cewl http://jobs.thm:5002 -w jobs.txt
```
Эндпоинт `/check` на 5000 — пазл-прогресс, гейтящий следующие шаги. *В заметках полная энумерация сервисов и стратегия вывода пароля; цепочку эксплуатации дописать.*

## Выводы
- Один хост, много микросервисов → энумерировать каждый порт; заголовки раскрывают vhost'ы.
- «Сгенерируй пароль из данных сайта» — сигнал CeWL/OSINT: публичный текст это источник паролей.
- Связанные демо-приложения с общей логикой учёток дают каскад от одного слабого аккаунта.
