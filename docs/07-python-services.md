# 07 — Python-сервисы и деплой

## Цель
Запустить Python-сервисы (Telegram-боты, CLI, API) через Docker или systemd.

---

## 1. Структура проекта на сервере

```bash
/opt/projects/
├── jira-telegram-bot/
│   ├── docker-compose.yml
│   ├── .env
│   └── app/
├── telegram-monitor/
│   ├── docker-compose.yml
│   └── app/
```

```bash
# Создать структуру
mkdir -p /opt/projects
chown myuser:myuser /opt/projects
```

## 2. Деплой через Docker Compose (рекомендуется)

```bash
# Клонировать проект
cd /opt/projects
git clone git@github.com:vicgor/jira-telegram-bot.git

# Создать .env из примера
cp .env.example .env
nano .env

# Запустить
docker compose up -d

# Проверить логи
docker compose logs -f
```

## 3. Systemd-юнит для Python-сервиса (без Docker)

```ini
# /etc/systemd/system/mybot.service
[Unit]
Description=My Telegram Bot
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=myuser
WorkingDirectory=/opt/projects/mybot
EnvironmentFile=/opt/projects/mybot/.env
ExecStart=/opt/projects/mybot/venv/bin/python -m app
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl enable --now mybot
systemctl status mybot
journalctl -u mybot -f
```

## 4. Виртуальное окружение Python

```bash
apt install python3-pip python3-venv -y

cd /opt/projects/mybot
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## 5. Управление секретами

```bash
# .env файл — НЕ коммитить в git!
# В .gitignore обязательно:
echo ".env" >> .gitignore

# Права на .env
chmod 600 .env
```

## 6. Обновление сервиса

```bash
cd /opt/projects/jira-telegram-bot
git pull
docker compose up -d --build

# Или для systemd:
git pull
systemctl restart mybot
```

---

## Следующий шаг
→ [08-unattended-upgrades.md](08-unattended-upgrades.md)
