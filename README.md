# INSTALACIÓN DE ODDO CON DOCKER-COMPOSE

Primero creamos un nuevo repositorio que llamaremos Tarea_10_SXE en el que subiremos nuestro proyecto a Github.
Entramos en la terminal y creamos una nueva carpeta, en nuestro caso se llamara *Odoo* en la que podremos añadir todos los archivos relacionados con esta instalación.

EL primer paso para instalar el Odoo con docker es realizar nuestro docker-compose.yml, el archivo con el que podremos crear y poner en funcionamento los contenedores de nuestro servicio que en este caso será : Odoo, nuestro servicio base y por ende el más importante, PostgresSQL, la BD que utilizaremos para hacer funcionar nuestro servicio base y por último PGAdmin, un gestor de base de datos que nos permitirá visualizar de manera más gráfica y con una interfaz el contenido de las BD.

Podemos sacar la información que necesitanmos para nuestro archivo de instalación en docker HUB. Nuestroi archivo quedará así

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
      PGADMIN_DEFAULT_EMAIL: cr.m23@hotmail.com  # Correo electrónico de la cuenta de administrador de PgAdmin
      PGADMIN_DEFAULT_PASSWORD: PgAdminPassword  # Contraseña de la cuenta de administrador de PgAdmin
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

Ahoara ya podemos entrar en la dirección

localhost:8069

y nos encontramos con esto

![ODOO](/img/odooWEB8069.png)

Si llamamos a la base de datos "postgres" que es como lllamamos a la base de datos del docker-compose nos dará este error.

![ErrorBD](/img/basededatosYaExistente.png)

Por ello debemos de rellenar los datos y llamar a la base de datos con otro nombre

![ODOO](/img/newDatabase.png)

Después de eso podemos logearnos con el email y contraseña que pusimos al registrar la bd

![ODOO](/img/entramosLogin.png)

Al registrarnos ya podemos estar dentro de Odoo

![ODOO](/img/entramos en odoo.png)










