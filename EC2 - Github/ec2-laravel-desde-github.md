# **Configuración de Laravel en AWS EC2 con Amazon Linux 2023 y Apache, clonado desde GitHub**

Esta guía describe los pasos para desplegar Laravel en una instancia EC2 con Amazon Linux 2023 y Apache, utilizando un certificado SSL autofirmado.

## **1. Conectarse a la instancia EC2**

Puedes conectarte a tu instancia EC2 de dos formas:

### **Opción 1: Desde SSH**
```sh
ssh -i "tu-llave.pem" ec2-user@tu-ip-publica
```

### **Opción 2: Desde el panel de AWS**
1. Ve a la consola de AWS EC2.
2. Selecciona tu instancia en ejecución.
3. Haz clic en **Conectar**.
4. Usa la opción **Session Manager** para conectarte sin necesidad de claves SSH.

## **2. Actualizar paquetes y preparar el entorno**
```sh
sudo dnf update -y
sudo dnf install -y httpd git unzip php-cli php-mbstring php-xml php-curl php-zip php-pdo php-fpm php-mysqlnd php-gd
```

## **3. Habilitar y arrancar Apache**
```sh
sudo systemctl enable --now httpd
sudo systemctl status httpd
```

## **4. Clonar el repositorio Laravel**
```sh
cd /var/www
sudo git clone tu-link-de-github laravel
```

## **5. Configurar permisos**
```sh
sudo chown -R ec2-user:apache /var/www/laravel
sudo chmod -R 775 /var/www/laravel
sudo chmod -R 775 /var/www/laravel/storage /var/www/laravel/bootstrap/cache
```

## **6. Instalar Composer**
```sh
cd /var/www/laravel
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

Verificar la instalación:
```sh
composer --version
```

## **7. Configurar Apache**
```sh
sudo nano /etc/httpd/conf.d/laravel.conf
```

Añadir:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot "/var/www/laravel/public"
    <Directory "/var/www/laravel/public">
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog "/var/log/httpd/laravel-error.log"
    CustomLog "/var/log/httpd/laravel-access.log" common
</VirtualHost>
```

Aplicar los cambios:
```sh
sudo systemctl restart httpd
```

## **8. Configurar SSL autofirmado**
```sh
sudo dnf install -y mod_ssl
sudo mkdir /etc/httpd/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/httpd/ssl/laravel.key -out /etc/httpd/ssl/laravel.crt
```

Editar:
```sh
sudo nano /etc/httpd/conf.d/laravel-ssl.conf
```

Añadir:
```apache
<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    DocumentRoot "/var/www/laravel/public"
    <Directory "/var/www/laravel/public">
        AllowOverride All
        Require all granted
    </Directory>
    SSLEngine on
    SSLCertificateFile /etc/httpd/ssl/laravel.crt
    SSLCertificateKeyFile /etc/httpd/ssl/laravel.key
    ErrorLog "/var/log/httpd/laravel-error.log"
    CustomLog "/var/log/httpd/laravel-access.log" common
</VirtualHost>
```

Reiniciar Apache:
```sh
sudo systemctl restart httpd
```

## **9. Instalar las dependencias de Laravel**
```sh
cd /var/www/laravel
composer install
sudo cp .env.example .env
sudo php artisan key:generate
```

Asegurar permisos:
```sh
sudo chown -R apache:apache /var/www/laravel
sudo chmod -R 775 /var/www/laravel/storage /var/www/laravel/bootstrap/cache
```

## **10. Acceder a la aplicación**
- HTTP: 🔗 `http://tu-ip-publica`
- HTTPS: 🔗 `https://tu-ip-publica` *(ignora la advertencia del certificado autofirmado)*

Esta guia esta inspirada por "richie20502"
