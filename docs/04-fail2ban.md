# 04 — Fail2ban

## Цель
Автоматически блокировать IP при многократных неудачных попытках входа.

---

## 1. Установка

```bash
apt install fail2ban -y
systemctl enable --now fail2ban
```

## 2. Конфигурация SSH-джейла

Создаём override-файл (не трогаем jail.conf!):

```bash
nano /etc/fail2ban/jail.d/sshd.local
```

Содержимое:

```ini
[sshd]
enabled  = true
port     = ssh
logpath  = /var/log/auth.log
maxretry = 5
findtime = 10m
bantime  = 1h
ignoreip = 127.0.0.1/8 YOUR_IP
```

## 3. Перезапуск и проверка

```bash
systemctl restart fail2ban
systemctl status fail2ban

# Статус SSH-джейла
fail2ban-client status sshd
```

## 4. Управление банами

```bash
# Посмотреть все забаненные IP
fail2ban-client banned

# Разбанить IP
fail2ban-client set sshd unbanip 1.2.3.4

# Забанить IP вручную
fail2ban-client set sshd banip 1.2.3.4

# Статус всех джейлов
fail2ban-client status
```

## 5. Просмотр логов

```bash
tail -f /var/log/fail2ban.log
journalctl -u fail2ban -f
```

## 6. Джейл для Nginx (добавить позже)

```ini
# /etc/fail2ban/jail.d/nginx.local
[nginx-http-auth]
enabled  = true
port     = http,https
logpath  = /var/log/nginx/error.log
maxretry = 5
bantime  = 1h

[nginx-limit-req]
enabled  = true
port     = http,https
logpath  = /var/log/nginx/error.log
maxretry = 10
bantime  = 10m
```

---

## Следующий шаг
→ [05-docker-setup.md](05-docker-setup.md)
