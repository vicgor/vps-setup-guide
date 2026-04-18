# 03 — UFW Firewall

## Цель
Настроить файрвол: разрешить только необходимые порты.

---

## 1. Установка UFW

```bash
apt install ufw -y
```

## 2. Политики по умолчанию

```bash
ufw default deny incoming
ufw default allow outgoing
```

## 3. Разрешить необходимые порты

```bash
# SSH — обязательно до включения UFW!
ufw allow 22/tcp

# HTTP и HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Rate limit для SSH (защита от brute-force)
ufw limit 22/tcp
```

## 4. Включить UFW

```bash
ufw enable

# Проверить статус
ufw status verbose
```

## 5. Полезные команды

```bash
# Список правил с номерами
ufw status numbered

# Удалить правило по номеру
ufw delete 3

# Разрешить порт только с конкретного IP
ufw allow from 91.149.143.96 to any port 8080

# Запретить конкретный IP
ufw deny from 1.2.3.4

# Перезагрузить правила
ufw reload
```

## 6. Правила для будущих сервисов

```bash
# Netdata мониторинг — только с вашего IP
ufw allow from YOUR_IP to any port 19999

# MySQL — только локально (не открывать в интернет!)
# По умолчанию MySQL слушает 127.0.0.1, дополнительных правил не нужно
```

---

## Следующий шаг
→ [04-fail2ban.md](04-fail2ban.md)
