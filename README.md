# INSTALACIÓN DE ODOO CON DOCKER-COMPOSE

Este documento explica cómo instalar Odoo con Docker Compose, configurando Odoo, PostgreSQL y PgAdmin. A continuación, se detallan los pasos para crear y ejecutar los contenedores necesarios para que el sistema funcione correctamente.

# Índice

1. [Instalación ODOO](#Instalación-ODOO)
2. [Docker-Compose](#DOCKER-COMPOSE)
3. [Instalación de PGAdmin](#Instalación-PGADMIN)
4. [Preguntas sobre la tarea](#preguntas-sobre-la-tarea)


## Instalación ODOO


### 1. Crear el repositorio

Comienza creando un nuevo repositorio llamado `Tarea_10_SXE` en GitHub, donde se almacenará el proyecto.

### 2. Crear la carpeta del proyecto

En tu terminal, crea una nueva carpeta para el proyecto. En este caso, la llamaremos `Odoo`, y dentro de ella agregarás todos los archivos necesarios para esta instalación.

```bash
mkdir Odoo
cd Odoo
```

### 3. Realizar nuestro docker-compose.yml

Para instalar el Odoo con docker debemos realizar nuestro docker-compose.yml, el archivo con el que podremos crear y poner en funcionamento los contenedores de nuestro servicio que en este caso será : Odoo, nuestro servicio base y por ende el más importante, PostgresSQL, la BD que utilizaremos para hacer funcionar nuestro servicio base y por último PGAdmin, un gestor de base de datos que nos permitirá visualizar de manera más gráfica y con una interfaz el contenido de las BD.

Podemos sacar la información que necesitanmos para nuestro archivo de instalación en docker HUB. 

## DOCKER-COMPOSE:

```
services:  # Definición de los servicios que utilizará la aplicación
  web:  # Servicio correspondiente a la aplicación Odoo
    image: odoo:17.0  # Usamos la imagen oficial de Odoo versión 17, que sigue recibiendo soporte
    container_name: odooWebContainer  # Nombre del contenedor que ejecutará Odoo
    restart: unless-stopped  # Configuración para reiniciar el contenedor solo si se detiene inesperadamente
    depends_on:  # Especificamos que Odoo depende del servicio de base de datos para iniciarse
      - db
    ports:  # Definimos el mapeo de puertos para que Odoo sea accesible externamente
      - "8069:8069"  # El puerto 8069 de Odoo se expone al mismo puerto en el host
    environment:  # Variables de entorno específicas para la configuración de Odoo
      HOST: db  # Odoo se conectará al contenedor de la base de datos con el nombre "db"
      USER: odoo  # Usuario de la base de datos para la conexión
      PASSWORD: myodoo  # Contraseña del usuario de la base de datos
    volumes:  # Definimos volúmenes para persistir los datos de Odoo
      - odoo-web-data:/var/lib/odoo  # Los datos de Odoo se guardarán de forma persistente en este volumen

  db:  # Servicio que contiene la base de datos PostgreSQL
    image: postgres:15  # Usamos la imagen oficial de PostgreSQL versión 15
    container_name: postgresSQLContainer  # Nombre del contenedor que ejecutará PostgreSQL
    restart: unless-stopped  # Configuración de reinicio automático del contenedor
    environment:  # Variables de entorno específicas para configurar PostgreSQL
      POSTGRES_DB: postgres  # Nombre de la base de datos por defecto
      POSTGRES_PASSWORD: myodoo  # Contraseña para el usuario 'odoo'
      POSTGRES_USER: odoo  # Usuario por defecto de la base de datos
    volumes:  # Volúmenes para la persistencia de datos de PostgreSQL
      - odoo-db-data:/var/lib/postgresql/data  # Persistimos los datos de la base de datos en este volumen
    ports:  # Definimos el mapeo de puertos para acceder a PostgreSQL
      - "5432:5432"  # El puerto 5432 de PostgreSQL se expone al mismo puerto en el host

  pgadmin:  # Servicio para el gestor de base de datos PgAdmin
    restart: unless-stopped  # Aseguramos que PgAdmin se reinicie solo si se detiene inesperadamente
    image: dpage/pgadmin4:latest  # Usamos la versión más reciente de la imagen oficial de PgAdmin
    container_name: PgAdminContainer  # Nombre del contenedor para PgAdmin
    depends_on:  # Indica que PgAdmin depende del servicio de base de datos para iniciar
      - db
    ports:  # Configuración de los puertos de PgAdmin
      - "5050:80"  # El puerto 80 de PgAdmin se mapea al puerto 5050 del host, para acceder desde un navegador
    environment:  # Variables de entorno para configurar la cuenta de PgAdmin
      PGADMIN_DEFAULT_EMAIL: kiglesiasesteves@danielcastelao.org  # Correo electrónico de la cuenta de administrador de PgAdmin
      PGADMIN_DEFAULT_PASSWORD: PgAdmin  # Contraseña de la cuenta de administrador de PgAdmin
    volumes:  # Persistencia de los datos de configuración de PgAdmin
      - pgadmin-data:/var/lib/pgadmin  # Persistencia de los datos de PgAdmin en un volumen

volumes:  # Definición de volúmenes persistentes para los datos de los servicios
  odoo-web-data:  # Volumen donde se guardan los datos persistentes de Odoo
  odoo-db-data:  # Volumen para la persistencia de los datos de la base de datos PostgreSQL
  pgadmin-data:  # Volumen para guardar los datos persistentes de PgAdmin
```
Al instalar este docker-compose hemos tenido este problema.
![Problema](/img/Screenshot_20250114_114033.png)

AL ver cual era el servicio que estaba utilizando ya de por si este puerto podemos ver que es postgres.
![postgres](/img/postgresEnUso.png)

En este caso tenemos dos opciones dependiendo de lo que queramos hacer y de la prioridad que tiene el servicio nuevo que queremos instalar. Podemos parar el servicio de postgres que se está ejecutando en nuestro ordenador con el comadno
```
sudo systemctl stop postgresql
```
O podemos cambiar el puerto que tenemos en el docker compose por ejemplo al 5433.
En nuestro caso no queremos eliminar el servicio de postgres en el ordenador así que cambiaremos el puerto del docker compose a
```
  db:
    ports:
      - "5433:5432"  # Cambia el puerto externo a 5433
```

Volvemos a subir los contenedores y podemos comprobar si se subieron con el comando
sudo docker ps -a

![subimosContenedores](/img/subidacontenedores.png)

### 4. Configurar el Odoo en el navegador

Para eso entramos en el navegador de nuestra preferencia con la dirección
´´´
http://localhost:8069
´´´
y nos encontramos con esto

![ODOO](/img/odooWEB8069.png)

Si llamamos a la base de datos "postgres" que es como lllamamos a la base de datos del docker-compose nos dará este error.

![ErrorBD](/img/basededatosYaExistente.png)

Por ello debemos de rellenar los datos y llamar a la base de datos con otro nombre

![ODOO](/img/newDatabase.png)

Después de eso podemos logearnos con el email y contraseña que pusimos al registrar la bd

![ODOO](/img/entramosLogin.png)

Al registrarnos ya podemos estar dentro de Odoo

![ODOO](/img/entramosEnOdoo.png)

## Instalación PGADMIN

Ahora que tenemos Odoo COmmunity versión 17 instalado podemos ver si PGAdmin también se ha instalado correctamente. Para eso, vamos a entrar en el localhost:5050. En la interfaz colocamos los datos que pusimos en el localhost para el registro

![PGADMIN](/img/PGADMIN.png)

Después al entrar nos encontraremos con esto

![PGADMIN](/img/DENTRODEPGADMIN.png)

Pero no estará el servidor de odoo, por eso tenemos que añadir ese servidor con el siguiente proceso, siempre registrando las palabras que nosotros hemos colocadoe n nuestro docker-compose.Primero colocamso el nombre

![PGADMIN](/img/registerServer.png)

Posteriormente colocamos el puerto, la bd, los datos, el username...

![PGADMIN](/img/registroServer2.png)

Guardamos el servidor y tendremos esto en PGADMIN, comprobando su funcionamento

![PGADMIN](/img/funcionamientoPGADMIN.png)

## PREGUNTAS SOBRE LA TAREA

¿Que ocurre si en el ordenador local el puerto 5432 está ocupado? ¿Y si lo estuviese el 8069? ¿Como puedes solucionarlo?

Esta pregunta ya ha sido respondida a lo largo del ejercicio cuando el puerto 5432 estaba efectivamente ocupado por un proceso. Ya vimos la solución para el puerto 5432 pero cual sería la solución para el 8069?

Sería lo mismo podemos modificar el puerto para 8070:8069 y reiniciar los contenedores como hicimos con el otro puerto. Así no tendremos problemas y conseguimos ejecutar los servicios sin fallos. 

















