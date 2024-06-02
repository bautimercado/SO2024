# Practica 5 - Docker Compose

## 1. Utilizando sus palabras describa, ¿qué es docker compose?

- Es una herramienta que simplifica el proceso de ejecutar, configurar y gestionar múltiples contenedores Docker.
- Permite definir y orquestar varios contenedores como una aplicación única, lo que facilita el manejo de servicios y dependencias complejas.
- En vez de tener que ejecutar comandos Docker largos y complejos para cada contenedor, Docker Compose permite definir y configurar todos los servicios de una aplicación.

## 2. ¿Qué es el archivo compose y cual es su función? ¿Cuál es el “lenguaje” del archivo?

- Es un archivo YAML que define los servicios, redes y volúmenes que componen una aplicación. Actúa como una receta que describe cómo deben configurarse y relacionarse los componentes de la aplicación.
- Cada servicio se define como un contenedor Docker, y se pueden especificar detalles como la imagen base, variables de entorno, puertos expuestos, volúmenes montados, dependencias, etc.

## 3. ¿Cuáles son las versiones existentes del archivo docker-compose.yaml existentes y qué características aporta cada una? ¿Son compatibles entre sí? ¿Por qué?

- Versión 1: Versión inicial de Docker Compose. Todavía es compatible, pero se recomienda usar versiones más recientes.
- Versión 2.x: Introdujo nuevas características como redes definidas por el usuario, mejor soporte para volúmenes, configurar opciones de construcciones de imágenes, puntos de montaje de volúmenes consistentes, escalar servicios, configurar credenciales, configurar recurso de CPU y memoria.
- Versión 3.x: Introdujo nuevas opciones, elimina la sintaxis de la versión 1, nuevas opciones de red, implementación de pilas, configuración de recursos mejorada (añadiendo restricciones), soporte para secrets, etc.
- La versión 1 no es compatible y está obsoleta, la versión 2 es compatible con la versión 3, pero carece de ciertas funcionalidades.

## 4. Investigue y describa la estructura de un archivo compose. Desarrolle al menos sobre los siguientes bloques indicando para qué se usan:

### a. services

- Bloque principal donde se definen todos los servicios (contenedores) que componen la aplicación.
- Cada servicio se especifica con un nombre único y se configura con una serie de opciones.

### b. build

- Se usa dentro de un servicio para especificar la ruta del Dockerfile o el contexto de construcción para crear la imagen del contenedor.
- Se suele usar un "." cuando el Dockerfile está en el mismo directorio.

### c. image

- En lugar de construir una imagen desde un Dockerfile (no usaríamos `build`), nos permite espeificar una imagen preexistente que se usará para el contenedor.

### d. volumes

- Define los volúmenes de datos que se montarán en los contenedores. Estos volúmenes pueden ser volúmenes nombrados, anónimos o bind mounts desde el host.

### e. restart

- Determina la política de reinicio para un servicio.
- Las opciones comunes son `no` (nunca reiniciar), `always` (siempre reiniciar), `on-failure` (reiniciar en caso de fallo) y `unless-stopped` (reiniciar siempre, excepto después de `docker stop`)

### f. depends_on

- Establece dependencias entre servicios.
- Un servicio solo se iniciará después de que los servicios de los que depende estén en ejecución.

### g. environment

- Define las variables de entorno que se establecerán dentro de los contenedores.

### h. ports

- Mapea los puertos expuestos por el contenedor a puertos en el host.
- Esto permite accedere a los servicios que se ejecutan dentro de los contenedores desde el host.

### i. expose

- Expone puertos desde el contenedor, pero sin mapearlos al host. Esto es útil cuando se usa un proxy de reserva o un load balancer.

### j. networks

- Define las redes Docker a las que se conectarán los contenedores. Se pueden especificar redes predefinidas o definir nuevas redes.

Ejemplo:

```yaml
version: '3.8'

services:

  web:
    build:
      context: ./web
      args:
        - ENV=development
    ports:
      - "8000:8000"
    volumes:
      - ./web:/code
    environment:
      - DJANGO_SECRET_KEY=mySecretKey
    depends_on:
      - db

  db:
    image: postgres:13
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=mysecretpassword

volumes:
  db-data:

networks:
  default:
    driver: bridge
```

- Se definen dos servicios: `web` y `db`.
- `web` se construye a partir de un Dockerfile en el directorio `./web`, se mapea el puerto 8000 del host al puerto 8000 del contenedor, se montan los archivos locales en el contenedor y se establecen las variables de entorno.
  - En concreto, se establece un bind mount que mapea el directorio `./web` del host al directorio `/code` dentro del contenedor.
- `db` usa una imagen PostgreSQL y define un volumen de datos y una variable de entorno para la contraseña. Además se especifica que `web` depende de `db`.

## 5. Conceptualmente: ¿Cómo se podrían usar los bloques “healthcheck” y “depends_on” para ejecutar una aplicación Web dónde el backend debería ejecutarse si y sólo si la base de datos ya está ejecutándose y lista?

- El `depends_on` se puede usar para especificar que el servicio backend depende del servicio de la base de datos.

```yaml
services:
  backend: 
    depends_on:
      - db
  db:
    # configuración de la base de datos
```

- El `healthcheck` se usa para definir una comprobación de estado para un servicio. Permite especificar un comando o solicitud HTTP que se ejecutará periódicamente para determinar si el contenedor está "saludable" y listo para aceptar tráfico.
  - Existen comandos específicos para verificar cada tipo de base de datos (`pg_isready` para PostgreSQL y `mysqladmin` para MySQL).
  - También podríamos configurarlo en el backend para verificar que la aplicación web esté lista para recibir tráfico antes de que otros servicios le ruteen solicitudes (algo así falopa como un proxy inverso o load balancer).
```yaml
services:
  db:
    healtcheck:
      test: ["CMD-SHELL", "comando_para_verificar_la_bd"]
      interval: 10s
      timeout: 5s
      retries: 5
```

- La ejecución sería algo así:
  - Docker Compose inicia el contenedor de la base de datos.
  - Docker Compose ejecuta periódicamente el comando `healthcheck` en el contenedor de la base de datos.
  - Una vez que el `healtcheck` se completa (bd lista), Docker Compose inicia el contenedor backend.  

## 6. Indique qué hacen y cuáles son las diferencias entre los siguientes comandos:

### a. docker compose create y docker compose up

- `docker compose create`: Crea los contenedores definidos en `docker-compose.yml` pero no los inicia. Solo construye las imágenes y crea los contenedores en un estado detenido.
- `docker compose up`: Crea e inicia los contenedores definidos en `docker-compose.yml`. Si los contenedores existe, los reinicia. También reconstruye las imágenes si los Dockerfile cambiaron.

### b. docker compose stop y docker compose down

- `docker compose stop`: Detiene los contenedores en ejecución que fueron creados por el `docker-compose.yml`, pero no los elimina (tampoco a sus redes y volúmenes). 
- `docker compose down`: Detinene los contenedores y los elimina (junto con sus redes y volúmenes). Básicamente desmonta toda la aplicación.

### c. docker compose run y docker compose exec

- `docker compose run`: Ejecuta un comando temporalmente en un nuevo contenedor creado a partir de una imagen específica. Útil para tareas de un solo uso (migraciones, scripts de prueba, etc.)
- `docker compose exec`: Ejecuta un comando dentro de un contenedor en ejecución que forma parte de los servicios definidos en el `docker-compose.yml`.

### d. docker compose ps

- Muestra una lista de los contenedores en ejecución como parte de los servicios definidos en `docker-compose.yml`.
- Brinda información como el nombre del contenedor, comando que se está ejecutando, puertos expuestos, etc.

### e. docker compose logs

- Muestra los logs de salida de los contenedores que forman parte del `docker-compose.yml`.

## 7. ¿Qué tipo de volúmenes puede utilizar con docker compose? ¿Cómo se declara cada tipo en el archivo compose?

Se pueden usar tres tipos de volúmenes:

- **Volúmenes nombrados:** Volúmenes persistentes creados y gestionados por Docker. Pueden compartirse entre contenedores y mantienen sus datos después de eliminar los contenedores.

    ```yaml
    services:
      myservice:
        volumes:
          - named_volume:/path/inside/container
    volumes:
      named_volume:
    ```

- **Volúmenes anónimos:** Volúmenes temporales que se crean y eliminan junto con los contenedores. No tienen un nombre específico y no persisten datos.

    ```yaml
    services:
      myservice:
        volumes:
          - /path/in/container
    ```

- **Bind Mounts:** Montan un directorio del host en un directorio interno al contenedor. Cualquier cambio en el directorio del host se refleja en el contenedor y viceversa.

    ```yaml
    services:
      myservice:
        volumes:
          - /absolute/path/in/host:/path/in/container
    ```

- **Volúmenes externos:** Volúmenes que ya existen y son gestionados fuera del contexto del `docker-compose.yml`

    ```yaml
    services:
      myservice:
        volumes:
         - external_volume:/path/inside/container
    volumes:
      external_volume:
        external: true
    ```

## 8. ¿Qué sucede si en lugar de usar el comando “docker compose down” utilizo “docker compose down -v/--volumes”?

- Con `docker compose down -v` o `docker compose down --volumes`, Docker Compose también elimina los volúmenes nombrados asociados con esos contenedores (se perderían los datos almacenados).

# Instanciando Wordpress y MySQL

## Preguntas antes de ejecutar el `docker-compose.yml`

### ¿Cuántos contenedores se instancian?

- Se están instanciando dos contenedores: wordpress y db.

### ¿Por qué no se necesitan Dockerfiles?

- No se necesitan Dockerfiles ya que se están usando las imagenes correspondientes a cada servicio.

### ¿Por qué el servicio identificado como “wordpress” tiene la siguiente línea?

  ```yml
  depends_on:
    - db
  ```

- Esto es así para marcar que wordpress depende de db, de esta manera wordpress se inicia solo si db se está ejecutando.

### ¿Qué volúmenes y de qué tipo tendrá asociado cada contenedor?

- mysql tiene un volúmen nombrado (al final del archivo está declarado el volúmen `db_data`).
- wordpress tiene dos: un bind mount y otro nombrado.

### ¿Por que uso el volumen nombrado para el servicio db en lugar de dejar que se instancie un volumen anónimo con el contenedor?

  ```yml
  volumes:
    - db_data:/var/lib/mysql
  ```

- A diferencia de los volúmenes anónimos, los volúmenes nombrados me permiten persistir los datos cuando se eliminan los contenedores (lo cuál tiene sentido al ser una base de datos).

### ¿Qué genera la línea en la definición de wordpress?

  ```yml
  volumes:
    - ${PWD}:/data
  ```

- Esa línea genera un volumen de tipo _bind mount_, el cuál se monta el directorio `/data` del contenedor al directorio actual del host (eso es parte del `pwd`).

### ¿Qué representa la información que estoy definiendo en el bloque environment de cada servicio? ¿Cómo se “mapean” al instanciar los contenedores?

- La información correspondiente a `environment` son las variables de entorno relacionadas a cada servicio (en este caso representan la información para configurar la BD).
- Cada par clave valor se mapea a una variable de entorno (del contenedor) con ese mismo nombre y valor. 
 

### ¿Qué sucede si cambio los valores de alguna de las variables definidas en bloque “environment” en solo uno de los contenedores y hago que sean diferentes? (Por ej: cambio SOLO en la definición de wordpress la variable WORDPRESS_DB_NAME)

- Si cambiamos los valores de alguna de las variables de entorno (en este caso de wordpress), podría pasar que la conexión no se de.

### ¿Cómo sabe comunicarse el contenedor “wordpress” con el contenedor “db” si nunca doy información de direccionamiento?

- Wordpress se puede comunicar con mysql usando el nombre del servicio (como un alias de host).
- Docker Compose crea una red interna donde los contenedores pueden comunicarse usando los nombres de servicio como alias (Docker Compose hace la resolución de nombres).

### ¿Qué puertos expone cada contenedor según su Dockerfile? (pista: navegue el sitio https://hub.docker.com/_/wordpress y https://hub.docker.com/_/mysql para acceder a los Dockerfiles que generaron esas imágenes y responder esta pregunta.)

- Worpress: 80 (así como se declara en el `docker-compose.yml`)
- MySQL: 3306 o 33060 (como se define por defecto)

### ¿Qué servicio se “publica” para ser accedido desde el exterior y en qué puerto? ¿Es necesario publicar el otro servicio? ¿Por qué?

- Wordpress se publica para que sea accedido en el puerto 8000 del host.
- No es necesario publicar el otro, ya que simplemente será usado por el servicio wordpress (aunque si quisieramos operar con la BD deberíamos exponerlo).

## Instanciando

- `mkdir docker-compose-ej1`
- `cd docker-compose-ej1`
- `vim docker-compose.yml`
- `docker compose up`
- `docker compose up -d` -> Detached
- `docker compose exec wordpress /bin/bash` -> Conectarnos con wordpress
  - Si no funciona, usar `/bin/sh`
- `cd /data`
- `touch test && ls` -> Veremos también el archivo test en `/docker-compose-ej-1`
- `docker compose ps`
- `docker compose stop`
- `docker compose down -v`