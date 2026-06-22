# VPS Setup Guide — Ubuntu 24.04 LTS

Базовое руководство по настройке и защите VPS сервера на Ubuntu 24.04 LTS.  
Заточено под стек: **Python · Docker · Nginx · MySQL · Telegram-боты**.

## Структура документов

| Файл | Описание |
|------|----------|
| [01-initial-setup.md](docs/01-initial-setup.md) | Первоначальная настройка: пользователь, sudo, обновления |
| [02-ssh-hardening.md](docs/02-ssh-hardening.md) | SSH: ключи, отключение root, sshd_config |
| [03-ufw-firewall.md](docs/03-ufw-firewall.md) | UFW: правила для SSH / HTTP / HTTPS |
| [04-crowdsec.md](docs/04-crowdsec.md) | CrowdSec: защита от brute-force и атак |
| [05-docker-setup.md](docs/05-docker-setup.md) | Docker + Docker Compose |
| [06-nginx-reverse-proxy.md](docs/06-nginx-reverse-proxy.md) | Nginx как reverse proxy |
| [07-python-services.md](docs/07-python-services.md) | Деплой Python-сервисов, systemd-юниты |
| [08-unattended-upgrades.md](docs/08-unattended-upgrades.md) | Автоматические обновления безопасности |
| [09-monitoring.md](docs/09-monitoring.md) | Мониторинг: Netdata |
| [10-checklist.md](docs/10-checklist.md) | Итоговый чек-лист |
| [11-vless-xray.md](docs/11-vless-xray.md) | VLESS + Xray (VLESS + Reality) — опционально |
| [12-backup.md](docs/12-backup.md) | Резервное копирование: Postgres, конфиги, systemd timer |

## Быстрый старт

```bash
# 1. Обновить систему
apt update && apt upgrade -y

# 2. Создать пользователя
adduser myuser && usermod -aG sudo myuser

# 3. Настроить UFW
ufw allow 2222/tcp && ufw allow 80/tcp && ufw allow 443/tcp && ufw enable

# 4. Установить CrowdSec
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | bash
apt install crowdsec crowdsec-firewall-bouncer-iptables -y
```

## Окружение

- **OS:** Ubuntu 24.04 LTS (Noble Numbat)
- **Стек:** Python 3.11+, Docker, Nginx, MySQL 8
- **Инструменты:** UFW, CrowdSec, unattended-upgrades, Netdata

---
> Документы обновляются по мере развития инфраструктуры.
