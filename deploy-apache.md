
# 🛠️ Despliegue de una API Laravel + Frontend React en Ubuntu (WSL) con Dominios Simulados

Esta guía te permite desplegar una API desarrollada en Laravel y un frontend hecho en React en un entorno **Ubuntu dentro de WSL**, simulando dominios personalizados usando `hosts` locales y configurando Apache.

---

## ✅ Requisitos Previos

- Tener **WSL2 con Ubuntu** instalado.
- Tener los proyectos Laravel y React listos.
- Instalar Apache, PHP, Composer, Node.js y npm.

---

## 🔧 1. Configuración del Entorno

```bash
sudo apt update && sudo apt upgrade
sudo apt install apache2 php php-mysql php-cli php-curl php-mbstring php-xml php-bcmath php-zip unzip curl git composer npm
```

Verifica:

```bash
php -v
composer -V
node -v
npm -v
```

---

## 🌐 2. Desplegar API Laravel en `api.local`

### 2.1 Clonar y configurar Laravel

```bash
cd /var/www/
sudo git clone https://github.com/tu-usuario/tu-api-laravel.git laravel-api
cd laravel-api
composer install
cp .env.example .env
php artisan key:generate
```

### 2.2 Configurar Apache

```bash
sudo nano /etc/apache2/sites-available/api.local.conf
```

Contenido:

```apache
<VirtualHost *:80>
    ServerName api.local
    DocumentRoot /var/www/laravel-api/public

    <Directory /var/www/laravel-api/public>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/api_error.log
    CustomLog ${APACHE_LOG_DIR}/api_access.log combined
</VirtualHost>
```

Habilita el sitio:

```bash
sudo a2ensite api.local.conf
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## ⚛️ 3. Desplegar Frontend React en `frontend.local`

### 3.1 Crear build de producción

```bash
cd /home/usuario/mi-react-app
npm install
npm run build
```

### 3.2 Mover el build a carpeta pública

```bash
sudo mkdir /var/www/react-frontend
sudo cp -r build/* /var/www/react-frontend/
```

### 3.3 Configurar Apache

```bash
sudo nano /etc/apache2/sites-available/frontend.local.conf
```

Contenido:

```apache
<VirtualHost *:80>
    ServerName frontend.local
    DocumentRoot /var/www/react-frontend

    <Directory /var/www/react-frontend>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/frontend_error.log
    CustomLog ${APACHE_LOG_DIR}/frontend_access.log combined
</VirtualHost>
```

Activa y recarga:

```bash
sudo a2ensite frontend.local.conf
sudo systemctl reload apache2
```

---

## 🧭 4. Configurar `/etc/hosts`

```bash
sudo nano /etc/hosts
```

Agrega al final:

```
127.0.0.1 api.local
127.0.0.1 frontend.local
```

---

## 🧪 5. Probar en el Navegador

- [http://api.local](http://api.local) → Debe mostrar Laravel.
- [http://frontend.local](http://frontend.local) → Debe mostrar el frontend React.

---

## 🔗 6. Conectar React con la API Laravel

En `.env` de React:

```env
REACT_APP_API_URL=http://api.local/api
```

Uso en código:

```js
fetch(`${process.env.REACT_APP_API_URL}/usuarios`)
  .then(response => response.json())
  .then(data => console.log(data));
```

---

## 🎉 ¡Listo!

Ya tienes una API y un frontend corriendo en WSL como si fuera producción, con dominios simulados usando `hosts`.

