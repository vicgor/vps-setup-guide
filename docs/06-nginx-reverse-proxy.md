# 06 — Nginx Reverse Proxy

## Цель
Настроить Nginx как reverse proxy для Python/Docker-сервисов с поддержкой HTTPS.

---

## 1. Установка Nginx

```bash
apt install nginx -y
systemctl enable --now nginx
```

## 2. Структура конфигов

```
/etc/nginx/
├── nginx.conf          # главный конфиг
├── conf.d/
│   └── default.conf
└── sites-enabled/
    ├── myapp.conf
    └── mybot.conf
```

## 3. Базовый reverse proxy (HTTP)

```nginx
# /etc/nginx/sites-enabled/myapp.conf
server {
    listen 80;
    server_name myapp.example.com;

    location / {
        proxy_pass         http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header   Host              $host;
        proxy_set_header   X-Real-IP         $remote_addr;
        proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_read_timeout 90;
    }
}
```

## 4. HTTPS через Certbot (Let's Encrypt)

```bash
apt install certbot python3-certbot-nginx -y
certbot --nginx -d myapp.example.com

# Автообновление сертификата
certbot renew --dry-run
```

## 5. Security headers

```nginx
# Добавить в server{} блок или /etc/nginx/conf.d/security.conf
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

## 6. Rate limiting (защита от DDoS)

```nginx
# В nginx.conf -> http{} блок
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

# В location{} блок
limit_req zone=api burst=20 nodelay;
```

## 7. Проверка конфига и перезагрузка

```bash
nginx -t
systemctl reload nginx
```

---

## Следующий шаг
→ [07-python-services.md](07-python-services.md)
