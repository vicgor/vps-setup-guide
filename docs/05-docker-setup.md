# 05 — Docker + Docker Compose

## Цель
Установить Docker и Docker Compose с безопасными настройками.

---

## 1. Установка Docker (официальный способ)

```bash
# Зависимости
apt install -y ca-certificates curl gnupg

# GPG-ключ Docker
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

# Репозиторий
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
| tee /etc/apt/sources.list.d/docker.list > /dev/null

# Установка
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 2. Добавить пользователя в группу docker

```bash
usermod -aG docker myuser
# Перелогиниться для применения изменений
```

## 3. Проверка

```bash
docker --version
docker compose version
docker run hello-world
```

## 4. Безопасность Docker

```bash
# /etc/docker/daemon.json
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

## 5. Базовая структура проекта

```
/opt/projects/
├── my-bot/
│   ├── docker-compose.yml
│   ├── .env
│   └── app/
├── my-api/
│   ├── docker-compose.yml
│   └── app/
```

## 6. Пример docker-compose.yml для Python-бота

```yaml
services:
  bot:
    build: ./app
    restart: unless-stopped
    env_file: .env
    networks:
      - internal
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

networks:
  internal:
    driver: bridge
```

---

## Следующий шаг
→ [06-nginx-reverse-proxy.md](06-nginx-reverse-proxy.md)
