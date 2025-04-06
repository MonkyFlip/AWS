# **Clonar-NodeJS-Express-EC2**

Repositorio creado para clonar una API de Node.js con Express desde un repositorio de GitHub e implementarlo en una instancia EC2 de AWS, utilizando PM2 y Nginx.

## **Justificación del uso de PM2 y Nginx**
- **PM2:** Gestor de procesos para Node.js que facilita la ejecución en segundo plano, proporcionando reinicio automático, balanceo de carga y monitoreo del rendimiento.
- **Nginx:** Se usa como proxy inverso para mejorar el rendimiento y seguridad de la API, manejando tráfico, peticiones HTTPS y distribuyendo solicitudes de manera eficiente.

---

## **1. Actualizar paquetes en Amazon Linux 2023**
```sh
sudo dnf update -y
```

## **2. Instalar Node.js y npm**
```sh
sudo dnf install -y nodejs npm
```

## **3. Instalar Git**
```sh
sudo dnf install -y git
```

## **4. Crear la carpeta del proyecto si no existe**
```sh
sudo mkdir -p /var/www/api
```

## **5. Clonar el repositorio desde GitHub**
```sh
cd /var/www/api
sudo git clone tu-link-de-github .
```

## **6. Instalar dependencias de Node.js**
```sh
sudo npm install
```

---

## **7. Instalar y configurar PM2 para gestionar la API de Node.js**
```sh
# Instalar PM2 globalmente
sudo npm install -g pm2

# Arrancar la API con PM2
pm2 start index.js --name "api-node"

# Configurar PM2 para iniciar automáticamente en el arranque del sistema
pm2 startup

# Guardar la configuración de PM2
pm2 save
```

---

## **8. Instalar y configurar Nginx como proxy inverso**
```sh
# Instalar Nginx
sudo dnf install -y nginx

# Crear el archivo de configuración si no existe
sudo mkdir -p /etc/nginx/conf.d
sudo nano /etc/nginx/conf.d/api.conf
```

Añadir:
```nginx
server {
    listen 80;
    server_name tu-dominio.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```sh
# Reiniciar Nginx para aplicar los cambios
sudo systemctl restart nginx
```

---

## **9. Configurar un certificado SSL autofirmado para HTTPS**
```sh
# Instalar mod_ssl en Nginx
sudo dnf install -y mod_ssl

# Crear la carpeta de certificados si no existe
sudo mkdir -p /etc/pki/tls/certs
sudo mkdir -p /etc/pki/tls/private

# Generar el certificado autofirmado y la clave privada
sudo openssl req -new -x509 -days 365 -nodes -out /etc/pki/tls/certs/api-cert.pem -keyout /etc/pki/tls/private/api-key.pem
```

Durante la generación del certificado, se te pedirá llenar un formulario con los siguientes campos:
- **Country Name (2 letter code):** `MX` *(Ejemplo: México)*
- **State or Province Name:** `Estado de México`
- **Locality Name:** `Xonacatlán`
- **Organization Name:** `MiEmpresa`
- **Organizational Unit Name:** `IT`
- **Common Name:** `tu-dominio.com`
- **Email Address:** `admin@tu-dominio.com`

Puedes dejar en blanco los campos opcionales y presionar `Enter` para continuar.

```sh
# Configurar Nginx para usar el certificado
sudo nano /etc/nginx/conf.d/api-ssl.conf

# Si el archivo no existe, créalo con:
sudo touch /etc/nginx/conf.d/api-ssl.conf
```

Añadir:
```nginx
server {
    listen 443 ssl;
    server_name tu-dominio.com;

    ssl_certificate /etc/pki/tls/certs/api-cert.pem;
    ssl_certificate_key /etc/pki/tls/private/api-key.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```sh
# Reiniciar Nginx para aplicar cambios
sudo systemctl restart nginx
```

---

## **Acceder a la API**
```sh
# Accede desde el navegador o prueba con cURL:
curl -k https://tu-dominio.com
```
