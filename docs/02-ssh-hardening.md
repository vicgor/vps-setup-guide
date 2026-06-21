# 02 — SSH Hardening

## Цель
Защитить SSH: использовать только ключи, запретить вход под root.

---

## 1. Генерация SSH-ключа (на локальной машине)

```bash
# Ed25519 — современный и быстрый алгоритм
ssh-keygen -t ed25519 -C "my-vps-$(date +%Y%m%d)"
```

## 2. Копирование ключа на сервер

```bash
ssh-copy-id myuser@YOUR_SERVER_IP

# Или вручную:
cat ~/.ssh/id_ed25519.pub | ssh myuser@YOUR_SERVER_IP \
  'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys'
```

## 3. Проверка входа по ключу

```bash
ssh myuser@YOUR_SERVER_IP
# Должен войти без пароля
```

## 4. Настройка sshd_config

```bash
nano /etc/ssh/sshd_config
```

Параметры для установки:

```text
Port 2222
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
AllowUsers myuser
X11Forwarding no
AllowTcpForwarding no
MaxAuthTries 3
LoginGraceTime 20
ClientAliveInterval 300
ClientAliveCountMax 2
```

> ⚠️ Нестандартный порт (2222 вместо 22) снижает шум от автоматических сканеров. Не забудь открыть его в UFW перед перезапуском SSH.

## 5. Проверка конфига и перезапуск

```bash
# Проверить синтаксис
sshd -t

# Перезапустить SSH
systemctl restart ssh
```

## 6. Проверка в новом терминале

```bash
# Должен работать
ssh myuser@YOUR_SERVER_IP -p 2222

# Должен быть запрещён
ssh root@YOUR_SERVER_IP -p 2222  # Permission denied
```

---

> ⚠️ **Важно:** Не закрывай текущую root-сессию до успешной проверки входа!

## Следующий шаг
→ [03-ufw-firewall.md](03-ufw-firewall.md)
