# Текущее состояние сервера

Снимок реальной конфигурации сервера на 2026-05-02.

---

## Система

| Параметр | Значение |
|----------|----------|
| OS | Ubuntu 24.04.4 LTS |
| Ядро | 6.8.0-110-generic |
| Hostname | vmi3240281 |
| Timezone | Europe/Minsk (+03) |
| IP | 194.163.161.167 |

---

## Пользователи

| Пользователь | UID | Группы |
|---|---|---|
| viktar | 1000 | sudo |
| deploy | 1001 | sudo, docker |

---

## SSH

Конфигурация вынесена в `/etc/ssh/sshd_config.d/hardening.conf`:

```text
Port                      2222
PermitRootLogin           no
PasswordAuthentication    no
KbdInteractiveAuthentication no
PubkeyAuthentication      yes
AuthorizedKeysFile        .ssh/authorized_keys
MaxAuthTries              3
LoginGraceTime            20
AllowUsers                deploy
X11Forwarding             no
AllowTcpForwarding        no
ClientAliveInterval       300
ClientAliveCountMax       2
```

> ⚠️ Основной `/etc/ssh/sshd_config` содержит `PermitRootLogin yes` и `X11Forwarding yes` — они перекрываются `hardening.conf`, т.к. `Include` идёт первым. Работает корректно, но стоит почистить основной файл.

> ⚠️ `AllowTcpForwarding no` блокирует SSH-туннель к Netdata (`ssh -L 19999:...`). Пока доступ к Netdata открыт по UFW.

---

## Сервисы

| Сервис | Статус |
|--------|--------|
| ssh | ✅ active |
| nginx | ✅ active |
| docker | ✅ active |
| netdata | ✅ active |
| unattended-upgrades | ✅ active |
| crowdsec | ✅ active |
| crowdsec-firewall-bouncer | ✅ active |
| coturn | ✅ active |
| xray | ✅ active |
| fail2ban | — не используется (заменён CrowdSec) |

---

## Firewall (UFW)

Проверка недоступна без `sudo`. По открытым портам (`ss -tlnp`) видно следующее:

**Открыто наружу:**

| Порт | Сервис |
|------|--------|
| 2222/tcp | SSH |
| 80/tcp | Nginx (HTTP) |
| 443/tcp | Nginx (HTTPS) |
| 8448/tcp | Matrix federation |
| 7880/tcp, 7881/tcp | LiveKit WebRTC |
| 3478/tcp, 5349/tcp | TURN/STUN (LiveKit) |
| **19999/tcp** | **Netdata ❌ — открыт без авторизации** |

**Только локально (127.0.0.1):**

DNS, Redis, все внутренние порты Matrix и LiveKit — корректно.

---

## Docker

Версия: `29.4.2`, Compose: `v5.1.3`

`/etc/docker/daemon.json` — **отсутствует** (нет лимитов логов, нет `live-restore`).

### Запущенные контейнеры

| Контейнер | Образ | Публичные порты |
|-----------|-------|-----------------|
| livekit-jwt | lk-jwt-service:latest-ci | `127.0.0.1:8880->8080` |
| livekit-server | livekit-server:latest | — |
| livekit-redis | redis:7-alpine | — |
| matrix-element | element-web:latest | `127.0.0.1:8009->80` |
| matrix-synapse | synapse:latest | `127.0.0.1:8008->8008`, `127.0.0.1:8048->8008` |
| matrix-postgres | postgres:16-alpine | — |

Все порты привязаны к `127.0.0.1` — корректно.

### Расположение compose-проектов

```
/opt/matrix/docker-compose.yml    — Matrix (Synapse + Element + Postgres)
/opt/livekit/docker-compose.yml   — LiveKit + JWT-сервис + Redis
```

---

## Nginx

Активные виртуальные хосты (`/etc/nginx/sites-enabled/`):

```
doi.by.conf
element.doi.by.conf
livekit.doi.by.conf
matrix-federation.conf
matrix.doi.by.conf
```

HTTPS через Let's Encrypt (`/etc/letsencrypt/live/doi.by/`).

---

## Netdata

Статус: active. Слушает на `0.0.0.0:19999` и `[::]:19999` — **доступен из интернета без авторизации**.

Требуется одно из:
- `bind to = 127.0.0.1` в `/etc/netdata/netdata.conf` + перезапуск
- или UFW-правило `ufw allow from YOUR_IP to any port 19999` (+ убрать общий доступ)

---

## Xray

Протокол: **VLESS + REALITY** — маскируется под TLS-трафик `www.microsoft.com`.

- Слушает на `127.0.0.1:8443` (за Nginx, SNI-routing на порту 443)
- Flow: `xtls-rprx-vision`
- 1 клиент: `deploy@doi.by`
- DNS через DoH: Cloudflare + Google
- Запускается от `nobody`, `NoNewPrivileges=true`
- Роутинг блокирует: приватные IP, BitTorrent, рекламные домены

Конфиг: `/usr/local/etc/xray/config.json`

> ⚠️ В `/usr/local/etc/xray/` лежат 3 файла бэкапов со старыми ключами — стоит удалить.

---

## CrowdSec

Используется вместо Fail2ban. Состав:

- `crowdsec` — агент, анализирует логи и детектирует атаки
- `crowdsec-firewall-bouncer-iptables` — блокирует IP через iptables + ipset
- Дополнительно использует облачный фид репутации IP-адресов

Версия: `1.7.7`, bouncer: `0.0.34`.

---

## Что требует внимания

| Приоритет | Проблема | Действие |
|-----------|----------|----------|
| 🔴 Высокий | Netdata открыт на 0.0.0.0:19999 | Ограничить через netdata.conf или UFW |
| 🟡 Средний | `/etc/docker/daemon.json` отсутствует | Добавить лимиты логов и `live-restore` |
| 🟡 Средний | Основной sshd_config содержит конфликтующие настройки | Почистить — убрать `PermitRootLogin yes`, `X11Forwarding yes` |
