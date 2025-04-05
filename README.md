# Clonar-NodeJS-Express-EC2
Repositorio creado para clonar una API de Node.js con Express desde un repositorio de GitHub e implementarlo en una instancia EC2 de AWS, utilizando PM2 y Nginx.  

## Justificación del uso de PM2 y Nginx
 PM2: Es un gestor de procesos para Node.js que facilita la ejecución de aplicaciones en segundo plano,
     permitiendo el reinicio automático, balanceo de carga y monitoreo del rendimiento.
 Nginx: Se usa como proxy inverso para mejorar el rendimiento y seguridad de la API,
     manejando tráfico, peticiones HTTPS y distribuyendo las solicitudes de manera eficiente.

---

## 1. Actualizar paquetes en Amazon Linux 2023
sudo dnf update -y

## 2. Instalar Node.js y npm
sudo dnf install -y nodejs npm

## 3. Instalar Git
sudo dnf install -y git

## 4. Crear la carpeta del proyecto si no existe
sudo mkdir -p /var/www/api

## 5. Clonar el repositorio desde GitHub
cd /var/www/api
sudo git clone https://github.com/usuario/repositorio-nodejs.git .

## 6. Instalar dependencias de Node.js
sudo npm install

---

## 7. Instalar y configurar PM2 para gestionar la API de Node.js
# Instalar PM2 globalmente
sudo npm install -g pm2

# Arrancar la API con PM2
pm2 start index.js --name "api-node"

# Configurar PM2 para iniciar automáticamente en el arranque del sistema
pm2 startup

# Guardar la configuración de PM2
pm2 save

---

## 8. Instalar y configurar Nginx como proxy inverso
# Instalar Nginx
sudo dnf install -y nginx

# Crear el archivo de configuración si no existe
sudo mkdir -p /etc/nginx/conf.d
sudo nano /etc/nginx/conf.d/api.conf

# Agregar la siguiente configuración:
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

# Reiniciar Nginx para aplicar los cambios
sudo systemctl restart nginx

---

## 9. Configurar un certificado SSL autofirmado para HTTPS

# 1. Instalar mod_ssl en Nginx
sudo dnf install -y mod_ssl

# 2. Crear la carpeta de certificados si no existe
sudo mkdir -p /etc/pki/tls/certs
sudo mkdir -p /etc/pki/tls/private

# 3. Generar el certificado autofirmado y la clave privada
sudo openssl req -new -x509 -days 365 -nodes -out /etc/pki/tls/certs/api-cert.pem -keyout /etc/pki/tls/private/api-key.pem

# Durante la generación del certificado, se te pedirá llenar un formulario con los siguientes campos:
 - Country Name (2 letter code) [XX]: MX   # Código del país, por ejemplo MX para México.
 - State or Province Name (full name) []: Estado de México   # Escribe el estado donde se encuentra el servidor.
 - Locality Name (eg, city) []: Xonacatlán   # La ciudad donde se hospeda el servidor.
 - Organization Name (eg, company) []: MiEmpresa   # Nombre de tu empresa o sitio.
 - Organizational Unit Name (eg, section) []: IT   # Área de tu organización, puede ser "IT" o "Desarrollo".
 - Common Name (e.g. server FQDN or YOUR name) []: tu-dominio.com   # Escribe el dominio o la IP pública del servidor.
 - Email Address []: admin@tu-dominio.com   # Correo de contacto.

# Puedes dejar en blanco los campos opcionales y presionar "Enter" para continuar.

# 4. Configurar Nginx para usar el certificado
sudo nano /etc/nginx/conf.d/api-ssl.conf

# Si el archivo no existe, créalo con:
sudo touch /etc/nginx/conf.d/api-ssl.conf

# Agregar la siguiente configuración:
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

# 5. Reiniciar Nginx para aplicar cambios
sudo systemctl restart nginx

---

## Acceder a la API
# Accede desde el navegador o prueba con cURL:
curl -k https://tu-dominio.com

