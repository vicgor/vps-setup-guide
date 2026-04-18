# 08 — Автоматические обновления безопасности

## Цель
Настроить автоматическую установку security-патчей без ручного вмешательства.

---

## 1. Установка

```bash
apt install unattended-upgrades apt-listchanges -y
dpkg-reconfigure --priority=low unattended-upgrades
# Выбрать: Yes
```

## 2. Настройка /etc/apt/apt.conf.d/50unattended-upgrades

```bash
nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Ключевые параметры:

```text
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
    "${distro_id}:${distro_codename}-updates";
};

// Автоматически удалять неиспользуемые зависимости
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Remove-New-Unused-Dependencies "true";

// Автоперезагрузка при обновлении ядра (опционально)
// Unattended-Upgrade::Automatic-Reboot "true";
// Unattended-Upgrade::Automatic-Reboot-Time "03:30";
```

## 3. Расписание обновлений

```bash
nano /etc/apt/apt.conf.d/20auto-upgrades
```

```text
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
APT::Periodic::AutocleanInterval "7";
```

## 4. Проверка

```bash
# Тест без реального применения
unattended-upgrades --dry-run --debug

# Логи
tail -n 50 /var/log/unattended-upgrades/unattended-upgrades.log
```

---

## Следующий шаг
→ [09-monitoring.md](09-monitoring.md)
