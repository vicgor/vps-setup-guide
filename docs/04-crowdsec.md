# 04 — CrowdSec

## Цель
Автоматически блокировать IP при атаках на SSH, Nginx и другие сервисы. CrowdSec — современная замена Fail2ban: анализирует логи через **агент**, блокирует трафик через **bouncer** (iptables/ipset), дополнительно использует облачный фид репутации IP-адресов.

---

## 1. Установка

```bash
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | bash
apt install crowdsec crowdsec-firewall-bouncer-iptables -y
```

Агент запустится автоматически. Базовые коллекции (SSH, Linux) устанавливаются при первом запуске.

## 2. Проверка статуса

```bash
systemctl status crowdsec
systemctl status crowdsec-firewall-bouncer

# Список активных коллекций (сценариев детектирования)
cscli collections list

# Список зарегистрированных bouncer'ов
cscli bouncers list
```

## 3. Коллекции

Коллекция — набор сценариев для определённого сервиса (аналог jail в Fail2ban).

```bash
# Установить коллекцию для Nginx
cscli collections install crowdsecurity/nginx

# Установить коллекцию для MySQL
cscli collections install crowdsecurity/mysql

# Обновить все коллекции
cscli hub update && cscli hub upgrade
```

После установки коллекции перезапустить агент:

```bash
systemctl restart crowdsec
```

## 4. Управление блокировками

```bash
# Список активных блокировок
cscli decisions list

# Заблокировать IP вручную
cscli decisions add --ip 1.2.3.4

# Разблокировать IP
cscli decisions delete --ip 1.2.3.4

# Заблокировать подсеть
cscli decisions add --range 1.2.3.0/24
```

## 5. Просмотр алертов и логов

```bash
# Последние алерты
cscli alerts list

# Логи агента
journalctl -u crowdsec -f

# Логи bouncer'а
journalctl -u crowdsec-firewall-bouncer -f
```

## 6. Whitelist (исключить свой IP)

Создать файл whitelist:

```bash
nano /etc/crowdsec/parsers/s02-enrich/mywhitelist.yaml
```

Содержимое:

```yaml
name: crowdsecurity/mywhitelist
description: "Whitelist own IPs"
whitelist:
  reason: "operator IP"
  ip:
    - "127.0.0.1"
    - "YOUR_IP"
```

```bash
systemctl restart crowdsec
```

---

## Следующий шаг
→ [05-docker-setup.md](05-docker-setup.md)
