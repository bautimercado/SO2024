# Practica 4 - Parte 2 - Docker

## 1. Utilizando sus palabras, describa qué es Docker y enumere al menos dos beneficios que encuentre para el concepto de contenedores.

- Docker es un software para crear, implementar y ejecutar aplicaciones en contenedores.
- Algunos de sus beneficios son:
  - **Portabilidad:** Los contenedores son portables y pueden ejecutarse en cualquier entorno que admita docker.
  - **Aislamiento:** Cada contenedor se ejecuta de forma aislada del resto de los contenedores y del host (más seguridad y menos conflictos).
  - **Ligereza:** Los contenedores son más livianos y eficientes que las VMs, ya que comparten el núcleo del SO.
  - **Escalabilidad:** Son escalables y fáciles de gestionar, lo que facilita el despliegue y la gestión de aplicaciones en entornos de alta demanda.
  - **Consistencia:** Garantizan que una aplicación se ejecuta de la misma manera en diferentes entornos.

## 2. ¿Qué es una imagen? ¿Y un contenedor? ¿Cuál es la principal diferencia entre ambos?

- **Imagen:** Archivo inmutable (read only) que contiene todas las instrucciones y dependencias necesarias para crear un contenedor.
  - Define el file system, binarios y librerías que se ejecutan dentro del contenedor.
  - Se crean a partir de un archivo de definición llamado `Dockerfile`.
- **Contenedor:** Instancia en ejecución de una imagen Docker. Es un entorno aislado y portátil que contiene todo lo necesario para ejecutar una aplicación (código, librerias, herramientas, configuraciones, etc.).
  - Cuando se ejecuta un contenedor a partir de una imagen, se crea una capa de lectura y escritura sobre la imagen de solo lectura.
- Entonces, una imagen es una plantilla estática e inmutable que define cómo se construye y configura un contenedor. Mientras que el contenedor es una instancia de una imagen, es un entorno aislado que puede modifcarse durante su ejecución.
  - Es como si la imagen fuera una clase en OO y el contenedor una instancia de esa clase.
  - Así, múltiples contenedores pueden crearse a partir de una imagen.

## 3. ¿Qué es Union Filesystem? ¿Cómo lo utiliza Docker?

- UnionFS es un file system que opera superponiendo varios file systems. Permite crear una vista unificada a partir de varias ramas o capas de solo lectura y potencialmente una rama de R/W.
- Docker usa UnionFS para construir y gestionar imágenes y contenedores eficientemente.
  - Cuando se construye una imagen Docker de un Dockerfile, cada instrucción crea una nueva capa que se apila sobre la anterior. UnionFS combina estas capas en una sola imagen.
  - Cuando se crea un nuevo contenedor, Docker no copia toda la imagen, sino que monta las capas de solo lectura existentes usando UnionFS. 
  - Además de las capas RO de la imagen, Docker crea una capa RW cuando se crea el contenedor. Esta capa se superpone sobre las capas RO y captura todas las modificaciones.
  - Cuando un archivo de una capa inferior necesita ser modificado, UnionFS no modifica la capa RO original, sino que copia el archivo a la capa RW y realiza los cambios ahí.

## 4. ¿Qué rango de direcciones IP utilizan los contenedores cuando se crean? ¿De dónde la obtiene?

- Los contenedores obtienen direcciones IP de un rango específico reservado para la red virtual de Docker. Por defecto, Docker usa la subred 172.17.0.0/16 (172.17.0.1 - 172.17.255.254) para asignar IPs a los contenedores.
- Este rango de direcciones IP se puede modificar mediante la configuración de red predeterminada de Docker (docker0) o a través de la creación de redes definidas por el usuario.

## 5. ¿De qué manera puede lograrse que las datos sean persistentes en Docker? ¿Qué dos maneras hay de hacerlo? ¿Cuáles son las diferencias entre ellas?

- Los datos generados por un contenedor se almacenan en el mismo contenedor y se pierden cuando el contenedor se detiene o se elimina. Pero se puede lograr la persistencia, hay dos mecanismos para hacerlo:
  - **Volumenes de datos:** Es un directorio especial dentro de uno o más contenedores diseñado para persistir datos. Los volumenes se almacenan en el file system del host Docker y se mantiene después de que el contenedor se detenga o elimine.
  - **Enlaces de datos (bind mounts):** Enlace directo a un directorio en el file system del host docker. Cuando se crea un contenedor, se puede configurar para montar un directorio del host en un path específico dentro del contenedor.
- Algunas diferencias entre ellos son:
  - **Ubicación de datos:** Los volúmenes se almacenan en un directorio administrado por Docker en el host. Los enlaces de datos apuntan a un directorio en el file system host.
  - **Gestión:** Docker gestiona los volúmenes automáticamente, los enlaces requieren una gestión manual.
  - **Rendimiento:** Los volúmenes tienen un mejor rendimiento que los enlaces (sobre todo en entornos de producción).
  - **Portabilidad:** Los volúmenes son más portátiles entre diferentes hosts Docker. Los enlaces dependen de la estructura de directorios del host.
  - **Casos de uso:** Los volúmenes son más adecuados para almacenar datos persistentes de aplicaciones. Los enlaces son útiles para montar directorios de configuración o código fuente.

# Taller

## 1. Instale Docker CE (Community Edition) en su sistema operativo. Ayuda: seguir las instrucciones de la página de Docker. La instalación más simple para distribuciones de GNU/Linux basadas en Debian es usando los repositorios.

- https://docs.docker.com/engine/install/debian/ 

## 2. Usando las herramientas (comandos) provistas por Docker realice las siguientes tareas:

### a. Obtener una imagen de la última versión de Ubuntu disponible. ¿Cuál es el tamaño en disco de la imagen obtenida? ¿Ya puede ser considerada un contenedor? ¿Qué significa lo siguiente: Using default tag: latest?

- `docker pull ubuntu:latest`
- Ver imagenes del sistema -> `docker images`
- Su tamaño es de 76.2MB
- La imagen de Ubuntu aún no es un contenedor. Pero podemos crear un contenedor a través de esta imagen.
- Using default tag: latest se usa para especificar la versión de la imagen a descargar. Si no se especifica, docker asuma la etiqueta `latest` de forma predeterminada.

### b. De la imagen obtenida en el punto anterior iniciar un contenedor que simplemente ejecute el comando ls -l.

- `docker run -it ubuntu:latest ls -l`
  - La opción `-it` es para tener una sesión interactiva y una terminal (`-t`).

![docker_run_ubuntu](/Practicas/Practica%204/img/Clipboard05.png)

### c. ¿Qué sucede si ejecuta el comando docker [container] run ubuntu /bin/bash1? ¿Puede utilizar la shell Bash del contenedor?

- No, no puedo usar la shell Bash del contenedor.

#### i. Modifique el comando utilizado para que el contenedor se inicie con una terminal interactiva y ejecutarlo. ¿Ahora puede utilizar la shell Bash del contenedor? ¿Por qué?

- `docker run -it ubuntu /bin/bash`
- La opción -i (interactive) mantiene la sesión STDIN abierta, lo que permite interactuar con el contenedor (sin esto, el contenedor ejecuta el comando y se detiene).
- La opción -t (pseudo-tty) asigna una termina al contenedor, lo que permite ver la salida del comando y enviar entrada (esto nos permite seguir interactuando con el contenedor).
- Con estas opciones es posible usar la shell Bash del contenedor.

#### ii. ¿Cuál es el PID del proceso bash en el contenedor? ¿Y fuera de éste?

- En la shell Bash del contenedor -> `echo $$`
- En otra terminal -> `ps -e`
- Dentro del contenedor, su PID es 1, fuera su PID es 4635

#### iii. Ejecutar el comando lsns. ¿Qué puede decir de los namespace?

- El contenedor comparte cgroup, user.
  - Por defecto, los contenedores comparten el namespace user con el host.

#### iv. Dentro del contenedor cree un archivo con nombre sistemas-operativos en el directorio raíz del filesystem y luego salga del contenedor (finalice la sesión de Bash utilizando las teclas Ctrl + D o el comando exit).

- `touch sistemas-operativos`

#### v. Corrobore si el archivo creado existe en el directorio raíz del sistema operativo anfitrión (host). ¿Existe? ¿Por qué?

- No, no existe.
- Esto sucede porque el archivo se almacena dentro del file system del contenedor (capa RW sobre la imagen RO). Por defecto, los contenedores no comparten file system con el host, a menos que se usen volúmenes o enlaces de datos para montar directorios del host en el contenedor.

### d. Vuelva a iniciar el contenedor anterior utilizando el mismo comando (con una terminal interactiva). ¿Existe el archivo creado en el contenedor? ¿Por qué?

- No existe. Los contenedores son efímeros y no persistentes. En cada `docker run` se crea una nueva instancia del contenedor, cualquier cambio se almacena en la capa RW y se descarta cuando el contenedor se detiene.
- Para mantener los datos persistentes en Docker es que usamos volúmenes o enlaces de datos.

### e. Obtenga el identificador del contenedor (container_id) donde se creó el archivo y utilícelo para iniciar con el comando docker start -ia container_id el contenedor en el cual se creó el archivo.

#### i. ¿Cómo obtuvo el container_id para para este comando?

- En el prompt de la shell bash de ubuntu podemos ver el container_id. Sino, podemos ejecutar `docker ps` en otra terminal.

#### ii. Chequee nuevamente si el archivo creado anteriormente existe. ¿Cuál es el resultado en este caso? ¿Puede encontrar el archivo creado?

- Si, podemos ver el archivo creado. Esto es porque reanudamos el mismo contenedor en el que creamos ese archivo.
- Docker start reinicia un contenedor existente con un estado previo, docker run crea un nuevo contenedor.

### f. ¿Cuántos contenedores están actualmente en ejecución? ¿En qué estado se encuentra cada uno de los que se han ejecutado hasta el momento?

- Con el comando `docker ps` podemos ver los contenedores ejecutandose.
- Cada uno está en estado "up" y se indica desde hace cuanto tiempo están así.
- Con el parámetro `-a` podemos ver todos.

### g. Elimine todos los contenedores creados hasta el momento. Indique el o los comandos utilizados.

- Listar contenedores -> `docker ps -a`
- Detener contenedores -> `docker stop $(docker ps -q)`
- Eliminar contenedores -> `docker rm $(docker ps -aq)`

## 3. Creación de una imagen a partir de un contenedor. Siguiendo los pasos indicados a continuación genere una imagen de Docker a partir de un contenedor:

### a. Inicie un contenedor a partir de la imagen de Ubuntu descargada anteriormente ejecutando una consola interactiva de Bash.

- `docker run -it ubuntu`

### b. Instale el servidor web Nginx, https://nginx.org/en/, en el contenedor utilizando los siguientes comandos2: `export DEBIAN_FRONTEND=noninteractive`, `export TZ=America/Buenos_Aires`, `apt update -qq` y `apt install -y --no-install-recommends nginx`

### c. Salga del contenedor y genere una imagen Docker a partir de éste. ¿Con qué nombre se genera si no se especifica uno?

- Podemos copiar el id del contenedor del prompt o a través de `docker ps`.
- `docker commit [CONTAINER_ID] [CONTAINER_NAME]`
- Podemos ver la imagen creada con `docker images`
  - Si no le asignamos un nombre, por defecto se usa `<none>`

### d. Cambie el nombre de la imagen creada de manera que en la columna Repository aparezca nginx-so y en la columna Tag aparezca v1.

- Necesitamos el id de la imagen, la podemos obtener de `docker images`.
- `docker tag [ID_IMAGEN] nginx-so:v1` 

### e. Ejecute un contenedor a partir de la imagen nginx-so:v1 que corra el servidor web nginx atendiendo conexiones en el puerto 8080 del host, y sirviendo una página web para corroborar su correcto funcionamiento. Para esto:

#### I. En el Sistema Operativo anfitrión (host) sobre el cual se ejecuta Docker crear un directorio que se utilizará para este taller. Éste puede ser el directorio nginx-so dentro de su directorio personal o cualquier otro directorio - para los fines de este enunciado haremos referencia a éste como /home/so/nginx-so, por lo que en los lugares donde se mencione esta ruta usted deberá reemplazarla por la ruta absoluta al directorio que haya decidido crear en este paso.

- `mkdir ~/nginx-so`

#### II. Dentro de ese directorio, cree un archivo llamado index.html que contenga el código HTML de este gist de GitHub: https://gist.github.com/ncuesta/5b959fce1c7d2ed4e5a06e84e5a7efc8.


#### III. Cree un contenedor a partir de la imagen nginx-so:v1 montando el directorio del host (/home/so/nginx-so) sobre el directorio /var/www/html del contenedor, mapeando el puerto 80 del contenedor al puerto 8080 del host, y ejecutando el servidor nginx en primer plano. Indique el comando utilizado.

- `docker run -it -p 8080:80 -v /home/so/nginx-so:/var/www/html nginx-so:v1 nginx -g 'daemon off;'`
  - `-p 8080:80`: Mapea el puerto 80 del contenedor al puerto 8080 del host.
  - `-v /home/so/nginx-so:/var/www/html`: Monta el directorio `/home/so/nginx` del host en el directorio `/var/www/html` dentro del contenedor.
  - `nginx -g 'daemon off;'`: Ejecutar nginx en primer plano

### f. Verifique que el contenedor esté ejecutándose correctamente abriendo un navegador web y visitando la URL http://localhost:8080.

- El contenedor se estaba ejecutando pero no podia conectarme a localhost:8080.
- Dentro del contenedor hice lo siguiente: `nginx -s reload` y me anduvo.

### g. Modifique el archivo index.html agregándole un párrafo con su nombre y número de alumno. ¿Es necesario reiniciar el contenedor para ver los cambios?

- No, no es necesario. Esto es porque los archivos servidos por NGINX dentro del contenedor están vinculados directamente al directorio `/home/so/nginx-so`

### h. Analice: ¿por qué es necesario que el proceso nginx se ejecute en primer plano? ¿Qué ocurre si lo ejecuta sin -g 'daemon off;'?

- Cuando se ejecuta un contenedor, el proceso principal debe permanecer en ejecución para que no se detenga el contenedor.
- Por defecto, nginx se ejecuta en segundo plano, si lo ejecutamos sin `-g 'daemon off;'`, el proceso se detendrá y el contenedor también.

## 4. Creación de una imagen Docker a partir de un archivo Dockerfile. Siguiendo los pasos indicados a continuación, genere una nueva imagen a partir de los pasos descritos en un Dockerfile.

### a. En el directorio del host creado en el punto anterior (/home/so/nginx-so), cree un archivo Dockerfile que realice los siguientes pasos:

#### i. Comenzar en base a la imagen oficial de Ubuntu.

#### ii. Exponer el puerto 80 del contenedor.

#### iii. Instalar el servidor web nginx.

#### iv. Copiar el archivo index.html del mismo directorio del host al directorio /var/www/html de la imagen.

#### v. Indicar el comando que se utilizará cuando se inicie un contenedor a partir de esta imagen para ejecutar el servidor nginx en primer plano: `nginx -g 'daemon off;'`. Use la forma exec4 para definir el comando, de manera que todas las señales que reciba el contenedor sean enviadas directamente al proceso de nginx. Ayuda: las instrucciones necesarias para definir los pasos en el Dockerfile son FROM, EXPOSE, RUN, COPY y CMD.

```docker
FROM ubuntu
EXPOSE 80
RUN apt-get update && apt-get install -y nginx
COPY index.html /var/www/html/
CMD ["nginx", "-g", "daemon off;"]
```
### b. Utilizando el Dockerfile que generó en el punto anterior construya una nueva imagen Docker guardándola localmente con el nombre nginx-so:v2.

- `docker build -t nginx-so:v2 .`
  - La opción `-t` (tag) es para especificar el nombre y el tag.
  - El `'.'` es para especificar que el Dockerfile está en el path actual

### c. Ejecute un contenedor a partir de la nueva imagen creada con las opciones adecuadas para que pueda acceder desde su navegador web ala página a través del puerto 8090 del host. Verifique que puede visualizar correctamente la página accediendo a http://localhost:8090.

- `docker run -it -p 8090:80 nginx-so:v2`

### d. Modifique el archivo index.html del host agregando un párrafo con la fecha actual y recargue la página en su navegador web. ¿Se ven reflejados los cambios que hizo en el archivo? ¿Por qué?

- No, los cambios no se ven reflejados.
- Esto es porque cuando se construye una imagen Docker, todos los archivos se copian dentro de la imagen en el momento de la construcción, cualquier cambio posterior en los archivos del host no se reflejarán automáticamente en el contenedor en ejecución.

### e. Termine el contenedor iniciado antes y cree uno nuevo utilizando el mismo comando. Recargue la página en su navegador web. ¿Se ven ahora reflejados los cambios realizados en el archivo HTML? ¿Por qué?

- Tampoco, siguen sin aparecer los cambios.
- Esto sucede porque de nuevo, cambiamos el archivo luego de construir la imagen Docker, para que se vean reflejados los cambios deberíamos re-construir nuevamente la imagen.

### f. Vuelva a construir una imagen Docker a partir del Dockerfile creado anteriormente, pero esta vez dándole el nombre nginx-so:v3. Cree un contenedor a partir de ésta y acceda a la página en su navegador web. ¿Se ven reflejados los cambios realizados en el archivo HTML? ¿Por qué?

- `docker build -t nginx-so:v3 .`
- `docker run -it -p 8090:80 nginx-so:v3`
- Ahora si se ven reflejados, esto es porque se creó una nueva imagen con el archivo modificado, y el contenedor usa una copia de ese archivo modificado incluido en la nueva imagen.