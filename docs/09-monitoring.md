# 09 — Мониторинг: Netdata

## Цель
Настроить мониторинг VPS в реальном времени через Netdata.

---

## 1. Установка Netdata

```bash
wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh
sh /tmp/netdata-kickstart.sh --stable-channel --disable-telemetry
```

## 2. Ограничить доступ к веб-интерфейсу

Netdata по умолчанию слушает на порту 19999. Закрыть от всех, кроме своего IP:

```bash
# UFW: разрешить только с вашего IP
ufw allow from YOUR_IP to any port 19999
```

## 3. Базовая конфигурация

```bash
nano /etc/netdata/netdata.conf
```

```ini
[global]
    run as user = netdata
    web files owner = root
    web files group = root

[web]
    bind to = 127.0.0.1
    # Или конкретный IP: bind to = YOUR_SERVER_IP
```

```bash
systemctl restart netdata
```

## 4. Доступ через SSH-туннель (безопаснее)

Если UFW закрывает 19999 от всех, можно пробросить порт через SSH:

```bash
# На локальной машине
ssh -L 19999:localhost:19999 myuser@YOUR_SERVER_IP -p 2222

# Открыть в браузере
http://localhost:19999
```

## 5. Полезные метрики для контроля

- **CPU**: load average, per-process usage
- **Memory**: RAM, swap
- **Disk**: I/O, пространство
- **Network**: incoming/outgoing трафик
- **Docker**: метрики контейнеров (автоопределяется)

## 6. Мониторинг Nginx-логов (web_log collector)

По умолчанию Netdata парсит только `/var/log/nginx/access.log` (default server). На этот лог попадает мусорный трафик сканеров — запросы с нестандартным форматом вызывают алерт `web_log_1m_unmatched`.

**Правильная конфигурация** — подключить реальные vhost-логи в `/etc/netdata/go.d/web_log.conf`:

```yaml
jobs:
  - name: nginx_default
    path: /var/log/nginx/access.log
    log_type: auto

  - name: doi.by
    path: /var/log/nginx/doi.by_access.log
    log_type: auto

  - name: matrix-federation
    path: /var/log/nginx/matrix-federation.access.log
    log_type: auto
```

Заглушить алерт для default server (там всегда будет мусор от сканеров):

```bash
nano /etc/netdata/health.d/web_log_silence.conf
```

```
 template: web_log_1m_unmatched
       on: web_log_nginx_default.excluded_requests
     info: ignored — default server receives scanner traffic only
       to: silent
```

```bash
systemctl restart netdata
```

## 7. Алёрты и уведомления

```bash
# Настройка уведомлений
nano /etc/netdata/health_alarm_notify.conf

# Telegram-уведомления
TELEGRAM_BOT_TOKEN="your_bot_token"
TELEGRAM_CHAT_ID="your_chat_id"
DEFAULT_RECIPIENT_TELEGRAM="${TELEGRAM_CHAT_ID}"
```

---

## Следующий шаг
→ [10-checklist.md](10-checklist.md)
