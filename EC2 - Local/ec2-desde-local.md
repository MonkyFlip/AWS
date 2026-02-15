
---

# **Despliegue de proyecto PHP nativo en AWS EC2 con Amazon Linux 2023 y Apache, subido desde PC local**

Esta guía describe los pasos para subir y configurar un proyecto PHP nativo en una instancia EC2 con Amazon Linux 2023 y Apache, utilizando un certificado SSL autofirmado.

---

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

---

## **2. Actualizar paquetes y preparar el entorno**
```sh
sudo dnf update -y
sudo dnf install -y httpd unzip php-cli php-mbstring php-xml php-curl php-zip php-pdo php-fpm php-mysqlnd php-gd
```

---

## **3. Habilitar y arrancar Apache**
```sh
sudo systemctl enable --now httpd
sudo systemctl status httpd
```

---

## **4. Subir el proyecto PHP desde tu PC local**

Desde tu terminal local, usa `scp` para transferir el proyecto comprimido:

```sh
scp -i "tu-llave.pem" proyecto.zip ec2-user@tu-ip-publica:/home/ec2-user/
```

Luego, en la instancia:

```sh
cd /var/www
sudo unzip /home/ec2-user/proyecto.zip -d php-nativo
```

---

## **5. Configurar permisos**
```sh
sudo chown -R ec2-user:apache /var/www/php-nativo
sudo chmod -R 775 /var/www/php-nativo
```

---

## **6. Configurar Apache**
```sh
sudo nano /etc/httpd/conf.d/php-nativo.conf
```

Añadir:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot "/var/www/php-nativo"
    <Directory "/var/www/php-nativo">
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog "/var/log/httpd/php-nativo-error.log"
    CustomLog "/var/log/httpd/php-nativo-access.log" common
</VirtualHost>
```

Aplicar los cambios:
```sh
sudo systemctl restart httpd
```

---

## **7. Configurar SSL autofirmado**
```sh
sudo dnf install -y mod_ssl
sudo mkdir /etc/httpd/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/httpd/ssl/php-nativo.key -out /etc/httpd/ssl/php-nativo.crt
```

Editar:
```sh
sudo nano /etc/httpd/conf.d/php-nativo-ssl.conf
```

Añadir:
```apache
<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    DocumentRoot "/var/www/php-nativo"
    <Directory "/var/www/php-nativo">
        AllowOverride All
        Require all granted
    </Directory>
    SSLEngine on
    SSLCertificateFile /etc/httpd/ssl/php-nativo.crt
    SSLCertificateKeyFile /etc/httpd/ssl/php-nativo.key
    ErrorLog "/var/log/httpd/php-nativo-error.log"
    CustomLog "/var/log/httpd/php-nativo-access.log" common
</VirtualHost>
```

Reiniciar Apache:
```sh
sudo systemctl restart httpd
```

---

## **8. Acceder a la aplicación**
- HTTP: 🔗 `http://tu-ip-publica`
- HTTPS: 🔗 `https://tu-ip-publica` *(ignora la advertencia del certificado autofirmado)*

---
