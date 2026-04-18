# 01 — Первоначальная настройка сервера

## Цель
Подготовить безопасную базовую среду сразу после получения VPS.

---

## 1. Обновление системы

```bash
apt update
apt upgrade -y
apt dist-upgrade -y
apt autoremove -y
```

## 2. Установка базовых пакетов

```bash
apt install -y curl wget git htop nano ufw fail2ban unattended-upgrades apt-listchanges
```

## 3. Создание непривилегированного пользователя

```bash
# Создать пользователя
adduser myuser

# Добавить в группу sudo
usermod -aG sudo myuser

# Проверить
groups myuser
```

## 4. Проверка входа под новым пользователем

В **отдельном** терминале (не закрывая root-сессию):

```bash
ssh myuser@YOUR_SERVER_IP
sudo whoami  # должен вернуть: root
```

## 5. Настройка временной зоны

```bash
timedatectl set-timezone Europe/Minsk
timedatectl status
```

## 6. Настройка hostname

```bash
hostnamectl set-hostname my-vps
# Добавить в /etc/hosts
echo "127.0.1.1 my-vps" >> /etc/hosts
```

## 7. Настройка локали

```bash
localectl set-locale LANG=en_US.UTF-8
```

---

## Следующий шаг
→ [02-ssh-hardening.md](02-ssh-hardening.md)
