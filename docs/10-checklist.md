# 10 — Итоговый чек-лист

Проверь каждый пункт перед деплоем первого приложения.

---

## Система

- [ ] Система обновлена (`apt update && apt upgrade -y`)
- [ ] Установлены базовые пакеты (curl, git, htop, nano)
- [ ] Настроена временная зона (`timedatectl`)
- [ ] Настроен hostname (`hostnamectl`)

## Пользователи

- [ ] Создан непривилегированный пользователь
- [ ] Пользователь добавлен в группу sudo
- [ ] Вход под новым пользователем проверен
- [ ] Root-вход по SSH отключён (`PermitRootLogin no`)

## SSH

- [ ] SSH-ключ сгенерирован (ed25519)
- [ ] Ключ скопирован на сервер
- [ ] Вход по паролю отключён (`PasswordAuthentication no`)
- [ ] Конфиг проверен (`sshd -t`)

## Firewall (UFW)

- [ ] UFW включён (`ufw enable`)
- [ ] Политика по умолчанию: deny incoming
- [ ] Открыты: 22/tcp, 80/tcp, 443/tcp
- [ ] Rate limit на SSH (`ufw limit 22/tcp`)
- [ ] Порты мониторинга ограничены по IP

## Fail2ban

- [ ] Fail2ban установлен и запущен
- [ ] SSH-джейл активен (`fail2ban-client status sshd`)
- [ ] Свой IP добавлен в ignoreip

## Автообновления

- [ ] unattended-upgrades установлен и настроен
- [ ] Dry-run прошёл без ошибок

## Docker

- [ ] Docker установлен (`docker --version`)
- [ ] Docker Compose установлен (`docker compose version`)
- [ ] daemon.json настроен (логи, no-new-privileges)
- [ ] Пользователь добавлен в группу docker

## Nginx

- [ ] Nginx установлен и запущен
- [ ] Конфиги проверены (`nginx -t`)
- [ ] HTTPS настроен через Certbot
- [ ] Security headers добавлены

## Мониторинг

- [ ] Netdata установлен
- [ ] Доступ к Netdata ограничен по IP или через SSH-туннель

---

> ✅ Все пункты отмечены? Сервер готов к деплою приложений.
