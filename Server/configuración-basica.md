# Conceptos básicos de configuración de un servidor

Esta guía es una referencia dee como configurar uns servidor para alojar aplicaciones.

### Configuracióno básica de inicio de sesión con SSH

Lo primero es iniciar sesión. Necesitamos acceder al dispositivo de manera segura.  
Para hacerlo necesitamos una ***key***(clave) SSH y una cuenta nueva de usuario. En primer lugar en el servidor remoto creamos un nuevo usuario y lo agregaremos al grupo "sudo"

```sh
$ sudo adduser nombre_usuario
$ sudo usermod -aG sudo nombre_usuario
```

Ahora en nuestro equipo (*importante!!* en nuestro equipo **no** en el servidor) corremos el siguiente comando en la términal

```bash
$ ssh-keygen -t ed25519 -C "mi_correo@porejemplo.com"
```

Seguimos las instrucciones, nos preguntará si queremos contraseña, nos aseguramos de establecer una. Ahora copiamos la clave publica en el servidor introduciendo el siguiente comando.

```bash
$ ssh-copy-id -i ~/.ssh/id_ed25519.pub nombre_usuario@ip_del_servidor
```

Tener en cuenta que *nombre_usuario@ip_del_servidor* es el nombre de usuario que se creó en el servidor y la dirección ip o dns del servidor.  

Cuando nos pida la contraseña, será la contraseña del usuario creado en el servidor, no la contraseña que acabamos de crear para la clave SSH.  

Una vez verificado, copiará la clave publica en el servidor y ahora podremos iniciar sesión mediante SSH.  

Si queremos no tener que iniciar sesión con usuario y contraseña, bastará con introducir el siguiente comando en la términal y establecer algunos parametros:

```bash
$ sudo nano /eetc/ssh/sshd_config
```

```
Port 2222                                                         # Cambiar el puerto por defecto (usa un número entre 1024 y 65535)
PermitRootLogin no                                       # Deshabilitar root login
PasswordAuthentication no                            # Deshabilitar password authentication
PubkeyAuthentication yes                              # Habilitar clave publica para autenticación
AuthorizedKeysFile .ssh/authorized_keys     # Especifica la ubicación del archivo authorized_keys
AllowUsers nombre_usuario                          # Solo permite que usuarios especiicos inicien sesión
```

Presionamos Ctrl+S para guardar los cambios y resetaamos el servicio.

```bash
$ sudo systemctl restart ssh.service
```

Lo más probable es que no eche fuera de la sesión una vez reiniciado el servicio, es un buen momento para intentar iniciar sesión por otros métodos y así asegurarnos de que cualquier otra manera de iniciar sesión es rechazada.

Si perdemos nuestra clave privada, ya no podremos acceder al servidor de manera remota. Podemos asegurar mucho más el inicio de sesión con:

```
Protocol 2                             # Usar solamente protocolo SSH version 2
MaxAuthTries 3                   # Limitar los intentos de acceso
ClientAliveInterval 300       # Intervalo en segundos que el cliente tiene acceso
ClientAliveCountMax 2      # Número máximo de clientes activos
```

### Usuarios

Para la gestión de un servidor, hay una idea que se llama "Principio del minimo privilegio", basicamente viene a decir que a una aplicación en concreto le queremos dar los privilegios que necesita para realizar su trabajo. Esto es beneficioso ya que limitamos los daños potenciales si la aplicación que estemos ejecutando esta comprometida. Añade aislamiento cuando se ejecuta más de una aplicación y ayuda con la auditoria para saber que aplicación utiliza que recursos del sistema.

En resumen, los usuarios ayudan a organizar el sistema y a resolver problemas cuando las cosas van mal.

Para añadir nuevos usuarios usaremos el siguiente comando:

```bash
$ sudo useradd -rms /usr/sbin/nologin -c "un comentario" usuario_uno
```

Se añade un nuevo usuario y se le asigna un directorio de inicio para los datos de la aplicación, pero no permite al usuario iniciar sesión. El flag -c es opcional, pero es bueno saber que es lo que hace este usuario, por ejemplo podria ejecutar Nextcloud.

Ahora clonaremos los archivos de la aplicación en el directorio ***/opt***

```bash
$ sudo mkdir /opt/mi_aplicacion
$ sudo chown usuario_uno:usuario_uno /opt/mi_aplicacion
```

### Logs

Los registros  (Logs) son importantes para la administración de un sistema. Llevan un seguimiento de lo que ocurre en el sistema, ayudan a resolver problemas y detectan amenazas. 

Para configurar un log que no ocupe mucho espacio y además sean fáciles de leer y gestionar, modificaremos el archivo *logrotate.conf* ubicado en el directorio /etc.  
Las configuraciones de las aplicaciones normalmente se almacenan en /etc/logrotate.d/, de manera que un ejemplo para NGINX seria más o menos así:

```
/var/log/nginx/*.log {
    weekly
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        [ -f /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid`
    endscript
}
```

Esta configuración establece lo siguiente
- Los logs rotan cada semana
- Conserva los logs durante 52 semanas
- Comprime los logs antiguos
- Crea nuevos logs con los permisos adecuados
- Indica a NGINX que vuelva a abrir los archivos después de una rotación

Para probarlo, ejecutar el siguiente comando
```bash
$ sudo logrotate -d /etc/logrotate.conf
```

Con todo esto, ahora podemos configurar cosas más avanzadas, como activar alertas basadas en entradas, etc.  
Esto es bueno para un solo servidor, pero si hemos de gestionar más de un servidor, es una buena idea usar herramientas como Grafana, Loki, Graylog o Fluentd.

### Backups

Hay tres tipos de copias de seguridad, 

1. Completa  
Es una copia completa de todos los archivos y carpetas. Utiliza muchos recursos de sistema pero es más fácil de restaurar.

2. Diferencial  
Estas hacen una copia de seguridad de todos los cambios desde la última copia de seguridad completa.

3. Incremental  
   Esta hace una coopia de seguridad de los datos que han cambiado desde la última copia de seguirdad (indiferente si fué completa, diferencial o incremental). Esta opción es la más rapida pero la peor a la hora de tener que restaurar.

Las copias de seguridad incrementales pueden ser una buena idea para fotos, documentos, carpetas de proyecto que se modifican mucho, etc.  

Las copias de seguridad completas son para cuando se quiere copiar todo un servidor o disco.

Las copias de seguridad diferenciales **no** se usarán para hacer copias de carpetas enteras como /etc, /opt ni carpetas de los logs.

Existe la regla 3-2-1, que significa que se han de hacer 3 copias de los datos, se ha de tener 2 tipos de almacenaje y copia de seguridad externa (este es el más importante y no se debe saltar)  
Existen una grán cantidad de recursos para las copias de seguridad.
[Recursos](https://github.com/awesome-foss/awesome-sysadmin#backups)

### Seguridad básica de la red

Hay que asegurar que el servidor bloquea los puertos que no deben estar expuestos a internet y evitar que intenten hacer un inicio de sesión cuando no deberian hacerlo.

Hay dos herramientas (UFW y Fail2Ban) que se utilizan mucho para tales cometidos, son fáciles de usar y sencillas.  
Por un lado ***UFW*** permite establecer reglas de tráfico por los puertos, y ***Fail2Ban*** banea direcciones IP que intenten hacer una llamada a un puerto donde no este permitido o iniciar sesión dependiendo de unas reglas predefinidas.  

Una vez tengamos estas herramientas instaladas, procederemos a establecer unas reglas o politicas por defecto:

```bash
sudo ufw default deny incoming
sudo ufw allow outgoing
```

Esto se considera una buena practica, ya que sigue la regla que mencionamos al principio de "los mínimos privilegios". Reduce considerablemente los ataques y ofrece un control preciso sobre lo que se expone.  

En resumen, esta configuración crea un equilibrio entre funcionalidad y seguridad.  

Nuestro servidor se puede conectar a internet cuando sea necesario, pero cualquier entidad que quiera conectar con nuestro servidor solo podrán de las maneras que hayamos permitido explicitamente.

```bash
$ sudo ufw allow ssh
$ sudo ufw allow 80
$ sudo ufw allow 443
```

Si estamos ejecutando un servidor web, necesitaremos los puertos 80, 443 y 22 abiertos.

Otros paramatros para configurar:
```bash
#Enumera una lista de reglas con numeros:
sudo ufw status numbered
#Elimina por numero una regla:
sudo ufw delete NUMBER
#Elimina una regla especifica:
sudo ufw delete allow 80
#Permite conexiones desde una dirección IP especifica:
sudo ufw allow from 192.168.1.100
#Permite conexiones desde una dirección IP a un puerto especifico: 
sudo ufw allow from 192.168.1.100 to any port 22
#Permite entre un rango de puertos: 
sudo ufw allow 6000:6007/tcp
#Protege ante ataques de fuerza bruta, se puede limitar a un puerto especifico: 
sudo ufw limit ssh 

#Obtenemos mas información
sudo ufw status verbose
#Reset, en caso de que tengamos que volver a empezar de nuevo: 
sudo ufw reset
#Habilitar o deshabilitar: 
sudo ufw enable 
sudo ufw disable 

#Habilita el registro de logs y ajuste del nivel: 
sudo ufw logging on
#Los niveles son: low, medium, high, full
sudo ufw logging medium  
```


### Fail2Ban

El archivo de configuración está ubicaddo en ***/etc/fail2ban/jail.conf***, pero lo recomendado es que crees un archivo de configuración con extension *.local

```bash
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
$ sudo nano /etc/fail2ban/jail.local
```

Los parametros por defecto en la sección [DEFAULT] son los siguientes

```bash
bantime = 10m       #El tiempo que una IP ha de estar baneada
findtime = 10m      #Es el periodo de tiempo en el Fail2Ban busca errores repetidos
maxretry = 5          #Máximo de errores permitidos antes de banear una IP
```


### NGINX

Configuración básica de un sitio estatico
```conf
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    root /var/www/example.com/html;
    index index.html index.htm;
    location / {
        try_files $uri $uri/ =404;
    }
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    # Logging
    access_log /var/log/nginx/example.com.access.log;
    error_log /var/log/nginx/example.com.error.log warn;

    # SSL configuration (uncomment after running Certbot)
    # listen 443 ssl http2;
    # listen [::]:443 ssl http2;
    # ssl_protocols TLSv1.2 TLSv1.3;
    # ssl_prefer_server_ciphers on;
    # ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;

    # Certbot will add its own SSL certificate paths
    # ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    # ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
}
```

Configuración Proxy
```
server {
    listen 80;
    listen [::]:80;
    server_name app.example.com;
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    # Logging
    access_log /var/log/nginx/app.example.com.access.log;
    error_log /var/log/nginx/app.example.com.error.log warn;

    # SSL configuration (uncomment after running Certbot)
    # listen 443 ssl http2;
    # listen [::]:443 ssl http2;
    # ssl_protocols TLSv1.2 TLSv1.3;
    # ssl_prefer_server_ciphers on;
    # ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;

    # Certbot will add its own SSL certificate paths
    # ssl_certificate /etc/letsencrypt/live/app.example.com/fullchain.pem;
    # ssl_certificate_key /etc/letsencrypt/live/app.example.com/privkey.pem;
}
```

Configuración WebSockets
```
server {
    listen 80;
    listen [::]:80;
    server_name ws.example.com;
    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    # WebSocket timeout settings
    proxy_read_timeout 300s;
    proxy_send_timeout 300s;
    # Logging
    access_log /var/log/nginx/ws.example.com.access.log;
    error_log /var/log/nginx/ws.example.com.error.log warn;

    # SSL configuration (uncomment after running Certbot)
    # listen 443 ssl http2;
    # listen [::]:443 ssl http2;
    # ssl_protocols TLSv1.2 TLSv1.3;
    # ssl_prefer_server_ciphers on;
    # ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;

    # Certbot will add its own SSL certificate paths
    # ssl_certificate /etc/letsencrypt/live/ws.example.com/fullchain.pem;
    # ssl_certificate_key /etc/letsencrypt/live/ws.example.com/privkey.pem;
}
```

***La configuración básica***, es para servir sitios estaticos.  
Especifica un nombre de dominio que escucha el puerto 80 tanto para IPv4 como para IPv6, configura el manejo de errores, añade cabeceras basicas que protegen de vulnerabilidades habituales, configura el registro de logs para los accesos y errores y se incluye una sección de SSL que se comenta.  
La mayor parte de configuración para SSL se encarga cerbot.

***La configuración proxy***, es similara a la básica pero en lugar de servir archivos estaticos, envia solicitudes a una aplicación local, en este caso se ejecuta en el puerto 3000.

***La configuración websokets***, está orientada a aplicaiones que necesiten comunicación bidireccional, es parecida a la configuración de proxy.

### SSL

Cebot es el mejor amigo para gestionar SSL, además de ser gratuito, rápido y funciona. Se puede instalar con la versión de python:

```bash
$ sudo pacman -S cerbot python3-cerbot-nginx
```

Una vez instalado, simplemente desde la terminal introducimos cerbot, sutomáticamente ya detecta la carpeta de sites-enabled y preguntará que hacer (renovar, reeditar, etc.)  

Al obtener un nuevo certificado, cerbot configurará de forma automática la renovación, es una tarea de la qeu nos podemos olvidar, pero si queremos asegurarnos basta con introducir en la términal lo siguiente:

```bash
$ sudo systemctl status cerbot.timer
```