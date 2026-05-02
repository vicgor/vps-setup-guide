# Улучшения безопасности сервера

Список задач по результатам аудита от 2026-05-02.

---

## 🔴 Высокий приоритет

### 1. Права доступа к конфигурации Xray

`/usr/local/etc/xray/config.json` имеет права `644` (читаем всеми пользователями системы). Файл содержит REALITY `privateKey` и UUID клиента.

```bash
chmod 600 /usr/local/etc/xray/config.json
```

В той же директории лежат три файла бэкапов со старыми ключами — удалить:

```bash
rm /usr/local/etc/xray/config.json.bak
rm /usr/local/etc/xray/config.json.bak2
rm /usr/local/etc/xray/config.json.bak-20260419
```

---

### 2. Netdata открыт на всех интерфейсах

`/etc/netdata/netdata.conf` не содержит настроек — используются дефолты, Netdata слушает на `0.0.0.0:19999` без авторизации.

```bash
nano /etc/netdata/netdata.conf
```

Добавить:

```ini
[web]
    bind to = 127.0.0.1
```

```bash
systemctl restart netdata
```

После этого Netdata доступен только через SSH-туннель:

```bash
# На локальной машине
ssh -L 19999:localhost:19999 deploy@YOUR_SERVER_IP -p 2222
```

> ⚠️ SSH-туннель требует `AllowTcpForwarding yes` в `/etc/ssh/sshd_config.d/hardening.conf`. Сейчас этот параметр выключен — нужно включить перед закрытием Netdata, иначе доступ к мониторингу будет потерян.

Порядок действий:
1. Включить `AllowTcpForwarding yes` в `hardening.conf` → `systemctl restart ssh`
2. Проверить туннель: `ssh -L 19999:localhost:19999 deploy@... -p 2222`
3. Открыть `http://localhost:19999` — убедиться что работает
4. Добавить `bind to = 127.0.0.1` в netdata.conf → `systemctl restart netdata`

---

## 🟡 Средний приоритет

### 3. Создать `/etc/docker/daemon.json`

Без этого файла логи контейнеров растут без ограничений.

```bash
nano /etc/docker/daemon.json
```

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "live-restore": true,
  "userland-proxy": false,
  "no-new-privileges": true
}
```

```bash
systemctl restart docker
```

> Флаг `live-restore` позволяет контейнерам продолжать работу при перезапуске демона Docker.

---

### 4. Почистить основной sshd_config

`/etc/ssh/sshd_config` содержит `PermitRootLogin yes` и `X11Forwarding yes`, которые перекрываются `hardening.conf`. Технически безопасно, но создаёт путаницу при аудите.

```bash
nano /etc/ssh/sshd_config
```

Удалить или закомментировать:
```text
# PermitRootLogin yes   ← убрать (задано в hardening.conf)
# X11Forwarding yes     ← убрать (задано в hardening.conf)
```

```bash
sshd -t && systemctl restart ssh
```

---

## ✅ Что уже настроено корректно

| Параметр | Статус |
|----------|--------|
| SSH: порт 2222, только ключи, только пользователь `deploy` | ✅ |
| CrowdSec + firewall bouncer | ✅ |
| UFW активен | ✅ |
| unattended-upgrades | ✅ |
| AppArmor | ✅ |
| sysctl: tcp_syncookies, rp_filter, no redirects | ✅ |
| Docker: все порты через `127.0.0.1` | ✅ |
| `.env` файлы: права `600` | ✅ |
| Xray: запуск от `nobody`, `NoNewPrivileges=true` | ✅ |
| Xray: блокировка приватных IP, BitTorrent | ✅ |
