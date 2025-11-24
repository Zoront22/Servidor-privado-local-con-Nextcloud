# 🖥️ Guía: Crear tu Propio Servidor Nextcloud

![Nextcloud](https://img.shields.io/badge/Nextcloud-Servidor%20Personal-blue)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04%20LTS-orange)
![PHP](https://img.shields.io/badge/PHP-8.3-purple)
![License](https://img.shields.io/badge/License-MIT-green)

## 🚀 Introducción

Esta guía documenta el proceso completo de creación de un servidor personal Nextcloud. Nextcloud es una suite de software de código abierto que te permite almacenar archivos, contactos, calendarios y más en tu propio servidor, dándote control total sobre tus datos. Esta idea surgio al querer darle una nueva vida a una laptop hp que tenia desde hace años.

**¿Por qué hacer esto?**
- ✅ **Control total** sobre tus datos
- ✅ **Eliminar dependencia** de servicios en la nube comerciales
- ✅ **Revivir hardware antiguo** dándole nueva vida útil
- ✅ **Aprender administración** de sistemas y redes

## 💻 Especificaciones de mi Equipo

| Componente | Especificaciones |
|------------|------------------|
| **Equipo** | Laptop HP 240 G4 Notebook PC |
| **Procesador** | Intel Dual Core |
| **RAM** | 4 GB DDR3 |
| **Almacenamiento** | SSD 500 GB |
| **Sistema Operativo** | Ubuntu Server 24.04 LTS (o cualquier otro SO de linux) |
| **Conexión de Red** | Ethernet/WiFi integrado |

## ⚙️ Requisitos del Sistema

### Mínimos Recomendados
- **Procesador:** Dual Core (64-bit)
- **RAM:** 1 GB mínimo, 2 GB recomendado
- **Almacenamiento:** 40 GB (depende del uso)
- **Sistema Operativo:** Ubuntu Server 22.04/24.04 LTS (Cualquier SO de linux)

### Requisitos de Software (las versiones dependeran de la version del SO que instales en tu equipo)
- Apache 2.4+
- MariaDB 10.6+ o MySQL 8.0+
- PHP 8.1+ (recomendado PHP 8.3)
- Nextcloud 25+

## 🔧 Instalación del Stack LAMP

### 1. Actualizar el Sistema
```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

### 2. Instalar Apache
```bash
sudo apt install apache2 -y
```

### 3. Instalar PHP y otras dependencias
```bash
apt install php php-common libapache2-mod-php php-bz2 php-gd php-mysql \
php-curl php-mbstring php-imagick php-zip php-common php-curl php-xml \
php-json php-bcmath php-xml php-intl php-gmp zip unzip wget -y
```

### 4. Habilitar modulos apache
```bash
a2enmod env rewrite dir mime headers setenvif ssl
sudo systemctl restart apache2
sudo systemctl enable apache2
```

### 5. Verificar que apache este en ejecucion
```bash 
root@nc:~# systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/apache2.service; 
     enabled; preset: enabled)
     Active: active (running) since Tue 2024-06-18 04:39:09 UTC; 8s ago
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 16319 (apache2)
      Tasks: 6 (limit: 2269)
     Memory: 16.1M (peak: 16.3M)
        CPU: 97ms
     CGroup: /system.slice/apache2.service
             ├─16319 /usr/sbin/apache2 -k start
             ├─16326 /usr/sbin/apache2 -k start
```

### 6. Verificar los modulos apache en ejecucion
```bash
root@nc:~# apache2ctl -M
Loaded Modules:
 core_module (static)
 so_module (static)
 watchdog_module (static)
 http_module (static)
 log_config_module (static)
............
```

### 7. Instalar MariaDB
```bash
sudo apt install mariadb-server mariadb-client -y
sudo systemctl enable mysql
sudo systemctl start mysql
```

Ajustar estos parámetros:
```bash
sudo nano /etc/php/8.3/apache2/php.ini
```
```ini
memory_limit = 512M
upload_max_filesize = 2G
post_max_size = 2G
max_execution_time = 300
date.timezone = America/Mexico_City
```

Reiniciar apache
```bash
sudo systemctl restart apache2
```

## 🗃️ Configuración de MariaDB

### 1. Iniciar sesión en MariaDB.
```bash
sudo mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 1131
Server version: 10.11.7-MariaDB-2ubuntu2 Ubuntu 24.04
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]>
```

### 2. Crear Base de datos y Usuario para Nextcloud y Proporcionar Permisos de Usuario.
```sql
-- Crear usuario
CREATE USER 'ncloud'@'localhost' IDENTIFIED BY 'Sh@do5!d';

-- Crear base de datos
CREATE DATABASE ncloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

-- Otorgar privilegios
GRANT ALL PRIVILEGES ON ncloud.* TO 'ncloud'@'localhost';
FLUSH PRIVILEGES;
quit;
```

Reiniciar y habilitar MariaDB:
```bash
systemctl restart mariadb
systemctl enable mariadb
```

### 2. Verificar el estado de MariaDB.
```bash
root@nc:~# systemctl status mariadb
● mariadb.service - MariaDB 10.11.7 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; 
     enabled; preset: enabled)
     Active: active (running) since Tue 2024-06-18 04:43:21 UTC; 10s ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
   Main PID: 17589 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 12 (limit: 14975)
     Memory: 78.8M (peak: 81.8M)
        CPU: 573ms
     CGroup: /system.slice/mariadb.service
             └─17589 /usr/sbin/mariadbd
```

## 📊 Tabla de Compatibilidad

### Nextcloud vs PHP

| Versión Nextcloud | PHP Requerido | Estado | Recomendación |
|-------------------|---------------|---------|---------------|
| Nextcloud 28+ | PHP 8.1+ | ✅ Soporte activo | **Recomendado** |
| Nextcloud 27 | PHP 8.1+ | ✅ Soporte activo | Recomendado |
| Nextcloud 26 | PHP 8.0+ | ✅ Soporte activo | Estable |
| Nextcloud 25 | PHP 7.4+ | ⚠️ Fin de soporte | No recomendado |
| Nextcloud 24 | PHP 7.4+ | ❌ Sin soporte | Evitar |

### Hardware vs Rendimiento

| Configuración | Usuarios Simultáneos | Rendimiento |
|---------------|---------------------|-------------|
| 1 GB RAM + HDD | 1-2 usuarios | ⚠️ Básico |
| 2 GB RAM + SSD | 3-5 usuarios | ✅ Adecuado |
| 4 GB RAM + SSD | 5-10 usuarios | ✅ Bueno |
| 8+ GB RAM + SSD | 10+ usuarios | ✅ Excelente |

## 🌐 Instalación de Nextcloud

### 1. Descargar Nextcloud
Descargar y descomprimir en la carpeta **/var/www/html**
```bash
cd /var/www/html
wget https://download.nextcloud.com/server/releases/latest.zip
unzip latest.zip
```

Eliminar archivo .zip (opcional):
```bash
rm -rf latest.zip
```

Cambia la propiedad del directorio nextcloud al usuario HTTP:
```bash
chown -R www-data:www-data /var/www/html/nextcloud/
```

### 2. Instalar y configurar Nextcloud desde la linea de comandos.
Instalar Nextcloud:
```bash
cd /var/www/html/nextcloud
sudo -u www-data php occ  maintenance:install --database \
"mysql" --database-name "ncloud"  --database-user "ncloud" --database-pass \
'Sh@do5!d' --admin-user "admin" --admin-pass "password"
```

Permitir el acceso a Nextcloud utilizando una dirección IP o un nombre de dominio:
```bash
vi /var/www/html/nextcloud/config/config.php

  'trusted_domains' =>
  array (
    0 => 'localhost',
    1 => 'nc.mailserverguru.com',   // we Included the Sub Domain
  ),
  .....
:x
```

### 3. Configurar apache.
```bash
sudo nano /etc/apache2/sites-available/nextcloud.conf
```

Contenido del archivo:
```apache
<VirtualHost *:80>
    DocumentRoot /var/www/nextcloud
    ServerName tu-dominio-o-ip

    <Directory /var/www/nextcloud/>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/nextcloud_error.log
    CustomLog ${APACHE_LOG_DIR}/nextcloud_access.log combined
</VirtualHost>
```

Reiniciar apache:
```bash
systemctl restart apache2
```

### 4. Habilitar Módulos y Sitio
```bash
sudo a2enmod rewrite headers env dir mime
sudo a2ensite nextcloud.conf
sudo a2dissite 000-default.conf
sudo systemctl reload apache2
```

### 5. Completar Instalación vía Web
1. Abre navegador: `http://tu-ip-servidor`
2. Crea usuario administrador
3. Configura base de datos:
   - MySQL/MariaDB
   - Usuario: `nextcloud_user`
   - Contraseña: `tu_password_seguro`
   - Base de datos: `nextcloud`
   - Localhost: `localhost`

## 🔒 Seguridad del Servidor

### 1. Configurar Firewall (UFW)
```bash
sudo apt install ufw -y
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw enable
```

### 2. Instalar y Configurar Fail2ban
```bash
sudo apt install fail2ban -y

# Configurar para Nextcloud
sudo nano /etc/fail2ban/jail.d/nextcloud.conf
```

Contenido:
```ini
[nextcloud]
enabled = true
port = 80,443
protocol = tcp
filter = nextcloud
logpath = /var/www/nextcloud/data/nextcloud.log
maxretry = 3
bantime = 86400
findtime = 600
```

### 3. Configurar SSL con Let's Encrypt
```bash
sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache -d tu-dominio.com
```

### 4. Hardening de PHP
```bash
sudo nano /etc/php/8.3/apache2/php.ini
```
```ini
expose_php = Off
display_errors = Off
log_errors = On
```

### 5. Seguridad de Nextcloud
```bash
cd /var/www/nextcloud
sudo -u www-data php occ config:system:set trusted_domains 1 --value=tu-dominio.com
sudo -u www-data php occ config:system:set enforceforceSSL --type=boolean --value=true
```

## 🛠️ Mantenimiento

### Actualizaciones Regulares
```bash
# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Actualizar Nextcloud
cd /var/www/nextcloud
sudo -u www-data php occ upgrade
sudo -u www-data php occ maintenance:mode --off
```

### Backup Automático
Crear script `/usr/local/bin/backup-nextcloud.sh`:
```bash
#!/bin/bash
# Backup de base de datos
mysqldump -u nextcloud_user -p nextcloud > /backup/nextcloud_$(date +%Y%m%d).sql

# Backup de archivos
tar -czf /backup/nextcloud_data_$(date +%Y%m%d).tar.gz /var/www/nextcloud/data/

# Rotar backups antiguos (mantener 30 días)
find /backup/ -name "nextcloud_*" -mtime +30 -delete
```

### Monitoreo de Recursos
```bash
# Instalar herramientas de monitoreo
sudo apt install htop nethogs -y

# Ver uso de recursos
htop
df -h
free -h
```

## ⚖️ Ventajas y Desventajas

### ✅ Ventajas
| Ventaja | Descripción |
|---------|-------------|
| **Soberanía de datos** | Control total sobre tu información personal |
| **Privacidad** | Tus datos no son escaneados para publicidad |
| **Ahorro económico** | Elimina suscripciones a servicios en la nube |
| **Personalización** | Configuración según tus necesidades específicas |
| **Aprendizaje** | Experiencia invaluable en administración de sistemas |

### ❌ Desventajas
| Desventaja | Mitigación |
|------------|------------|
| **Mantenimiento constante** | Automatizar actualizaciones y backups |
| **Coste eléctrico** | Usar hardware eficiente energéticamente |
| **Responsabilidad de seguridad** | Implementar medidas de seguridad robustas |
| **Dependencia de conexión** | Considerar acceso offline con apps móviles |
| **Curva de aprendizaje** | Documentar procesos y aprender gradualmente |


### 📚 Recursos Adicionales
- [Documentación Oficial Nextcloud](https://docs.nextcloud.com/)
- [Comunidad Nextcloud](https://help.nextcloud.com/)
- [Guías de Seguridad Ubuntu](https://ubuntu.com/security)
