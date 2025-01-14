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

