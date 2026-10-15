# Easy rooms — RU

Прохождения TryHackMe (русский). Флаги скрыты (`THM{REDACTED}`) по правилам THM.

| Комната | Ключевые техники |
|---|---|
| [simple-ctf](simple-ctf/) | SQLi в CMS Made Simple (CVE-2019-9053) → privesc через `sudo vim` |
| [pyrat](pyrat/) | RCE через Python eval → утечка учёток в git → брут → root |
| [billing](billing/) | RCE в MagnusBilling (CVE-2023-30258) → privesc через `fail2ban-client` |
| [silver-platter](silver-platter/) | Брут Silverpeas (Hydra + CeWL) |
| [lookup](lookup/) | Брут логина → vhost → command injection в elFinder |
| [checkmate](checkmate/) | Энумерация мульти-сервиса Flask → вывод пароля |
| [grep](grep/) | Разведка, OSINT по TLS-серту, выставленный git / API-ключ (в процессе) |
