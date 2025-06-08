
# 🐳 Docker + Laravel + React + Nginx: Configuración

Esta guía contiene la configuración completa para desplegar una API en Laravel y un frontend en React, usando Docker y Nginx como reverse proxy, **con montajes directos desde carpetas locales (bind mounts)** para asegurar que los archivos reales estén accesibles.

---

## 📦 Estructura del Proyecto

```
.
├── docker-compose.yml
├── frontend/
│   └── Dockerfile
├── backend/
│   └── Dockerfile
├── nginx/
│   └── conf.d/
│       └── default.conf
```

---

## 🐘 backend/Dockerfile – Laravel (PHP-FPM)

```dockerfile
FROM php:8.2-fpm

RUN apt-get update && apt-get install -y \
    git curl zip unzip libpq-dev libonig-dev libzip-dev \
    && docker-php-ext-install pdo pdo_mysql mbstring zip

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/api

COPY . .

RUN composer install --no-dev --optimize-autoloader

RUN chown -R www-data:www-data /var/www/api \
    && chmod -R 755 /var/www/api

EXPOSE 9000
CMD ["php-fpm"]
```

---

## ⚛️ frontend/Dockerfile – React (Build)

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

RUN npm run build

CMD ["echo", "Build complete."]
```

---

## 🌐 nginx/conf.d/default.conf – Reverse Proxy

```nginx
server {
    listen 80;
    server_name api.local;

    location / {
        root /var/www/api;
        index index.php index.html index.htm;
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass backend:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /var/www/api$fastcgi_script_name;
    }
}

server {
    listen 80;
    server_name frontend.local;

    location / {
        root /var/www/frontend;
        index index.html index.htm;
        try_files $uri /index.html;
    }
}
```

---

## 🧭 Hosts en Windows

Editar el archivo `C:\Windows\System32\drivers\etc\hosts` y agregar:

```
127.0.0.1 api.local
127.0.0.1 frontend.local
```

---

## 🐳 docker-compose.yml (Corregido con Bind Mounts)

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./frontend/build:/var/www/frontend:ro
      - ./backend/public:/var/www/api:ro
    depends_on:
      - frontend
      - backend
    networks:
      - appnet

  frontend:
    build:
      context: ./frontend
    volumes:
      - ./frontend/build:/app/build
    networks:
      - appnet
    command: ["npm", "run", "build"]

  backend:
    build:
      context: ./backend
    volumes:
      - ./backend:/var/www/api
    networks:
      - appnet
    environment:
      - DB_HOST=mysql
      - DB_DATABASE=laravel
      - DB_USERNAME=root
      - DB_PASSWORD=root
    depends_on:
      - mysql

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: laravel
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - appnet

volumes:
  db_data:

networks:
  appnet:
```

---

## ✅ Levantar el entorno

```bash
docker-compose up --build
```

---

## 🚀 Acceso en Navegador

- [http://api.local](http://api.local)
- [http://frontend.local](http://frontend.local)

---

## 🎉 ¡Listo!

Con esta configuración, tus contenedores Docker estarán sirviendo archivos reales desde tu proyecto local, con Nginx como proxy y dominios personalizados simulados.
