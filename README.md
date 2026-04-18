# VPS Setup Guide — Ubuntu 24.04 LTS

Базовое руководство по настройке и защите VPS сервера на Ubuntu 24.04 LTS.  
Заточено под стек: **Python · Docker · Nginx · MySQL · Telegram-боты**.

## Структура документов

| Файл | Описание |
|------|----------|
| [01-initial-setup.md](docs/01-initial-setup.md) | Первоначальная настройка: пользователь, sudo, обновления |
| [02-ssh-hardening.md](docs/02-ssh-hardening.md) | SSH: ключи, отключение root, sshd_config |
| [03-ufw-firewall.md](docs/03-ufw-firewall.md) | UFW: правила для SSH / HTTP / HTTPS |
| [04-fail2ban.md](docs/04-fail2ban.md) | Fail2ban: защита SSH от brute-force |
| [05-docker-setup.md](docs/05-docker-setup.md) | Docker + Docker Compose |
| [06-nginx-reverse-proxy.md](docs/06-nginx-reverse-proxy.md) | Nginx как reverse proxy |
| [07-python-services.md](docs/07-python-services.md) | Деплой Python-сервисов, systemd-юниты |
| [08-unattended-upgrades.md](docs/08-unattended-upgrades.md) | Автоматические обновления безопасности |
| [09-monitoring.md](docs/09-monitoring.md) | Мониторинг: Netdata |
| [10-checklist.md](docs/10-checklist.md) | Итоговый чек-лист |

## Быстрый старт

```bash
# 1. Обновить систему
apt update && apt upgrade -y

# 2. Создать пользователя
adduser myuser && usermod -aG sudo myuser

# 3. Настроить UFW
ufw allow 22/tcp && ufw allow 80/tcp && ufw allow 443/tcp && ufw enable

# 4. Установить Fail2ban
apt install fail2ban -y
```

## Окружение

- **OS:** Ubuntu 24.04 LTS (Noble Numbat)
- **Стек:** Python 3.11+, Docker, Nginx, MySQL 8
- **Инструменты:** UFW, Fail2ban, unattended-upgrades, Netdata

---
> Документы обновляются по мере развития инфраструктуры.
