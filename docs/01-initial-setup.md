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
apt install -y curl wget git htop nano ufw unattended-upgrades apt-listchanges
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

## 8. Ubuntu Pro (опционально)

Бесплатная подписка для физических лиц (до 5 машин). Даёт расширенные патчи безопасности (ESM), Livepatch и аудит-инструменты.

```bash
# Установить клиент (обычно уже есть в Ubuntu 22.04+)
apt install ubuntu-advantage-tools -y

# Привязать машину к аккаунту (токен — на ubuntu.com/pro)
pro attach YOUR_TOKEN
```

Рекомендуемые сервисы после привязки:

```bash
pro enable esm-infra    # расширенные патчи для main-пакетов
pro enable esm-apps     # расширенные патчи для universe-пакетов
pro enable livepatch    # патчи ядра без перезагрузки
```

Проверка статуса:

```bash
pro status
```

---

## Следующий шаг
→ [02-ssh-hardening.md](02-ssh-hardening.md)
