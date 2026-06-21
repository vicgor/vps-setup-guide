# 11 — VLESS + Xray (VLESS + Reality)

Рекомендуемый вариант — **VLESS + Reality**: маскирует трафик под обычный TLS, не требует доменного имени и сертификата.

---

## Фаза 1 — Подготовка VPS

**Шаг 1. Выбор VPS**

Возьмите VPS за пределами России. Хорошие варианты: Hetzner (Финляндия/Германия), DigitalOcean, Vultr. Достаточно минимального тарифа: 1 vCPU, 512 МБ RAM, Ubuntu 22.04 или Debian 12.

**Шаг 2. Подключение по SSH**

```bash
ssh root@YOUR_VPS_IP
```

**Шаг 3. Обновление системы**

```bash
apt update && apt upgrade -y
```

---

## Фаза 2 — Установка Xray

**Шаг 4. Установка через официальный скрипт**

```bash
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

Xray установится в `/usr/local/bin/xray`, конфиг — в `/usr/local/etc/xray/config.json`.

**Шаг 5. Генерация UUID пользователя**

```bash
xray uuid
```

Скопируйте и сохраните вывод — это ваш идентификатор подключения.

**Шаг 6. Генерация ключей Reality (X25519)**

```bash
xray x25519
```

Вывод будет таким:
```
Private key: aBcDeFgH...
Public key:  XyZaBcDe...
```

Сохраните оба ключа — они нужны в конфиге сервера и в настройках клиента.

---

## Фаза 3 — Конфигурация сервера

**Шаг 7. Создание конфига**

```bash
nano /usr/local/etc/xray/config.json
```

Вставьте следующее, заменив `YOUR_UUID` и `YOUR_PRIVATE_KEY` на ваши значения:

```json
{
  "log": { "loglevel": "warning" },
  "inbounds": [{
    "listen": "0.0.0.0",
    "port": 443,
    "protocol": "vless",
    "settings": {
      "clients": [{
        "id": "YOUR_UUID",
        "flow": "xtls-rprx-vision"
      }],
      "decryption": "none"
    },
    "streamSettings": {
      "network": "tcp",
      "security": "reality",
      "realitySettings": {
        "show": false,
        "dest": "www.microsoft.com:443",
        "xver": 0,
        "serverNames": ["www.microsoft.com"],
        "privateKey": "YOUR_PRIVATE_KEY",
        "shortIds": [""]
      }
    }
  }],
  "outbounds": [{"protocol": "freedom"}]
}
```

> **`dest`** — сайт-камуфляж. Можно заменить на `dl.google.com:443`, `addons.mozilla.org:443` и т.д. — любой крупный сайт с TLS 1.3.

**Шаг 8. Запуск Xray**

```bash
systemctl enable xray && systemctl start xray && systemctl status xray
```

Статус должен быть `active (running)`.

**Шаг 9. Открыть порт (если включён UFW)**

```bash
ufw allow 443/tcp && ufw reload
```

---

## Фаза 4 — Настройка клиента

**Шаг 10. Выбор клиента**

| Платформа | Клиент |
|---|---|
| Windows | Nekoray, v2rayN, Hiddify |
| macOS | Hiddify, Nekoray |
| Android | v2rayNG, Hiddify |
| iOS | Streisand, Shadowrocket |
| Linux | Nekoray, Hiddify |

**Шаг 11. Параметры подключения**

В клиенте введите:

| Параметр | Значение |
|---|---|
| Протокол | VLESS |
| Адрес | YOUR_VPS_IP |
| Порт | 443 |
| UUID | YOUR_UUID |
| Flow | xtls-rprx-vision |
| Security | Reality |
| SNI | www.microsoft.com |
| Public Key | YOUR_PUBLIC_KEY |
| Fingerprint | chrome |
| Short ID | (пусто) |

**Шаг 12. VLESS URI для QR-кода / импорта**

Большинство клиентов принимают ссылку напрямую:

```
vless://YOUR_UUID@YOUR_VPS_IP:443?type=tcp&security=reality&sni=www.microsoft.com&fp=chrome&pbk=YOUR_PUBLIC_KEY&flow=xtls-rprx-vision#MyVPN
```

---

## Полезные команды для диагностики

```bash
# Перезапустить Xray
systemctl restart xray

# Смотреть логи в реальном времени
journalctl -u xray -f

# Проверить синтаксис конфига
xray -test -config /usr/local/etc/xray/config.json

# Убедиться что порт слушается
ss -tlnp | grep 443
```

---

**Почему VLESS + Reality, а не обычный Shadowsocks?** Reality не требует доменного имени и сертификата, при этом трафик выглядит как обычное TLS-соединение с настоящим сайтом (microsoft.com / google.com) — DPI-системы не могут его отличить от легитимного.

---

## Предыдущий шаг
← [10-checklist.md](10-checklist.md)

