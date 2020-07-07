# Instalacion de NGINX

1. Primero actualizar y descargar el paquete:

```
sudo apt update
sudo apt install nginx
```

2. Revisar el status con:

```BASH
sudo systemctl status nginx
```

3. Tiene que verse similar a esto:

```apache
nginx.service - A high performance web server and a reverse proxy server
Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
Active: active (running) since Sun 2018-04-29 06:43:26 UTC; 8s ago
Docs: man:nginx(8)
Process: 3091 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
Process: 3080 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
Main PID: 3095 (nginx)
Tasks: 2 (limit: 507)
CGroup: /system.slice/nginx.service
├─3095 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
└─3097 nginx: worker process
```

4. Configurar el firewall, asumiendo que se usa UFW, es necesario abrir los puertos 80 y 443:

```
sudo ufw allow 'Nginx Full'
```

5. Verificar el status con:

```
sudo ufw status
```

```
Status: active

To Action From

---

22/tcp ALLOW Anywhere
Nginx Full ALLOW Anywhere
22/tcp (v6) ALLOW Anywhere (v6)
Nginx Full (v6) ALLOW Anywhere (v6)
```

6. Una vez hecho esto, hay que revisar si el servicio levanto correctamente, se abre una ventana en el navegador y se ingresa la IP del server:

Ejemplo:

> http://10.10.0.45

## Como hacer manage al servicio NGINX:

Parar el servicio:

```
sudo systemctl stop nginx
```

Iniciar el servicio:

```
sudo systemctl start nginx
```

Reiniciar el servicio:

```
sudo systemctl restart nginx
```

Recargar el servicio (bueno en caso de que se modifique el nginx.conf):

```
sudo systemctl reload nginx
```

Activar o desactivar el servicio en boot:

```
sudo systemctl enable nginx
sudo systemctl disable nginx
```

## Configuracion del nginx.conf:

Todos los archivos de configuracion se guardan en /etc/nginx.

El archivo principal de configuracion es: /etc/nginx/nginx.conf.

nginx permite guardar bloques de configuracion separados para cada sitio o app, estos los guarda en: /etc/nginx/sites-available

Pero los archivos de configuracion de este folder son activos hasta que se linkean en: /etc/nginx/sites-enabled

Los logs de nginx se guardan en: /var/log/nginx

Uno puede guardar el domain root en cualquiera de los siguientes directorios:

- /home/<user_name>/<site_name>
- /var/www/<site_name>
- /var/www/html/<site_name>
- /opt/<site_name>

---

# Instalacion de Rocket.Chat

1. Instalar primero Node.js y otras dependencias necesarias:

```BASH
sudo apt install nodejs npm build-essential curl software-properties-common graphicsmagick
```

2. Para manejar de manera mas sencilla Node, se instala n:

```BASH
sudo npm install -g inherits n
```

3. Se instala la version recomendada de node para el software:

```BASH
sudo n 8.11.3

sudo npm install -g inherits n && sudo n 12.14.0
```

4. Este servicio utilizar MongoDB, entonces para instalarlo hay que agregar el repositorio de mongo a Ubuntu:

```BASH
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4

sudo add-apt-repository 'deb [arch=amd64] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse'
```

5. Cuando se agrega, se actualiza y se instala mongodb:

```BASH
sudo apt update
sudo apt install mongodb-org
```

6. Una vez instalado, inciar el servicio y permitirlo en boot:

```BASH
sudo systemctl start mongod
sudo systemctl enable mongod
```

7. Instalada la DB, hay que crear un usuario y grupo para el sistema de chat, en este caso es 'rocket':

```BASH
sudo useradd -m -U -r -d /opt/rocket rocket
```

8. Se agrega el www-data al grupo y se dan permisos al folder /opt/rocket para que nginx pueda tomar la config:

```BASH
sudo usermod -a -G rocket www-data
sudo chmod 750 /opt/rocket
```

9. Para instalar el software Rocket.Chat, se cambia al usuario rocket y se descarga con curl el software:

```BASH
sudo su - rocket
curl -L https://releases.rocket.chat/latest/download -o rocket.chat.tgz
```

10. Cuando finalize la descarga, se descomprime y se cambia el nombre a Rocket.Chat

```BASH
tar zxf rocket.chat.tgz
mv bundle Rocket.Chat
```

11. Ir a la ruta: Rocket.Chat/programs/server e instalar todos los paquetes:

```BASH
cd Rocket.Chat/programs/server
npm install
```

12. Para testear la instalacion antes de crear el servicio de systemd, setteamos unas variables de entorno:

```BASH
export PORT=3000
export ROOT_URL=http://0.0.0.0:3000/
export MONGO_URL=mongodb://localhost:27017/rocketchat
```

13. Desde el directorio que estamos actualmente: /opt/rocket/Rocket.Chat/programs/server, vamos al Rocket.Chat e iniciamos los servicios:

```BASH
cd ../../
node main.js
```

14. Tiene que dar un output como este:

```dsconfig
LocalStore: store created at
LocalStore: store created at
LocalStore: store created at
Setting default file store to GridFS
{"line":"120","file":"migrations.js","message":"Migrations: Not migrating, already at version 197","time":{"\$date":1593992291014},"level":"info"}
Loaded the Apps Framework and loaded a total of 0 Apps!
Using GridFS for custom sounds storage
Using GridFS for custom emoji storage
Updating process.env.MAIL_URL
Browserslist: caniuse-lite is outdated. Please run next command `npm update`
➔ System ➔ startup
➔ +---------------------------------------------+
➔ | SERVER RUNNING |
➔ +---------------------------------------------+
➔ | |
➔ | Rocket.Chat Version: 3.4.1 |
➔ | NodeJS Version: 12.14.0 - x64 |
➔ | MongoDB Version: 4.0.19 |
➔ | MongoDB Engine: wiredTiger |
➔ | Platform: linux |
➔ | Process Port: 3000 |
➔ | Site URL: http://0.0.0.0:3000/ |
➔ | ReplicaSet OpLog: Enabled |
➔ | Commit Hash: 21157c0c4f |
➔ | Commit Branch: HEAD |
➔ | |
➔ +---------------------------------------------+
```

En caso de tener algun error de oplog-required hay que activar la replica en mongoDB:
https://docs.rocket.chat/installation/manual-installation/mongo-replicas/

```BASH
echo -e "replication:\n replSetName: \"rs01\"" | sudo tee -a /etc/mongod.conf

sudo systemctl restart mongod
```

```
mongo
> rs.initiate()
```

El output tiene que ser como este:

```JSON
{
"info2" : "no configuration specified. Using a default configuration for the set",
"me" : "127.0.0.1:27017",
"ok" : 1,
"operationTime" : Timestamp(1593991356, 1),
"\$clusterTime" : {
"clusterTime" : Timestamp(1593991356, 1),
"signature" : {
"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
"keyId" : NumberLong(0)
}
}
}
```

En ok, tiene que tener un valor de 1, si sale 93 significa problemas.

Si esta bien, presiona enter y el prompt tiene que cambiar a primary:

```
rs01:PRIMARY>
```

Salir con exit.

```
rs01:PRIMARY> exit
```

15. Creamos un servicio en systemd para ejecutar el software como un servicio:

```
sudo vim /etc/systemd/system/rocketchat.service
```

16. Insertar:

```ini
[Unit]
Description=Rocket.Chat server
After=network.target nss-lookup.target mongod.target

[Service]
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=rocketchat
User=rocket
Environment=MONGO_URL=mongodb://localhost:27017/rocketchat ROOT_URL=https://chat.example.com PORT=3000
ExecStart=/usr/local/bin/node /opt/rocket/Rocket.Chat/main.js

[Install]
WantedBy=multi-user.target
```

En caso de tener el problema de oplog, en la linea de environment agregar:

```ini
MONGO_OPLOG_URL=mongodb://localhost:27017/local?replSet=rs01
```

17. Cargamos el servicio al sistema y lo iniciamos:

```
sudo systemctl daemon-reload
sudo systemctl start rocketchat
```

18. Revisar status con:

```
sudo systemctl status rocketchat
```

19. Y se da enable con:

```
sudo systemctl enable rocketchat
```

20. Si todo salio bien, se puede acceder al servicio con solo digitar la IP del servidor en el navegador.

Ejemplo:

> http://10.10.0.45
