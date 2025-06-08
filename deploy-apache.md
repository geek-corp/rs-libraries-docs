
# 🛠️ Despliegue de una API Laravel + Frontend React en Ubuntu (WSL) con Dominios Simulados (PHP 8.3 + Node 22)

Esta guía te permite desplegar una API desarrollada en Laravel y un frontend hecho en React en un entorno **Ubuntu dentro de WSL**, simulando dominios personalizados usando `hosts` locales y configurando Apache.

---

## ✅ Requisitos Previos

- Tener **WSL2 con Ubuntu** instalado.
- Tener los proyectos Laravel y React listos.
- Instalar Apache, PHP 8.3, Composer, Node.js (v22) y npm.

---

## 🔧 1. Configuración del Entorno (PHP 8.3 y Node.js 22)

### 🔹 PHP 8.3

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

sudo apt install apache2 php8.3 php8.3-cli php8.3-common php8.3-curl php8.3-mbstring \
php8.3-mysql php8.3-xml php8.3-bcmath php8.3-zip libapache2-mod-php8.3 php8.3-intl unzip curl git composer
```

```bash
sudo a2enmod php8.3
sudo systemctl restart apache2
```

### 🔹 Node.js 22 (usando NVM)

```bash
# Instalar NVM (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Recargar perfil
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Instalar Node.js 22
nvm install 22
nvm use 22
```

Verifica versiones:

```bash
php -v
composer -V
node -v
npm -v
```

---
### 🔹 MySQL Server

```bash
sudo apt update
sudo apt install mysql-server
sudo systemctl start mysql
sudo systemctl enable mysql
```

Acceder como root:

```bash
sudo mysql
```

Crear base de datos y usuario:

```sql
CREATE DATABASE laravel CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'laraveluser'@'localhost' IDENTIFIED BY 'claveSegura';
GRANT ALL PRIVILEGES ON laravel.* TO 'laraveluser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Verifica:

```bash
mysql -u laraveluser -p
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

sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache

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

## 🧭 4. Configurar `/etc/hosts` si desean probar configuración solamente en LOCAL Windows
### en el gestor de archivos Ir a 
```bash
C:\Windows\System32\drivers\etc\hosts
```

Agrega al final:

```
172.26.xx.xx  api.local
172.26.xx.xx  frontend.local
```
(Sustituye xx.xx por lo que te dé wsl hostname -I)
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

Ya tienes una API y un frontend corriendo en WSL como si fuera producción, con PHP 8.3, Node.js 22 y dominios simulados usando `hosts`.
