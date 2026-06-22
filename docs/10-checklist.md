# 10 — Итоговый чек-лист

Проверь каждый пункт перед деплоем первого приложения.

---

## Система

- [ ] Система обновлена (`apt update && apt upgrade -y`)
- [ ] Установлены базовые пакеты (curl, git, htop, nano)
- [ ] Настроена временная зона (`timedatectl`)
- [ ] Настроен hostname (`hostnamectl`)

## Система

- [ ] Swap настроен (`swapon --show`)
- [ ] `vm.swappiness=10` в `/etc/sysctl.d/99-swappiness.conf`
- [ ] `/swapfile` добавлен в `/etc/fstab`

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
- [ ] Открыты: 2222/tcp, 80/tcp, 443/tcp
- [ ] Rate limit на SSH (`ufw limit 2222/tcp`)
- [ ] Порты мониторинга ограничены по IP

## CrowdSec

- [ ] CrowdSec установлен и запущен (`systemctl status crowdsec`)
- [ ] Bouncer зарегистрирован (`cscli bouncers list`)
- [ ] SSH-коллекция активна (`cscli collections list`)
- [ ] Свой IP добавлен в whitelist

## Ubuntu Pro (опционально)

- [ ] Машина привязана к аккаунту (`pro attach`)
- [ ] `esm-infra` включён (`pro enable esm-infra`)
- [ ] `esm-apps` включён (`pro enable esm-apps`)
- [ ] `livepatch` включён (`pro enable livepatch`)
- [ ] Статус проверен (`pro status`)

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

## VLESS + Xray (опционально)

- [ ] Xray установлен (`/usr/local/bin/xray`)
- [ ] Конфиг VLESS + Reality создан (`/usr/local/etc/xray/config.json`)
- [ ] UUID и ключи Reality сгенерированы и сохранены
- [ ] Xray запущен (`systemctl status xray`)
- [ ] Порт 443 открыт в UFW
- [ ] Клиент настроен и подключение проверено

---

> ✅ Все пункты отмечены? Сервер готов к деплою приложений.

## Следующий шаг
→ [11-vless-xray.md](11-vless-xray.md)
