# Practica 3

## File Systems

### 1. ¿Qué es un file system?

- Es una forma de organizar y almacenar datos en un disco (HDD, unidad flash, tarjeta de memoria, etc.).
- Determina cómo se divide el disco en sectores, clusters y particiones, cómo se nombran y ubican los archivos y carpetas, y cómo interactúa el disco con el SO y otros dispositivos.

### 2. Describa las principales diferencias y similitudes entre los file systems: ext(2,3,4) y XFS.

- extN usan una estructura jerárquica de árbol, mientras que XFS usa una estructura de extensión de diseño más moderno.
- Ambos usan journaling para mejorar la integridad, pero XFS usa un enfoque de journaling más avanzado llamado _journaling de extensión_.
- XFS se desgragmenta automáticamente, los sistemas extN pueden requerir desfragmentación manual.
- XFS soporta volúmenes más grandes que Ext4 (16EiB vs 1 EiB).
- XFS tiene mejor rendimiento para operaciones I/O intensivas, mientras que Ext4 es mejor para operaciones de metadatos.
- Ambos soportan características avanzadas como extents, asignación de múltiples bloques y asignación retrasada.
- Ambos son file systems a que admiten permisos y atributos estándares de Unix (owner, group y permisos rwx).
- Ambos son compatibles con la mayoría de distribuciones GNU/Linux y otros SO tipo Unix.

### 3. En ext4, describa las siguientes características: extents, multiblock allocation, delay allocation y persistent pre-allocation (https://kernelnewbies.org/Ext4).

- **Extents:** Son una forma eficiente de almacenar los bloques de datos de un archivo en el disco. En lugar de tener una lista de bloques no contiguos para un archivo, los extents permiten agrupar bloques contiguos en un solo registro, lo cuál reduce el overhead de gestión de archivos y mejora el rendimiento al acceder a grandes archivos.
- **Multiblock Allocation:** Permite asignar múltiples bloques contiguos de espacio en disco a un archivo al mismo tiempo. En lugar de asignar bloques uno por uno, ext4 puede asignar varios extents de bloques contiguos simultáneamente. Esto reduce la fragmentación y mejora el rendimiento, especialmente para operaciones de escritura de archivos grandes.
- **Delayed Allocation:** Es una técnica que retrasa la asignación real de bloques en disco hasta que los datos se escriben. Cuando se crea un archivo nuevo o se extiende uno existente, ex4 no asigna bloques de inmediato, sino que reserva espacio en el registro de transacciones (journal) y aplaza la asignación real de bloques hasta se escriben los datos. Esto mejora el rendimiento al reducir la fragmentación y operaciones I/O innecesarias.
- **Persistent Preallocation:** Reserva espacio en disco para un archivo de antemano. Cuando se crea un archivo, se puede especificar un tamaño objetivo y ext4 reserva ese espacio continuo. Esto evita la fragmentación y mejora el rendimiento.

### 4. ¿Es siempre necesario tener un file system para acceder a un disco o partición?

- No siempre. En GNU/Linux podemos acceder directamente a un dispositivo de bloque sin un file system usando el comando `dd`.
- Acceder a un disco de manera práctica y organizada, normalmente requiere formatear el disco con un file system.

### 5. ¿Qué es el área de swap en Linux? ¿Existe un área similar en Windows?

- El área de swap es una partición usada como memoria virtual cuando la RAM se llena. Cuando un proceso necesita más memoria de la disponible, algunas páginas de memoria se mueven a la swap para liberar espacio en RAM.
- Windows no tiene un área de swap, pero usa un archivo de paginación `pagefile.sys` cuyo propósito es similar 

### 6. ¿Qué función cumple el directorio lost+found en Linux?

- Es usado para recuperar archivos huérfanos después de un error del sistema o un desmontaje forzado.
- El file system coloca cualquier archivo recuperado en este directorio durante el proceso de verificación y reparación fcsk. 

### 7. En Linux, ¿dónde se almacenan el nombre y los metadatos de los archivos?

- El nombre y los metadatos de los archivos se almacenan en los inodos, que son estructuras de datos que contienen información sobre un archivo como permisos, propietario, timestamps y punteros a los bloques de datos. Los inodos se almacenan en una tabla de inodos.

### 8. Seleccione una de sus particiones y conteste usando el comando dumpe2fs (si está usando la MV de la cátedra, como root, dumpe2fs /dev/sda1):

- Importante: En mi caso, tuve que ejecutar el comando así -> `/usr/sbin/dumpe2fs`

#### ¿Qué información muestra el comando dumpe2fs?

- Cuando se ejecuta `dumpe2fs` en un dispositivo que tiene un file system extX, nos da la siguiente información:
  - **Información del súper bloque:** Muestra detalles del súper bloque, cómo el número de bloques, tamaño de bloques, número de indodos, etc.
  - **Información del grupo de bloques:** Brinda detalles sobre los grupos de bloques, incluyendo la cantidad de bloques usados, libres, inodos usados y libres en cada grupos.
  - **Información del directorio raiz:** Muestra los metadatos del inodo del directorio raíz, como permisos, owner, timestamps, etc.
  - **Información del journal:** Si el file system tiene journal (ext3 y ext4), muestra detalles sobre el dispositivo de journal, su tamaño, tipo y estado.
  - **Características del file system:** Lista las características habilitadas en el file system, como extents, sparse_super, dir_nlink, etc.
  - **Estadísticas del file system:** Proporciona estadísticas sobre el uso del file systems, como la cantidad de inodos usados, directorios, enlaces simbólicos, archivos regulares, etc.
  - **Información de la línea de montaje y UUID:** Muestra la línea de montaje guardada en el súper bloque y el UUID del file system. 
  - **Resumen del grupo de bloques:** Presenta un resumen del uso de bloques e inodos de cada grupo de bloques.

#### ¿Cuál es el tamaño de bloque del file system?

- Si ejecutamos `dumpe2fs` por si solo, podemos ver que en la línea:

```bash
...
Block size: 4096
...
```

- También podemos usar la opción `-h` para ver la información del super bloque (nos da menos info).
- Aunque también podemos usar un pipeline y `grep` para reducir la información

#### ¿Cuántos inodos en total contiene el file system? ¿Cuántos archivos más se podrían crear con el estado actual del file system?

- `/usr/sbin/dumpe2fs /dev/sda1 | grep inode`
- En total hay 2445984 inodos.
- Tenemos 2381753 inodos libres, por lo tanto, podemos crear 2381753 archivos.

#### ¿Cuántos grupos de bloques existen?

- `/usr/sbin/dumpe2fs /dev/sda1 | grep blocks per group`

#### ¿Cómo haría para incrementar la cantidad de inodos de un file system?

- Para incrementar la cantidad de inodos en un file system extX, es necesario reformatear el dispositivo.
- Podríamos añadir más dispositivos de almacenamiento, usar algún file system que permita aumentar dinámicamente los metadatos (XFS o Btrfs).
- Cuando se crea un nuevo file system ext4, podemos especificar la cantidad de nodos usando la opción `-N`. Por ejemplo:

```bash
mkfs.ext4 -N 10000000 /dev/sdb1
```

### 9. ¿Qué es el file system procfs? ¿Y el sysfs?

- **procsfs (Process File System)**
  - Es un file system virtual que brinda información sobre los procesos en ejecución.
  - Cada proceso tiene un directorio en `/proc`, con archivos que tienen información como la memoria usada, estado del proceso, archivo abiertos, etc.
  - También nos da información sobre el kernel, su versión, los parámetros de arranque, información de CPU, etc.
  - Los archivos en `/proc` no existen en el disco, son generados dinámicamente por el kernel.
- **sysfs (System File System)**
  - File system virtual que expone información sobre los dispositivos y drivers del kernel.
  - Generalmente se monta en el directorio `/sys`.
  - Brinda una visión jerárquica de los dispositivos y subsistemas del kernel, incluyendo información del hardware, drivers, buses, dispositivos, etc.
  - Permite la interacción con el kernel y los dispositivos a través de los archivos y directorios en `/sys`.
  - Por ejemplo, en `/sys/devices` se puede encontrar información sobre los dispositivos conectados, como tarjetass PCI, USB, CPU, etc.
  - Al igual que `procfs`, no almacena datos en el disco.

### 10. Usando el directorio /proc, contestar:

#### ¿Cuál es la versión de SO que tiene instalado?

```bash
cat /proc/version
```

- Linux version 4.19.0-19-amd64

#### ¿Cuál es procesador de su máquina?

```bash
cat /proc/cpuinfo
```

- Intel(R) Core(TM) i5-8250U

#### ¿Cuánta memoria RAM disponible tiene?

```bash
cat /proc/meminfo
```

- En concreto la línea "MemAvailable:"
- 797360 kB

#### ¿Qué archivo debería consultar si se quiere ver el mismo resultado que el comando lsmod?

- `lsmod` muestra un listado de los módulos del kernel cargados actualmente.

```bash
cat /proc/modules
```

### 11. Usando el comando stat, contestar:

#### ¿Cuándo fue la última vez que se modificó el archivo /etc/group?

```bash
stat /etc/group
```

- 2022-03-20 20:09:30.652632460 -0300

#### ¿Cuál es la diferencia entre los datos Cambio (Change) y Modificación (Modify)?

- Modify se refiere a la última vez que se modificó el contenido del archivo.
- Change se refiere a la última vez que se modificaron los metadatos del archivo.

#### ¿Cuál es el inodo que ocupa? ¿Cuántos bloques ocupa?

- I-Nodo: 1314367
- Bloques: 8

#### ¿Qué número de inodo ocupa el directorio raíz?

- I-Nodo: 2

#### ¿Es posible conocer la fecha de creación de un file en ext4? ¿Cómo lo haría?

- No, ya que ext4 no guarda la fecha de creación de un archivo como metadato. Solo almacena las fechas de última modificación, último cambio y último acceso.

### 12. ¿Qué es un link simbólico? ¿En qué se diferencia de un hard-link?

- Un symbolic link (o soft link) es un tipo especial de archivo que tiene una ruta a otro archivo o directorio. Al acceder al link simbólico, el SO redirige la operación al archivo o directorio apuntado por el link.
- Un hard-link es una entrada adicional en el SO que apunta directamente al mismo inodo (y al mismo contenido) que el archivo original. Es como si el archivo tuviera dos nombres diferentes.
- Un link simbólico solo tiene una ruta, mientras que un hard-link apunta al inodo del archivo.
- Un link simbólico puede apuntar a un archivo o directorio en cualquier parte del file system, mientras que un hard-link debe estar en el mismo file system que el archivo original.
- Si se elimina el archivo original, el link simbólico se "rompe" y apunta a un archivo inexistente, mientras que el hard-link sigue siendo accesible y apuntando al mismo contenido.
- Los hard-link no pueden apuntar a directorios.

### 13. Al crear un hard-link, ¿se ocupa un nuevo inodo? ¿Y con un link simbólico?

- Al crear un hard-link no se crea un nuevo inodo.
- Al crear un link simbólico si se ocupa un nuevo inodo.

### 14. Si se tiene un archivo llamado prueba.txt y se le genera un link simbólico, ¿qué sucede con el link simbólico si se elimina el archivo prueba.txt? ¿Y si el link fuese hard-link? (Ver el comando ln para la creación de links)

- Si se genera un link simbólico al archivo `prueba.txt` y después se elimina, el link simbólico se "rompe" y apunta a un archivo inexistente. Al intentar acceder al link simbólico, se obtendrá un error indicando que el archivo no existe.
- Si se genera un hard-link para `prueba.txt` y lo eliminamos, el hard-link sigue siendo accesible y apunta al mismo contenido (porque apunta al inodo del archivo).

### 15. Crear un archivo llamado prueba2.txt. Si ahora se genera un hard-link sobre ese archivo llamado pruebahd.txt, ¿cómo se refleja la creación de ese hard-link?

```bash
$ echo "Hola, mundo" > prueba2.txt
$ ln prueba2.txt pruebahd.txt
```

- La creación del hard-link no se reflejará en el contenido del archivo, pero sí en el número de hard links que apuntan a ese inodo (lo podemos ver con `stat`).

### 16. Elimine el archivo prueba2.txt, ¿es posible acceder al archivo pruebahd.txt? ¿Cómo se refleja la eliminación de ese archivo?

- Si, es posible acceder al archivo `pruebahd.txt` después de eliminar `prueba2.txt`, ya que ambos son hard-links que apuntan al mismo inodo.
- Si ejecutamos `stat ~/pruebahd.txt` podemos ver que el campo "Links" ahora tiene 1.

## RAID

### 1. ¿Qué es un RAID? Explique las diferencias entre los distintos niveles de RAID

- RAID (Redundat Array of Independent Disks) es una tecnología que combina múltiples discos para brindar redundancia de datos, mejora del rendimiento o ambas cosas. Existen varios niveles de RAID, cada uno con sus propias características y ventajas.

1. RAID 0 (Striping): No brinda redundancia, distribuye los datos en múltiples discos para aumentar el rendimiento de R/W. Si un disco falla, se pierden todos los datos.
2. RAID 1 (Mirroring): Los datos se escriben de forma idéntica en dos o más discos. Brinda redundancia pero no mejora el rendimiento. Si un disco falla, los datos permanecen intactos en los discos restantes.
3. RAID 5 (Striping con paridad distribuida): Los datos se distribuyen en tres o más discos, y la información de paridad se distribuye también en los discos. Tolera la falla de un disco y ofrece un buen equilibrio entre rendimiento y redundancia.
4. RAID 6 (Striping con doble paridad distribuida): Similar a RAID 5, pero con dos conjuntos de paridad, lo que permite tolerar la falla de hasta dos discos.
5. RAID 10 (Mirroring y Striping): Combina RAID 0 y 1. Los datos se escriben en matrices de mirroring (RAID 1) y luego se implementa el striping (RAID 0) en estas matrices. Ofrece redundancia y mejora el rendimiento (requiere más discos).

### 2. Usando el comando fdisk (fdisk /dev/sda) crear una nueva partición de tipo extendida, si es que no existe previamente. Para evitar tener que crearla nuevamente crear esta partición con el tamaño máximo posible.

- `/sbin/fdisk /dev/sda`
  - `p` -> Ver tabla de particiones
  - `n` -> Crear nueva partición

### 3. Dentro de esta partición extendida crear 3 nuevas particiones de 300MB cada una. Para esto utilizar nuevamente el comando fdisk, pero ahora las particiones deben ser de tipo logical. Reiniciar.

- `/sbin/fdisk /dev/sda`
  - `n` -> Crear partición lógica.
  - En mi caso, cree la partición del punto anterior con todo el espacio.
  - Por lo tanto, solo podemos crear particiones lógicas sobre la partición física.
  - Nos da el número de partición por defecto, para la primera dirección de cilindro la dejamos así, para la última le ponemos +300M.
  - Repito 3 veces esto para hacer las 3 particiones.

### 4. Utilizar el comando mdadm para crear un RAID 5 utilizando las 3 particiones lógicas que se generaron en el punto anterior (fdisk -l para ver el nombre de las particiones que generaron): `mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sda5 /dev/sda6 /dev/sda7` (Obs.: md0 es el nombre que le dará al nuevo RAID)

![captura_mdadm_create](/Practicas/Practica%203/img/Clipboard01.png)

### 5. ¿Qué significan los valores sda5, sda6 y sda7?

- Los valores sda5, sda6 y sda7 son las particiones lógicas creadas dentro de la partición extendida.
- La nomenclatura sda se refiere al primer disco duro (a) y los números se asignan secuencialmente a las particiones.

### 6. Ejecutar la siguiente consulta y contestar: mdadm --detail /dev/md0

#### a) ¿Cuál es el tamaño del RAID?

- El tamaño del RAID se indica en "Array Size". En mi caso, su tamaño es de 596MiB.

#### b) ¿Qué significa Used Dev Size?

- Se refiere al tamaño de cada uno de los dispositivos individuales que componen el RAID.
- En RAID 5, el tamaño del RAID es ligeramente menor que la suma del tamaño de los dispositivos/particiones.


### 7. Analizar el contenido del siguiente comando: `cat /proc/mdstat`

![captura_mdstat](/Practicas/Practica%203/img/Clipboard03.png)

- La salida nos muestra los niveles de RAID soportados por el kernel.
- Después nos muestra el RAID que tenemos (nombre, estado, nivel y dispositivos).
- Después nos muestra datos como el tamaño total del RAID en bloques, nivel de RAID, tamaño de chunk, algoritmo de paridad usado, dispositivos presentes y el estado de cada uno (cada uno en U que significa UP o activo).

### 8. Ahora se va a probar la funcionalidad del RAID 5. Para esto completar los siguientes pasos:

#### a) Crear un file system de tipo ext4 en el RAID 5 recién generado `mkfs.ext4 /dev/md0`

- `/sbin/mkfs.ext4 /dev/md0`

#### b) Montar la partición con el file system generado en el directorio /mnt/rd5

- `mkdir /mnt/rd5`
- `mount /dev/md0 /mnt/rd5`

#### c) Crear un directorio con dos archivos

- `cd /mnt/rd5`
- `mkdir test && touch file1 file2`

#### d) Quitar una de las particiones del RAID. Para esto ponemos uno de los componentes en falla: mdadm --fail /dev/md0 /dev/sda7

![captura_mdadm_fail](/Practicas/Practica%203/img/Clipboard04.png)

#### e) Observar el estado del RAID y contestar

##### 1) ¿Cuál es el estado del RAID? ¿Cuántos dispositivos activos existen?

- El estado del RAID muestra que uno de los dispositivos falló `[UU_]`.

###### 2) El tamaño del RAID, ¿se modificó?

- No. El tamaño del RAID no se modificó.

###### 3) ¿Qué sucedería si se ejecuta el comando anterior sobre una de las particiones restantes?

- El RAID entraría en estado "inactivo" y se perderían todos los datos, ya que no quedaría ningún dispositivo activo.

#### f) Quitar del RAID el componente puesto en falla en el paso anterior `mdadm --remove /dev/md0 /dev/sda7`


#### g) Observar nuevamente el estado del RAID y contestar:

##### 1) ¿Se puede acceder al directorio /mnt/rd5? ¿Están los archivos creados anteriormente?

- Si se puede, ya que RAID 5 puede tolerar la falla de un dispositivo.

##### 2) ¿Qué hubiese sucedido si teníamos otra partición como “hot-spare”?

- Se habría activado automáticamente para reemplazar al dispositivo fallido, reconstruyendo los datos en ella.

#### h) Por último, remover la partición permanentemente del RAID (Obs.: esto es muy importante para que el próximo booteo mdadm no intente usar a esta partición como parte del RAID, lo que provocaría la pérdida de todos los datos) `mdadm --zero-superblock /dev/sda7` A partir de este momento la partición /dev/sda7 se puede utilizar como una partición común


### 9. Para evitar la pérdida de datos es fundamental volver al RAID a un estado estable (sacarlo del estado degradado). Para esto se agregará nuevamente la partición /dev/sda7 que se quitó en el paso anterior

#### a) Ejecutar el comando mdadm –add /dev/md0 /dev/sda7

#### b) Ejecutar el comando mdadm –detail /dev/md0

#### c) ¿Qué hace el RAID con la nueva partición recientemente agregada? ¿Qué significa el estado “Rebuild Status”?

- Cuando se agrega una nueva partición a un RAID degradado, el RAID comienza a reconstruir los datos faltantes en la nueva partición. Este proceso se conoce como "rebuild".
- El estado "Rebuild Status" indica el progreso de esta reconstrucción. Durante este estado, el RAID sigue estando operativo y accesible, pero puede haber un rendimiento menor.

#### d) ¿Es posible ingresar al recurso /mnt/rd5? ¿Se encuentran disponibles los datos creados en el punto anterior?

- Si, es posible ingresar a `/mnt/rd5` y a los datos creados. RAID 5 está diseñado para brindar redundancia y tolerancia a fallas, lo que permite acceder a los datos incluso cuando un dispositivo está fallando.

### 10. Como los datos que mantiene el RAID son muy importantes es necesario tener un disco (partición en nuestro ejemplo) de respaldo. Para esto se va a agregar una partición como “hot-spare”.

### 11. Usando el comando fdisk o parted generar una nueva partición, /dev/sda8, con igual tamaño a las anteriores.


### 12. Agregar la nueva partición al RAID: `mdadm --add /dev/md0 /dev/sda8`

#### a) ¿Cómo se agregó la nueva partición? ¿Por qué?

- La nueva partición se agregó como **spare** ya con `--add` se agergan como un disco de respaldo (salvo que se diga lo contratio).

### 13. Volver a poner en falla a la partición /dev/sda7. Ver el estado del RAID y contestar `mdadm --detail /dev/mda0`

- `/sbin/mdadm --fail /dev/md0 /dev/sda7`

#### a) ¿Qué hace el RAID con la partición que estaba como spare?

- El RAID usa automáticamente la partición que estaba configurada como spare para reemplazar la partición fallida /dev/sda7.

### 14. Por último, se eliminará el RAID creado en los pasos anteriores:

#### a) Desmontar el RAID (comando umount)

- `umount /mnt/rd5` 

#### b) Para cada una de las particiones del RAID ejecutar los pasos realizados cuando se quitó una partición del RAID (mdadm con las opciones -–fail y –remove). Por cada partición que se quita ir mirando el estado del RAID para ver como se comporta

- Se puede ver como el RAID se va quedando sin dispositivos disponibles (en un momento se usa el sda8 para restaurar).

#### c) Remover los superbloques de cada una de las particiones `mdadm --zero-superblock /dev/sda5 /dev/sda6 /dev/sda7`


#### d) Remover el RAID `mdadm --remove /dev/md0` Obs.: si existe, quitar la ĺinea ARRAY... del archivo /etc/mdadm/mdadm.conf

#### e) Reiniciar y comprobar que el RAID ya no existe

## LVM (Logical Volumen Managment)

### 1. ¿Qué es LVM? ¿Qué ventajas presenta sobre el particionado tradicional de Linux?

- Es un mecanismo de gestión de discos y particiones en GNU/Linux, ofrece una capa de abstracción sobre las particiones físicas. En vez de operar directamente con particiones, trabaja con volúmenes lógicos (que pueden abarcar varios discos físicos).
- Algunas de sus ventajas son:
  - **Flexibilidad:** Los volúmenes lógicos pueden redimensionarse sin reiniciar el sistema.
  - **Aprovechamiento del espacio:** Se puede combinar el espacio libre de varios discos en un único volumen lógico.
  - **Snapshots:** LVM permite crear snapshots de los volúmenes lógicos.
  - **Redundancia:** Facilita la implementación de RAID a nivel lógico.

### 2. ¿Cómo funcionan los “snapshots” en LVM?

- Permiten crear una imagen de una partición lógica en un momento determinado. Son rápidos, eficientes en cuando a espacio y sirven como copia de seguridad.
  - Al crear un snapshot, LVM reserva un espacio de almacenamiento para registrar los cambios posteriores en los datos de la partición lógica original.
  - El snapshot inicial tiene una copia de los datos de la partición original.
  - Cuando se modifica un bloque de datos en la partición, su contenido original se copia en el espacio reservado para el snapshot <u>antes de aplicar los cambios.</u>
  - De esta forma, el snapshot mantiene una vista estática de los datos originales, mientras que la partición continúa actualizándose.
  - Los snapshots son de sólo lectura y permiten realizar copias de seguridad consistentes, probar actualizaciones, etc.

### 3. Instalar la herramienta lvm2: `apt-get install lvm2`

### 4. Ejecutar el siguiente comando, ¿qué es lo que realiza? `pvcreate /dev/sda5 /dev/sda6`

- Se usa en LVM para inicializar un dispositivo (comúnmente una partición) y convertirlo en un volúmen físico (componente básico de LVM).
- Después de convertir las particiones `/dev/sda5` y `/dev/sda6` en PV, podemos asignarlas a un volume group y crear volúmenes lógicos.

### 5. Mediante el comando pvdisplay observar el estado del volumen físico recientemente creado

![captura_pvdisplay](/Practicas/Practica%203/img/Clipboard05.png)

### 6. Crear un grupo de volúmenes (volume group, VG) llamado “so”. `vgcreate so /dev/sda5 /dev/sd6`

### 7. Utilizar el comando `vgdisplay` para ver el estado del VG ¿Cuál es tamaño total del VG? ¿Qué significa PE? ¿Qué define?

- El tamaño total del VG es de 592 MiB (VG Size).
- PE (Physical Extents) es una unidad de asignación de espacio dentro del VG. Cuando creamos el VG, LVM divide los PV (`/dev/sda5` y `/dev/sda6`) en unidades más pequeñas llamadas PE. Es el mismo para todos los PV dentro de ese VG.
  - Total PE: Número total de PE disponibles en el VG.
  - Alloc PE: Número de PE asignados a los LV dentro del VG.
  - Free PE: Número de PE no asignados y disponibles.
  - Size PE: Tamaño en bytes de cada PE.

### 8. Crear dos volúmenes lógicos (logical volume, LV) de 8MB y 117MB respectivamente `lvcreate -l 2 -n lv_vol1 so` y `lvcreate -L 117M -n lv_vol2 so`

### 9. ¿Cuál es la diferencia entre los dos comandos utilizados en el punto anterior?

- En el 1er comando se asignan 2 LE (Logical Extents). El tamaño de `lv_vol1` depende del tamaño de las LE en ese VG.
- En el 2do comando se especifica el tamaño exacto del nuevo LV. LVM calcula automáticamente el número de LE necesarias para crear un LV de ese tamaño.

### 10. ¿Con qué tamaño se generó el LV lv_vol2? ¿Por qué?

- LVM asigna espacio en unidades PE. Cada PE tiene un tamaño fijo, determinado por la configuración del VG. El tamaño solicitado para un LV se redondea al múltiplo más cercano de PE.

### 11. Formatear los dos LV generados en el paso anterior con un file system de tipo ext4: `mkfs.ext4 /dev/so/lv_vol1`

### 12. Crear dos directorios, vol1 y vol2, dentro de /mnt y montar ambos LVs en estos directorios (montar el LV lv_vol1 en el directorio vol1 y lv_vol2 en vol2)

- `mkdir /mnt/vol1 /mnt/vol2`
- `mount /dev/so/lv_vol1 /mnt/vol1`
- `mount /dev/so/lv_vol2 /mnt/vol2`

### 13. Ejecutar el comando proof (Puede tomar un rato su ejecución. Este comando estará disponible en la plataforma y deberán copiarlo a la VM)

### 14. Crear un nuevo archivo en /mnt/vol1. ¿Es posible? ¿Por qué? ¿Qué debería hacerse para solucionarlo?

- No es posible, ya que el script proof crea n archivos vacíos (siendo n la cantidad de inodos de `/dev/so/lv_vol1`) dejando sin posibilidad de crear nuevos archivos.
- Para solucionarlo, podríamos (por ejemplo) borrar todos los archivos vacios creados o ampliar el tamaño del LV.

### 15. Para aplicar lo solución propuesta en el punto anterior, para esto primero se debe incrementar el tamaño del LV correspondiente:

#### a) Extender el LV lv_vol1 en 20M `lvextend -L +20M /dev/so/lv_vol1`

#### b) Ejecute el comando `df -h`, ¿se refleja en la salida del comando el incremento del espacio? ¿Por qué?

- No, no se refleja el incremento del espacio.
- Esto es así porque `lvextend` solo modifica el tamaño del LV dentro del VG, pero no realiza ningún cambio en el filesystem montado en ese LV.

#### c) Incrementar el tamaño del file system: `resize2fs /dev/so/lv_vol1`

### 16. Después de la operación previa, ¿siguen estando los datos disponibles?

- Si, los datos siguen estando disponible.
- `resize2fs` se encarga de redimensionar el tamaño del filesystem para que ocupe todo el espacio disponible en el LV, pero no midifica ni elimina el contenido existente.

### 17. Intentar crear un nuevo archivo en /mnt/vol1. ¿Es posible? ¿Por qué?

- Ahora si es posible ya que extendimos el espacio del LV (más inodos).

### 18. Se desea crear un nuevo LV de 500M. ¿Hay suficiente espacio? ¿Cómo lo solucionaría?

- No, no tenemos el espacio suficiente ya que nos quedan solo 333MiB libres (lo podemos ver con el comando `vgdisplay`).
- Una solución podría ser agregar un nuevo PV al VG.

### 19. Para aplicar la solución indicada en el punto anterior, realizar lo siguiente: `pvcreate /dev/sda7` y `vgextend so /dev/sda7`


### 20. Comprobar con los comando correspondientes que se haya extendido el tamaño del VG

- Usando `vgdisplay` podemos ver que ahora tenemos 555MiB de espacio libre, por lo tanto podemos crear un nuevo LV de 500MB.

### 21. Generar el nuevo LV de 500M (llamarlo lv_vol3) A continuación se mostrará el funcionamiento de los snapshot en LVM

- `/sbin/lvcreate -L 500M -n lv_vol3 so`

### 22. Generar un LV de 100M, nombrarlo lv1, (o usar uno de los generados en pasos anteriores). Montarlo en el directorio /dir1

- En mi caso voy a usar el `lv_vol3`.
- `mkfs.ext4 /dev/so/lv_vol3`
- `mkdir /dir1 && mount /dev/so/lv_vol3 /dir1`

### 23. Copiar desde /etc todo los archivos y directorios que comiencen con la letra a, b, c y d.

- `cp -r /etc/[a-d]* /dir1`

### 24. Mediante el siguiente comando generar un snapshot del LV anterior `lvcreate -L 30M -s /dev/so/lv1 -n lvcopy` (-s indica que este LV será un snapshot)


### 25. Verificar la creación del snapshot con el comando lvs. Montarlo en el directorio /snap. ¿Qué contenido tiene en el snapshot?

![captura_lvs](/Practicas/Practica%203/img/Clipboard06.png)
- `mkdir snap && mount /dev/so/lvcopy /snap`
- El snapshot tiene el mismo contenido que su LV (todo el contenido que copiamos de `/etc` que empiece con 'a', 'b', 'c' o 'd').

### 26. ¿Cuánto espacio hay consumido en el snapshot creado? ¿Por qué sucede esto?

- `lvdisplay /dev/so/lvcopy`
- El espacio consumido por el snapshot es de aprox. 205KiB.
- Cuando se crea un snapshot, LVM no crea una copia completa, sino que reserva espacio de almacenamiento y usa un mecanismo COW (copy-on-write)
  - Al principio el snapshot apunta a los mismos bloques de datos que el LV.
  - Cuando se modifica un bloque de datos en el LV origianl, se copia el contenido original de ese bloque en el espacio reservado para el snapshot antes de aplicar los cambios.
  - Así, el snapshot mantiene una vista estática de los datos originales.

### 27. Para probar el snapshot, elimine una carpeta del LV original (por ej, la carpeta apt). ¿Se eliminó en el LV original? ¿Y qué sucedió en el snapshot?

- `rm -r /dir1/apt`
- Se elimino del LV original pero el directorio existe en el snapshot.

### 28. Si se desea volver el LV a su estado original se debe hacer un “merge” entre el LV y su snapshot. Para esto primero deben desmontar el LV original y su snapshot correspondiente. Luego, realizar el “merge” de ambos LVs: `lvconvert --merge /dev/so/lvcopy`

- `umount /dev/so/lv_vol3`
- `umount /dev/so/lvcopy`

### 29. Comprobar si el LV original contiene nuevamente los datos eliminados anteriormente (Deberá montarlo nuevamente)

- `mount /dev/so/lv_vol3 /dir1`
- Si, el LV original contiene los datos borrados.

### 30. ¿Qué sucedió con el snapshot? Obs.: en caso que aparezca el error “Can’t merge over...” ejecutar los siguientes comandos para desactivar y activar el LV `lvchange -an /dev/so/lv1` y `lvchange -ay /dev/so/lv1`

- El snapshot se descarta y deja de existir. El contenido del LV original se mergea con el del snapshot. El espacio ocupado por el snapshot es liberado y queda disponible para ser reutilizado dentro del VG.


## BTRFS & ZFS

### 1. Tanto para BTRFS como para ZFS, responder: ¿Cuál es el significado de las siglas? ¿Quién los creó? ¿Cuál es su modo de licenciamiento? ¿Cuáles son las características más importante de cada uno? Investigar qué es la técnica copy-on-write

- BTRFS (B-Tree File System o Better FS):
  - Creado por Oracle y mantenido principalmente por desarrolladores de GNU/Linux.
  - Esta licenciado bajo la GPL.
  - Algunas de sus características más importantes son:
    - Provee un filesystem COW.
    - Tiene verificación de datos integrada mediante checksum.
    - Subvolúmenes.
    - Snapshots y replicación de datos.
    - Compresión transparente.
    - Desfragmentación en línea.
    - Equilibrio de carga de E/S.
- ZFS (Zettabye File System):
  - Crea por Sun Microsystem y mantenido por OpenZFS (Oracle, FreeBSD, Linux, etc.).
  - Esta livenciado bajo la licencia de código abierto CDDL.
  - Algunas de sus características más importantes son:
    - Filesystem COW.
    - Snapshots y replicación de datos.
    - Protección de datos mediante checksum y RAID.
    - Envío de datos y replicación remota.
    - Administración de volúmenes lógicos.
    - Compresión y desduplicaión de datos.
- La técnica COW implica que cuando se modifica un bloque, en lugar de sobreescribir el bloque original, se crea una nueva copia del bloque con los cambios.
  - Como los bloques de datos no se modifican, hay una copia coherente de los datos antiguos, lo que permite crear snapshots y reversionar estados anteriores.
  - La escritura de un nuevo bloque puede ser más eficiente que la sobreescritura (más cuando los datos son muchos).
  - Simplifica operaciones de gestión de datos, como crear snapshots, replicar datos, etc.
  - También implica cierto desperdicio de espacio y tener un proceso que elimine bloques de datos viejos no usados.

### 2. Generar en la MV dos particiones de 3GB cada una. Crear 3 nuevos directorios llamados /disco5, /volumen1 y /volumen2.

- `fdisk /dev/sda`
  - Crear partición -> `n`
  - Dejar primer sector como default
  - Para el último sector ponerle +3G
  - Repertir 2 veces más
  - Escribir los cambios -> `w`
  - `/dev/sda9`, `/dev/sda10` y `/dev/sda11`
- `mkdir /disco5 /volumen1 /volumen2`

### 3. Tomar una partición, /dev/sdaX (X=número de una de las particiones creadas en el punto anterior), y crearle un file system de tipo BTRFS. Montarla en el directorio disco5. (Sino están instalados los comandos para BTRFS: `apt-get install btrfs-progs`)

- `mkfs.btrfs /dev/sda9`

![captura_mkfs.btrfs](/Practicas/Practica%203/img/Clipboard07.png)

- `mount /dev/sda9 /disco5`

### 4. Por defecto, BTRFS, ¿replica los datos? ¿Y los metadatos? ¿Es posible modificar esto? ¿Cómo lo haría? (Hint: usar `btrfs fi df -h /disco5` o `btrfs df usage /disco5`)

- No, por defecto no replica ni datos ni metadatos.
- Es posible configurar RAID al crear un filesystem BTRFS, lo que permite replicar datos entre múltiples dispositivos.

![captura_btrfs_fi_df](/Practicas/Practica%203/img/Clipboard08.png)

### 5. ¿Cuál es el espacio alocado? ¿Y el ocupado realmente? Utilice los comandos `df -h`, `btrfs fi show`, `btrfs fi df -h /disco5` y `btrfs df usage /disco5`.

- `df -h` me dice que el espacio usado es de 17MB y `btrfs fi show` me dice que el espacio usado es de 331.12MB.
  - `df -h` me muestra el espacio ocupado por archivos y directorios.
  - `btrfs fi show` me muestra no solo el espacio de datos, sino también el espacio de metadatos, snapshots, copias redundantes (si se usa RAID), reservas del sistema, etc.
  - Esto sucede porque BTRFS usa COW.

### 6. Generar un archivo de 2000MB en el directorio /disco5 (dd if=/dev/zero of=/disco5/so1 bs=100M count=20). Analizar nuevamente el espacio alocado/ocupado. (Obs.: puede que tenga que esperar un tiempo para ver los cambios en la salida de los comandos btrfs....). ¿Cómo quedó el espacio asignado y el utilizado tanto de los datos como de los metadatos?

- Ahora de espacio usado tengo 2.0G, mientras que de ocupado realmente tengo 2.51G.

### 7. Asignar la otra partición a /disco5. ¿Se modificaron los valores con respecto al punto anterior?

- `mkfs.btrfs /dev/sda10`
- `mount /dev/sda10 /disco5`
- Si, `/dev/sda10` tiene como espacio libre lo que tenia `/dev/sda9` antes de crear el archivo de 2GB.

### 8. Generar otro archivo de 3000MB en el directorio /disco5. ¿Aumenta el espacio alocado? ¿Cuánto espacio se ha ocupado realmente?

- El espacio alocado aumento (2,8G).
- El espacio ocupado realmente es de 3.0G.

### 9. Usando las dos particiones anteriores crear un RAID1 y montarlo en /disco5. ¿Qué partición puede elegir para montar el file system? (Desmontar previamente la partición montada en /disco5.)

- `umount /mnt/rd5` 
- `mkfs.btrfs -f -m raid1 -d raid1 /dev/sda9 /dev/sda10`
- Pude elegir la partición `/dev/sda9` para montar el file system.

![captura_mkfs.btrfs_raid1](/Practicas/Practica%203/img/Clipboard09.png)


### 10. ¿Es posible generar los dos archivos anteriores en ese filesystem? ¿Por qué?

- Si, esto es posible por el balanceo de datos. Esto permite que los datos se distribuyan uniformemente entre múltiples dispositivos cuando se configura un RAID.

### 11. Eliminar todo el contenido de /disco5 y generar dos subvolúmenes, llamados vol1 y vol2. ¿Puede ver los subvolúmenes creados? ¿Qué ID tiene cada uno? ¿Qué significa el ID 5?

- `cd /disco5 && rm -r *`
- `btrfs subvolume create /disco5/vol1`
- `btrfs subvolume create /disco5/vol2`
- Con `btfs subvolume list /disco5` podemos ver información de los subvolumenes.
  - El vol1 tiene id 256 y el vol2 tiene 259.

### 12. Montar esos volúmenes, vol1 y vol2, en los directorios /volumen1 y /volumen2 respectivamente. ¿Qué espacio disponible tiene cada volumen? ¿Es posible acotar el espacio de un volumen? ¿Es necesario que esté montado el subvolumen top-level para poder montar sus subvolúmenes?

- `mount -o subvol=vol1 /dev/sda9 /volumen1`
- `mount -o subvol=vol2 /dev/sda9 /volumen2`
- El espacio disponible de cada volumen es de 2,7G.
- Es posible acotar el espacio de un subvolumen (cuotas de subvolumen).
- No es necesario que el subvolumen superior esté montado para montar sus subvolumenes.

### 13. Generar un archivo de 300MB en el directorio /disco5/vol1, ¿es posible ver el archivo en /volumen1? Si ejecuta el comando df -h, ¿qué espacio se ha consumido disco5, volumen1 y volumen2? ¿Por qué sucede esto?

- Si, es posible ver el archivo en `/volumen1`.
- Para los 3, el espacio consumido es el mismo.
- Esto es así por el COW y por el "espacio compartido" (todos los subvoluemenes comparten el mismo espacio de almacenamiento subyacente).


### 14. Limitar el tamaño del subvolumen volumen2 a 300MB. Intentar copiar un archivo de 400MB, ¿es posible hacerlo? (Hint: btrfs quota ...)

- `btrfs quota enamble /disco5`
- `btrfs qgroup limit 300M /disco5/vol2 /disco5`
- No pude copiar un archivo de 400MB.

### 15. Elevar el tamaño de la cuota a 450MB, ¿es posible ahora? (Previamente revisar si el volument está vacío.)

- `btrfs qgroup limit 450M /disco5/vol2 /disco5`
- Ahora si pude copiar el archivo de 400MB.

### 16. Realizar un snapshot del subvolumen /disco5/vol1 en /disco5/snap. Antes de esto crear un archivo con el texto Esto es una prueba de un snapshot y otro archivo de 100MB. Chequear, antes y después de generar el snapshot, con df -h y los comandos de btrfs el espacio alocado y consumido. ¿Se incrementó el espacio consumido? ¿Por qué?

- `cd /disco5/vol1 && rm -r *`
- `echo "Esto es una prueba de un snapshot" > prueba.txt`
- `dd if=/dev/zero of=/disco5/vol1/pruebal`
- Tamaño usado = 2.8G, Tamaño total usado = 3.0G
- `mkdir /disco5/snap`
- `btrfs subvolume snapshot /disco5/vol1 /disco5/snap`

### 17. Modificar el contenido el archivo original agregándole para sistemas operativos. ¿Se modifica la copia en el snapshot?

- No, no se modificó la copia del snapshot.

### 18. Si se desea volver al subvolumen original, ¿cómo lo haría? (sin hacer un copy o move de los archivos)

- `umount /volumen1`
- `btrfs subvolume delete /disco5/vol1`
- `btrfs subvolume snapshot /disco5/snap/vol1 /disco5/vol1`
- `mount -o subvol=vol1 /dev/sda9 /volumen1`