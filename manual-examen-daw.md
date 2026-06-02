# Manual de apoyo para examen – DAW

## Basado en las prácticas del repositorio `Ngel07/DAW`

Este documento está pensado como **material de consulta rápida para examen**. Incluye:

- respuestas modelo para entregar,
- comandos exactos en orden,
- resultados esperados,
- errores típicos y cómo solucionarlos,
- plantillas completas de archivos.

---

# ÍNDICE

1. [Unidad 1 – Práctica Evaluación Continua 1](#unidad-1--práctica-evaluación-continua-1)
2. [Unidad 2 – Práctica Evaluación Continua 2](#unidad-2--práctica-evaluación-continua-2)
3. [Unidad 3 – Práctica Evaluación Continua 3](#unidad-3--práctica-evaluación-continua-3)
4. [Anexo – Comandos rápidos](#anexo--comandos-rápidos)
5. [Anexo – Errores frecuentes](#anexo--errores-frecuentes)

---

# UNIDAD 1 – PRÁCTICA EVALUACIÓN CONTINUA 1

## 1. Personalizar error 404 en Apache y Nginx

### Respuesta modelo para entregar
He personalizado la página de error 404 tanto en Apache como en Nginx creando un archivo `404.html` con mi nombre y configurando ambos servidores para que muestren esa página cuando el recurso solicitado no exista. Después reinicié los servicios y comprobé su funcionamiento accediendo a una ruta inexistente desde el navegador y con `curl`.

---

## A) Apache – Página 404 personalizada

### Paso 1. Crear la carpeta del proyecto
```bash
sudo mkdir -p /var/www/miproyecto
```

### Paso 2. Crear la página 404
```bash
sudo nano /var/www/miproyecto/404.html
```

Contenido:
```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Error 404</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin-top: 60px;
      background-color: #f4f4f4;
    }
    h1 {
      color: red;
    }
  </style>
</head>
<body>
  <h1>Error 404 - Página no encontrada</h1>
  <p>Autor: TU NOMBRE</p>
</body>
</html>
```

### Paso 3. Editar el sitio por defecto de Apache
```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Ejemplo de configuración:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/miproyecto
    ErrorDocument 404 /404.html
</VirtualHost>
```

### Paso 4. Reiniciar Apache
```bash
sudo systemctl restart apache2
```

### Paso 5. Probar el error 404
```bash
curl -i http://localhost/noexiste
```

### Resultado esperado
- Código `404 Not Found`
- Tu página personalizada con tu nombre

---

## B) Nginx – Página 404 personalizada

### Paso 1. Crear la página 404
```bash
sudo nano /var/www/miproyecto/404.html
```

Puedes usar el mismo contenido HTML.

### Paso 2. Editar el sitio por defecto de Nginx
```bash
sudo nano /etc/nginx/sites-available/default
```

Ejemplo de configuración:
```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/miproyecto;
    index index.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;

    location = /404.html {
        internal;
    }
}
```

### Paso 3. Comprobar sintaxis
```bash
sudo nginx -t
```

### Salida esperada
```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### Paso 4. Reiniciar Nginx
```bash
sudo systemctl restart nginx
```

### Paso 5. Probar
```bash
curl -i http://localhost/noexiste
```

---

## 2. Cambiar Apache y Nginx al puerto 8989

### Respuesta modelo para entregar
He modificado la configuración de Apache y Nginx para que el sitio por defecto escuche en el puerto `8989`. En Apache he cambiado `ports.conf` y el `VirtualHost`, y en Nginx la directiva `listen` del sitio por defecto. Después he reiniciado ambos servicios y he comprobado su acceso mediante `http://localhost:8989`.

---

## A) Apache en 8989

### Paso 1. Editar puertos
```bash
sudo nano /etc/apache2/ports.conf
```

Cambiar:
```apache
Listen 80
```
por:
```apache
Listen 8989
```

### Paso 2. Editar VirtualHost
```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Cambiar:
```apache
<VirtualHost *:80>
```
por:
```apache
<VirtualHost *:8989>
```

### Paso 3. Reiniciar Apache
```bash
sudo systemctl restart apache2
```

### Paso 4. Probar
```bash
curl http://localhost:8989
```

---

## B) Nginx en 8989

### Paso 1. Editar configuración
```bash
sudo nano /etc/nginx/sites-available/default
```

Cambiar:
```nginx
listen 80 default_server;
listen [::]:80 default_server;
```
por:
```nginx
listen 8989 default_server;
listen [::]:8989 default_server;
```

### Paso 2. Validar y reiniciar
```bash
sudo nginx -t
sudo systemctl restart nginx
```

### Paso 3. Probar
```bash
curl http://localhost:8989
```

---

## 3. Multisitio en Nginx

### Respuesta modelo para entregar
He configurado varios sitios en la misma máquina virtual con Nginx. Para ello he creado dos carpetas web independientes, he creado dos archivos de configuración en `sites-available`, los he habilitado en `sites-enabled` y he asignado un puerto distinto a cada sitio. Finalmente, comprobé que ambos sitios funcionaban simultáneamente.

---

### Paso 1. Crear carpetas web
```bash
sudo mkdir -p /var/www/sitio1
sudo mkdir -p /var/www/sitio2
```

### Paso 2. Crear páginas de prueba
```bash
echo "<h1>Sitio 1 en Nginx</h1>" | sudo tee /var/www/sitio1/index.html
echo "<h1>Sitio 2 en Nginx</h1>" | sudo tee /var/www/sitio2/index.html
```

### Paso 3. Crear configuración del sitio 1
```bash
sudo nano /etc/nginx/sites-available/sitio1
```

Contenido:
```nginx
server {
    listen 8081;
    server_name sitio1.local;
    root /var/www/sitio1;
    index index.html;
}
```

### Paso 4. Crear configuración del sitio 2
```bash
sudo nano /etc/nginx/sites-available/sitio2
```

Contenido:
```nginx
server {
    listen 8082;
    server_name sitio2.local;
    root /var/www/sitio2;
    index index.html;
}
```

### Paso 5. Habilitar los sitios
```bash
sudo ln -s /etc/nginx/sites-available/sitio1 /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/sitio2 /etc/nginx/sites-enabled/
```

### Paso 6. Validar y reiniciar
```bash
sudo nginx -t
sudo systemctl restart nginx
```

### Paso 7. Probar ambos sitios
```bash
curl http://localhost:8081
curl http://localhost:8082
```

---

## 4. Apache + Nginx simultáneos y multisitio

### Respuesta modelo para entregar
He instalado Apache y Nginx en la misma máquina virtual y he configurado varios sitios en ambos. Como no pueden compartir el mismo puerto, he asignado puertos diferentes a cada sitio. En total he configurado cuatro sitios: dos en Apache y dos en Nginx, todos funcionando simultáneamente.

---

## Ejemplo de distribución de puertos
- Apache sitio 1 → `8080`
- Apache sitio 2 → `8083`
- Nginx sitio 1 → `8081`
- Nginx sitio 2 → `8082`

---

## Apache sitio 1
```bash
sudo mkdir -p /var/www/apache1
echo "<h1>Apache sitio 1</h1>" | sudo tee /var/www/apache1/index.html
sudo nano /etc/apache2/sites-available/apache1.conf
```

Contenido:
```apache
<VirtualHost *:8080>
    DocumentRoot /var/www/apache1
</VirtualHost>
```

## Apache sitio 2
```bash
sudo mkdir -p /var/www/apache2
echo "<h1>Apache sitio 2</h1>" | sudo tee /var/www/apache2/index.html
sudo nano /etc/apache2/sites-available/apache2.conf
```

Contenido:
```apache
<VirtualHost *:8083>
    DocumentRoot /var/www/apache2
</VirtualHost>
```

## Añadir puertos Apache
```bash
sudo nano /etc/apache2/ports.conf
```

Añadir:
```apache
Listen 8080
Listen 8083
```

## Habilitar sitios Apache
```bash
sudo a2ensite apache1.conf
sudo a2ensite apache2.conf
sudo systemctl restart apache2
```

## Nginx
Utilizar la configuración del apartado anterior para `8081` y `8082`.

## Comprobación final
```bash
curl http://localhost:8080
curl http://localhost:8081
curl http://localhost:8082
curl http://localhost:8083
```

---

## 5. Acceso por SSH y FTP

### Respuesta modelo para entregar
He accedido por SSH al servidor desde la máquina anfitriona usando la IP de la máquina virtual. También he instalado y configurado un servidor FTP para transferir archivos desde FileZilla.

---

## SSH
### Instalar OpenSSH Server
```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl status ssh
```

### Permitir SSH en firewall
```bash
sudo ufw allow ssh
sudo ufw enable
sudo ufw status
```

### Ver IP del servidor
```bash
ip a
```

### Conectar desde otra máquina
```bash
ssh usuario@192.168.X.X
```

---

## FTP
### Instalar servidor FTP
```bash
sudo apt install vsftpd -y
sudo systemctl restart vsftpd
sudo systemctl status vsftpd
```

### Datos para FileZilla
- Host: IP de la máquina virtual
- Usuario: usuario del sistema
- Contraseña: contraseña del sistema
- Puerto: `21`

---

# UNIDAD 2 – PRÁCTICA EVALUACIÓN CONTINUA 2

## 1. Dockerfile basado en Apache Alpine

### Respuesta modelo para entregar
He clonado el repositorio indicado y he modificado el `Dockerfile` para basar la imagen en un servidor Apache sobre Alpine. También he configurado la copia de los archivos de la carpeta `static` al directorio público del servidor Apache. Después construí la imagen y arranqué un contenedor mapeando el puerto `80` del contenedor al `9090` del host.

---

### Paso 1. Clonar repositorio
```bash
git clone https://github.com/JuanraCollado/octopus-underwater-app.git
cd octopus-underwater-app
```

### Paso 2. Crear o modificar Dockerfile
```bash
nano Dockerfile
```

Contenido:
```dockerfile
FROM httpd:alpine
COPY static/ /usr/local/apache2/htdocs/
EXPOSE 80
```

### Paso 3. Construir imagen
```bash
docker build -t octopus-app .
```

### Resultado esperado
```bash
Successfully tagged octopus-app:latest
```

### Paso 4. Ejecutar contenedor
```bash
docker run -d --name octopus-container -p 9090:80 octopus-app
```

### Paso 5. Comprobar
```bash
docker ps
curl http://localhost:9090
```

---

## 2. MySQL + phpMyAdmin con red y volumen persistente

### Respuesta modelo para entregar
He desplegado un entorno con MySQL y phpMyAdmin en Docker. He creado una red propia para que ambos contenedores se comuniquen por nombre, y un volumen con nombre para garantizar la persistencia de los datos de MySQL. Finalmente accedí a phpMyAdmin desde `localhost:8080` y comprobé la conectividad con la base de datos.

---

### Paso 1. Crear red Docker
```bash
docker network create examen-net
```

### Paso 2. Crear volumen
```bash
docker volume create mysql_data
```

### Paso 3. Lanzar MySQL
```bash
docker run -d \
  --name mysql57 \
  --network examen-net \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=testdb \
  -v mysql_data:/var/lib/mysql \
  mysql:5.7
```

### Paso 4. Lanzar phpMyAdmin
```bash
docker run -d \
  --name phpmyadmin \
  --network examen-net \
  -e PMA_HOST=mysql57 \
  -e PMA_USER=root \
  -e PMA_PASSWORD=rootpass \
  -p 8080:80 \
  phpmyadmin/phpmyadmin
```

### Paso 5. Verificar contenedores
```bash
docker ps
```

### Paso 6. Acceder al navegador
- URL: `http://localhost:8080`
- Usuario: `root`
- Contraseña: `rootpass`

### Paso 7. Comprobación
Crear una tabla desde phpMyAdmin para demostrar conectividad.

---

# UNIDAD 3 – PRÁCTICA EVALUACIÓN CONTINUA 3

## 1. Explicación del fichero docker-compose

### Respuesta modelo para entregar
Este fichero `docker-compose.yml` define un entorno con tres servicios: una aplicación PHP con Apache, una base de datos MySQL y phpMyAdmin para administrar la base de datos. Además, usa un volumen persistente para conservar la información de MySQL y define dependencias entre servicios para facilitar su arranque coordinado.

---

## Versión comentada del fichero
```yaml
services:
  app:
    image: php:8.3-apache         # Imagen con Apache y PHP
    container_name: php_app       # Nombre del contenedor
    volumes:
      - ~/Escritorio/projectesPHP:/var/www/html   # Monta la carpeta local en la raíz web
    ports:
      - "8080:80"                 # Puerto 8080 del host al 80 del contenedor
    depends_on:
      - db                        # Depende del servicio db
    restart: always               # Reinicio automático

  db:
    image: mysql:8.0              # Imagen de MySQL
    container_name: mysql_db
    environment:
      MYSQL_ROOT_PASSWORD: rootpass   # Contraseña root
      MYSQL_DATABASE: testdb          # Base de datos inicial
      MYSQL_USER: user                # Usuario adicional
      MYSQL_PASSWORD: userpass        # Contraseña del usuario
    volumes:
      - db_data:/var/lib/mysql        # Volumen persistente para datos
    restart: always

  phpmyadmin:
    image: phpmyadmin/phpmyadmin      # Imagen de phpMyAdmin
    container_name: phpmyadmin
    environment:
      PMA_HOST: db                    # Host de MySQL en la red Docker
      PMA_USER: root
      PMA_PASSWORD: rootpass
    ports:
      - "8081:80"                    # Puerto del host para phpMyAdmin
    depends_on:
      - db
    restart: always

volumes:
  db_data:                            # Definición del volumen
```

---

## 2. Entorno completo con Docker Compose

### Respuesta modelo para entregar
He desplegado un entorno web profesional con Docker Compose formado por tres servicios: Apache con PHP, MySQL y phpMyAdmin. La aplicación PHP se sirve desde la carpeta local `./web`, MySQL guarda sus datos en un volumen persistente y phpMyAdmin está expuesto en el puerto `8081`. La aplicación PHP se conecta a la base de datos y muestra un mensaje indicando si la conexión ha sido correcta.

---

## Estructura del proyecto
```bash
mkdir examen-compose
cd examen-compose
mkdir web
```

## Archivo `web/index.php`
```php
<?php
$servername = getenv('MYSQL_HOST');
$username = getenv('MYSQL_USER');
$password = getenv('MYSQL_PASSWORD');
$dbname = getenv('MYSQL_DATABASE');

$conn = new mysqli($servername, $username, $password, $dbname);

if ($conn->connect_error) {
    die("<h2 style='color:red;'>Conexión fallida: " . $conn->connect_error . "</h2>");
}

echo "<h2 style='color:green;'>Conexión exitosa a la base de datos '$dbname' en el host '$servername'</h2>";
$conn->close();
?>
```

## Archivo `Dockerfile`
```dockerfile
FROM php:8.3-apache
RUN docker-php-ext-install mysqli
```

## Archivo `docker-compose.yml`
```yaml
services:
  web:
    build: .
    container_name: web_php
    ports:
      - "8080:80"
    volumes:
      - ./web:/var/www/html
    environment:
      MYSQL_HOST: db
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DATABASE: blog
    depends_on:
      - db
    restart: always

  db:
    image: mysql:8.0
    container_name: mysql_blog
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: blog
    volumes:
      - db_data:/var/lib/mysql
    restart: always

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: pma_blog
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      PMA_USER: root
      PMA_PASSWORD: secret
    depends_on:
      - db
    restart: always

volumes:
  db_data:
```

## Levantar el entorno
```bash
docker compose up -d --build
```

## Comprobar
```bash
docker compose ps
```

## Accesos
- Aplicación: `http://localhost:8080`
- phpMyAdmin: `http://localhost:8081`

---

## 3. Misma infraestructura con Kubernetes y 3 pods web

### Respuesta modelo para entregar
He reproducido la infraestructura anterior en Kubernetes mediante un manifiesto YAML único con varios recursos. He creado un Deployment para la parte web con 3 réplicas, un Deployment y Service para MySQL, y un Deployment y Service para phpMyAdmin. Después he desplegado todo con `kubectl apply` y he verificado los pods y servicios creados.

---

## Archivo `k8s-app.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: php:8.3-apache
          ports:
            - containerPort: 80
          env:
            - name: MYSQL_HOST
              value: mysql-service
            - name: MYSQL_USER
              value: root
            - name: MYSQL_PASSWORD
              value: secret
            - name: MYSQL_DATABASE
              value: blog
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: secret
            - name: MYSQL_DATABASE
              value: blog
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: phpmyadmin-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: phpmyadmin
  template:
    metadata:
      labels:
        app: phpmyadmin
    spec:
      containers:
        - name: phpmyadmin
          image: phpmyadmin/phpmyadmin
          ports:
            - containerPort: 80
          env:
            - name: PMA_HOST
              value: mysql-service
            - name: PMA_USER
              value: root
            - name: PMA_PASSWORD
              value: secret
---
apiVersion: v1
kind: Service
metadata:
  name: phpmyadmin-service
spec:
  type: NodePort
  selector:
    app: phpmyadmin
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30081
```

## Desplegar
```bash
kubectl apply -f k8s-app.yaml
```

## Comprobar pods
```bash
kubectl get pods
```

## Comprobar servicios
```bash
kubectl get svc
```

## Accesos
- Web: `http://localhost:30080`
- phpMyAdmin: `http://localhost:30081`

---

# ANEXO – COMANDOS RÁPIDOS

## Apache
```bash
sudo apt install apache2 -y
sudo systemctl status apache2
sudo nano /etc/apache2/ports.conf
sudo nano /etc/apache2/sites-available/000-default.conf
sudo systemctl restart apache2
sudo apachectl configtest
```

## Nginx
```bash
sudo apt install nginx -y
sudo systemctl status nginx
sudo nano /etc/nginx/sites-available/default
sudo nginx -t
sudo systemctl restart nginx
```

## SSH / FTP
```bash
sudo apt install openssh-server -y
sudo ufw allow ssh
sudo apt install vsftpd -y
ip a
ssh usuario@IP
```

## Docker
```bash
docker run -it ubuntu bash
docker ps
docker ps -a
docker network create examen-net
docker volume create mysql_data
docker build -t nombreimagen .
docker run -d --name nombre -p 8080:80 imagen
```

## Docker Compose
```bash
docker compose version
docker compose up -d --build
docker compose ps
docker compose down
```

## Kubernetes
```bash
kubectl apply -f archivo.yaml
kubectl get pods
kubectl get svc
kubectl describe pod nombre-pod
kubectl exec -it nombre-pod -- bash
```

---

# ANEXO – ERRORES FRECUENTES

## 1. Apache no arranca
```bash
sudo apachectl configtest
```
Revisar errores de sintaxis en `ports.conf` o VirtualHost.

## 2. Nginx no arranca
```bash
sudo nginx -t
```
Suele deberse a un `;` faltante o llaves mal cerradas.

## 3. El puerto está ocupado
```bash
sudo ss -tulpn | grep 80
sudo ss -tulpn | grep 8080
sudo ss -tulpn | grep 8989
```

## 4. phpMyAdmin no conecta con MySQL
- Verificar que ambos contenedores estén en la misma red.
- Revisar `PMA_HOST`, usuario y contraseña.

## 5. PHP no conecta con MySQL en Compose
Si da error con `mysqli`, usar este `Dockerfile`:
```dockerfile
FROM php:8.3-apache
RUN docker-php-ext-install mysqli
```

## 6. Kubernetes no expone la app
```bash
kubectl get pods
kubectl get svc
kubectl describe svc web-service
```
Verificar `type: NodePort` y el puerto asignado.
