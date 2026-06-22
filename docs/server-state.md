# Текущее состояние сервера

Снимок реальной конфигурации сервера на 2026-06-22.

---

## Система

| Параметр | Значение |
|----------|----------|
| OS | Ubuntu 24.04.4 LTS |
| Ядро | 6.8.0-124-generic |
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

Активный drop-in файл: `/etc/ssh/sshd_config.d/99-hardening.conf`:

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

Старый файл отключён: `hardening.conf.disabled.2026-06-09-2049`.

> Основной `/etc/ssh/sshd_config`: `X11Forwarding yes` закомментирован, конфликтов нет.

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
| 3478/tcp, 5349/tcp | TURN/STUN (coturn) |
| **19999/tcp** | **Netdata — UFW: allow only 91.149.143.96** |

**Только локально (127.0.0.1):**

DNS, Redis, все внутренние порты Matrix и LiveKit — корректно.

---

## Docker

Версия: `29.6.0`, Compose: `v5.1.4`

`/etc/docker/daemon.json`:

```json
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "10m", "max-file": "3" },
  "live-restore": true,
  "userland-proxy": false,
  "no-new-privileges": true
}
```

### Запущенные контейнеры

| Контейнер | Образ | Порты |
|-----------|-------|-------|
| livekit-jwt | ghcr.io/element-hq/lk-jwt-service:latest-ci | `127.0.0.1:8880->8080` |
| livekit-server | livekit/livekit-server:latest | — |
| livekit-redis | redis:7-alpine | — |
| matrix-element | vectorim/element-web:latest | `127.0.0.1:8009->80` |
| matrix-synapse | matrixdotorg/synapse:latest | `127.0.0.1:8008->8008`, `127.0.0.1:8048->8008` |
| matrix-postgres | postgres:16-alpine | — |

**Остановлены (Exited):**

| Контейнер | Статус |
|-----------|--------|
| geo-bot-app-1 | ⚠️ Exited (1) — упал ~8 ч назад |
| geo-bot-postgres-1 | Exited (0) |
| geo-bot-redis-1 | Exited (0) |

### Расположение compose-проектов

```
/opt/matrix/docker-compose.yml               — Matrix (Synapse + Element + Postgres)
/opt/livekit/docker-compose.yml              — LiveKit + JWT-сервис + Redis
/home/shoostrik/geo-bot/docker-compose.prod.yml — geo-bot (App + Postgres + Redis)
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

Статус: active. Слушает на `0.0.0.0:19999`, но UFW ограничивает доступ:

| Правило | Действие |
|---------|----------|
| `from 91.149.143.96 to any port 19999` | ALLOW |
| `19999` | DENY (Anywhere) |
| `19999 (v6)` | DENY (Anywhere v6) |

Доступен только с IP оператора. SSH-туннель не требуется.

---

## Xray

Протокол: **VLESS + REALITY** — маскируется под TLS-трафик `www.microsoft.com`.

- Слушает на `127.0.0.1:8443` (за Nginx, SNI-routing на порту 443)
- Flow: `xtls-rprx-vision`
- 1 клиент: `deploy@doi.by`
- DNS через DoH: Cloudflare + Google
- Запускается от `nobody`, `NoNewPrivileges=true`
- Роутинг блокирует: приватные IP, BitTorrent, рекламные домены

Конфиг: `/usr/local/etc/xray/config.json` — права `640 root:nogroup` (xray запускается как `nobody` ∈ `nogroup`), бэкапы удалены.

---

## CrowdSec

Состав:

- `crowdsec` — агент, анализирует логи и детектирует атаки
- `crowdsec-firewall-bouncer-iptables` — блокирует IP через iptables + ipset
- Дополнительно использует облачный фид репутации IP-адресов

Версия: `1.7.8`, bouncer: `0.0.34`.

---

## Ubuntu Pro

Подписка: **Ubuntu Pro — free personal** (`vicgor@gmail.com`)

| Сервис | Статус | Описание |
|--------|--------|----------|
| `esm-infra` | ✅ enabled | Расширенные патчи безопасности для main-пакетов |
| `esm-apps` | ✅ enabled | Расширенные патчи безопасности для universe-пакетов |
| `livepatch` | ✅ enabled | Патчи ядра без перезагрузки |
| `usg` | ✅ enabled | Аудит соответствия CIS/DISA |
| `fips-updates` | — disabled | Не требуется |
| `realtime-kernel` | — disabled | Не требуется |

---

## Что требует внимания

| Приоритет | Проблема | Действие |
|-----------|----------|----------|
| 🔴 Срочно | `geo-bot-app-1` упал с exit code 1 (~8 ч назад) | Проверить логи: `docker logs geo-bot-app-1` |
| 🟡 Средний | `geo-bot` размещён в `/home/shoostrik/`, не в `/opt/` | Нарушение конвенции; вынести в `/opt/geo-bot/` |
