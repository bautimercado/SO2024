# Practica 4 - Parte 1 - cgroups & namespaces

## Conceptos teóricos

### 1. Defina virtualización. Investigue cuál fue la primera implementación que se realizó.

- La virtualización es un proceso que permite crear versiones virtuales de recursos informáticos, como servidores, redes, almacenamientos o sistemas operativos. Su objetivo es optimizar el uso de esos recursos, mejorar la eficiencia, flexibilidad y reducir costos.
  - Hace posible la ejecución de múltiples sistemas operativos y aplicaciones en un mismo hardware de forma aislada y segura.
- La primera implementación de virtualización x86 fue de VMware, que lanzó "VMware Virtual Plataform"

### 2. ¿Qué diferencia existe entre virtualización y emulación?

- La virtualización implica crear versiones virtuales de recursos informáticos para optimizar su uso y eficiencia.
  - Se ejecutan múltiples SO virtualizados en un mismo hardware, permitiendo una gestión eficiente de recursos.
- La emulación se centra en imitar el funcionamiento de un software o sistema en un entorno diferente.
  - Se busca que un sistema se comporte de cierta forma o imite otro sistema

### 3. Investigue el concepto de hypervisor y responda:

#### (a) ¿Qué es un hypervisor?

- El hypervisor (o VMM) es un software que crea y gestiona VMs. Funciona abstrayendo y aislando las VMs y sus SO del hardware subyacente, permitiendo un uso más eficiente de los recursos físicos, simplificando el mantenimiento y operaciones, y reduciendo costos.
- Asigna recursos como CPU, memoria y almacenamiento a las VMs, garantizando su correcto funcionamiento y aislamiento del entorno físico.

#### (b) ¿Qué beneficios traen los hypervisors? ¿Cómo se clasifican?

- Beneficios de los hypervisors:
  - Permiten configurar, implementar y administrar aplicaciones sin estar limitado a una configuración de hardware específica (independencia del hardware).
  - Simplifican la configuración de SOs de servidor, agilizando la creación de entornos virtuales (eficientes).
  - Maximizan el uso de recursos en computadoras físicas al ejecutar mútiples cargas de trabajo en una sola máquina (escalables).
  - Facilitan la continuidad de software al virtualizar el entorno de hardware necesario, permitiendo la ejecución de aplicaciones en diferentes entornos (portabilidad).
- Clasificación de hypervisors:
  - **Tipo 1 (Bare-Metal):** Se ejecutan directamente en el hardware del servidor, gestionando los SOs invitados.
  - **Tipo 2 (Hosted):** Se ejecutan como aplicaciones en un SO existente. Son menos eficientes, pero más fáciles de implementar.

### 4. ¿Qué es la full virtualization? ¿Y la virtualización asistida por hardware?

- **Full Virtualization:** Técnica en la que el hypervisor simula el hardware subyacente, permitiendo la ejecución de SOs sin modificaciones.
  - El SO invitado no es consciente de que se está virtualizando, lo que faciltia la portabilidad y ejecución de SOs no modificados.
  - Combina la ejecución directa y la traducción binaria para lograr un rendimiento óptimo, manteniendo una caché de instrucciones traducidas recientemente para mejorar la eficiencia.
- **Hardware-assisted Virtualization:** Técnica en la que el hardware subyacente brinda instrucciones de CPU para facilitar la virtualización.
  - Permite que el hypervisor ejecute SOs no modificados, y simplifica la implementación del hypervisor, al hacerlo menos complejo y más mantenible.

### 5. ¿Qué implica la técnica binary translation? ¿Y trap-and-emulate?

- **Binary translation:** Técnica en la que el hypervisor reescribe dinámicamente las instrucciones del SO invitado, de forma que las instrucciones no virtualizables puedan generar una trampa al hypervisor para ser emuladas.
  - Esto permite evitar trampas en la mayoría de instrucciones no privilegiadas, mejorando el rendimiento.
  - Tiene mejor rendimiento al evitar la mayoría de las trampas.
  - Requiere un proceso más complejo de reescritura de código.
- **trap-and-emulate:** El hypervisor ejecuta instrucciones del SO invitado y captura las instrucciones privilegiadas que generan una trampa.
  - El hypervisor emula el efecto de esas instrucciones sobre el estado virtual de la máquina.
  - Tiene un mayor overhead por cada trampa.
  - Es más sencillo de implementar.
- Con trampas se refiere a las excepciones o interrupciones generadas por el SO invitado (guest) que intenta ejecutar instrucciones privilegiadas.

### 6. Investigue el concepto de paravirtualización y responda:

#### (a) ¿Qué es la paravirtualización?

- Técnica de virtualización que permite ejecutar SOs modificados sobre un hypervisor.
- A diferencia de la virtualización completa, el hypervisor no emula un hardware completo, sino que ofrece una API a través de la cuál los SOs guests acceden al hardware físico del servidor host.

#### (b) Mencione algún sistema que implemente paravirtualización.

- **Xen:** Un hypervisor de código abierto que permite la paravirtualización de SOs como Linux, FreeBDS y NetBSD.
- **Oracle VM Server for SPARC:** Hypervisor de Oracle que soporta paravirtualización para SOs SPARC.
- **IBM z/VM:** SO de IBM para mainframes que implementa paravirtualización.

#### (c) ¿Qué beneficios trae con respecto al resto de los modos de virtualización?

- **Mejor rendimiento:** Al no emular un hardware completo, la paravirtualización reduce la sobrecarga y mejora el rendimiento de las VMs.
- **Simplicidad del hypervisor:** Al requerir modificaciones en el SO guest, el hypervisor es más simple y eficiente.
- **Aprovechamiento de recursos:** La paravirtualización permite un mejor aprovechamiento de los recursos del servidor físico al reducir la sobrecarga.

### 7. Investigue sobre containers y responda:

#### (a) ¿Qué son?

- Es una unidad de software que empaqueta una aplicación junto con sus dependencias (librerías, configuraciones, datos, etc.) en un único paquete que es fácil de transportar y ejecutar.
- Permite que la aplicación funcione consistentemente en diferentes entornos, más allá del SO o infraestructura adyacente.

#### (b) ¿Dependen del hardware subyacente?

- No directamente, ya que se ejecutan sobre un SO host.
- Si dependen de ciertas características del kernel del SO (como cgroups y namespaces), que brindan aislamiento y virtualización a nivel SO.

#### (c) ¿Qué lo diferencia por sobre el resto de las tecnologías estudiadas?

- A diferencia de las VMs, los containers comparten el kernel del SO host, lo que los hace más livianos y eficientes en el uso de recursos.
- Los containers se enfocan en empaquetar aplicaciones y sus dependencias, mientras que las VMs virtualizan hardware completo.

#### (d) Investigue qué funcionalidades son necesarias para poder implementar containers.

- Un motor de contenedores (Docker o containerd) que brinda las funcionalidades básicas para ejecutar y administrar contenedores.
- Características del kernel como cgroups y namespaces (para aislamiento y virtualización).
- Herramientas para crear, desplegar y administrar contenedores de forma fácil y eficiente, como las que proporciona Docker.
- Un orquestador de contenedores, como Kubernetes, para gestionar y escalar aplicaciones basadas en contenedores.

## Chroot

### 1. ¿Qué es el comando chroot? ¿Cuál es su finalidad?

- Es una herramienta para crear entornos aislados dentro de un sistema Unix, ofreciendo seguridad y flexibilidad en diversos contextos de administración y desarrollo
- Se usa para cambiar el root directory del proceso actual y de sus procesos hijo a un nuevo directorio especificado. En su nuevo root directory, el proceso no puede acceder a ningún archivo fiera de este nuevo directorio.

### 2. Crear un subdirectorio llamado sobash dentro del directorio root. Intente ejecutar el comando chroot /root/sobash. ¿Cuál es el resultado? ¿Por qué se obtiene ese resultado?

- `mkdir /root/sobash`

![chroot_salida1](/Practicas/Practica%204/img/Clipboard01.png)

- `chroot` no pudo encontrar el intérprete de bash en el nuevo entorno. Para esto, debemos asegurarnos que este directorio tenga todas las dependencias y archivos necesarios (podríamos copiar todos los archivos que se requieren).

### 3. Copiar en el directorio anterior todas las librerías que necesita el comando bash. Para obtener esta información ejecutar el comando `ldd /bin/bash`. ¿Es necesario copiar la librería linux-vdso.so.1? ¿Por qué? Dentro del directorio anterior crear las carpetas donde va el comando bash y las librerías necesarias. Probar nuevamente. ¿Qué sucede ahora?

![salida_ldd](/Practicas/Practica%204/img/Clipboard02.png)

- No es necesario copiar la librería `linux-vdso.so.1`, ya que es un mecanismo del kernel y no es un archivo real, sino una interfaz para syscalls.

```bash
mkdir -p /root/sobash/{bin,lib,lib64}
cp /bin/bash /root/sobash/bin
cp /lib/x86_64-linux/gnu/{libtinfo.so.6,libdl.so.2,libc.so.6} /root/sobash/lib
cp /lib64/ld-linux-x86-64.so.2 /root/sobash/lib64/
chroot /root/sobash /bin/bash
```

- El prompt cambió, estamos en una sesión de shell `bash` y estamos ejecutando como superusuario (`#`). 

### 4. ¿Puede ejecutar los comandos cd "directorio" o echo? ¿Y el comando ls? ¿A qué se debe esto?

- No no pude ejecutar ninguno de esos comandos.
- Esto es porque el comando `chroot` cambia el directorio raiz para el proceso actual y es posible que no se tengan todos los binarios y librerías necesarias dentro del nuevo entorno.

### 5. ¿Qué muestra el comando pwd? ¿A qué se debe esto?

- El comando `pwd` muestra `/`, esto es porque para el proceso actual, su directorio raiz es donde ejecutamos el `chroot` (`/root/sobash`).

### 6. Salir del entorno chroot usando exit

- Muy difícil.

## Control Groups

### 1. ¿Dónde se encuentran montados los cgroups? ¿Qué versiones están disponibles?

- Los cgroups en Linux se montan en el file system virtual ubicado en `/sys/fs/cgroup`. Este es un directorio que da una representación jerárquica de los cgroups y sus configuraciones.
- **Cgroups v1 (legado):** Versión original e inicial de cgroups. Nació en el kernel 2.6.24 y era la implementación por defecto. Cada driver tiene su propia jerarquía de cgroups.
- **Cgroups v2:** Versión más reciente y mejorada de cgroups. Fue introducida en el kernel 4.5 y se volvio totalmente operativa en el kernel 4.19. Brinda una jerarquía unificada para todos los controladores, lo que simplifica su uso y administración.
- Con `grep cgroup /proc/filesystems` podemos ver las versiones presentes en el sistema

### 2. ¿Existe algún controlador disponible en cgroups v2? ¿Cómo puede determinarlo?

- Si, en cgroups exiten varios controladores. Estos permiten gestionar y limitar diferentes recursos del sistema para los procesos pertenecientes a un determinado cgroup.
- Podemos determinar qué controladores están disponibles en tu sistema con cgroups v2 con `grep -P '^(controller|kernel)' /sys/kernel/cgroup/cgroup.controllers`.
- Algunos controladores más comunes son:
  - **cpu:** Controla la asignación de tiempo de CPU.
  - **cpuset:** Permtie asignar procesos a CPU y nodos de memoria específicos.
  - **io:** Controla las operaciones de entrada/salida en dispositivos de bloque.
  - **memory:** Controla el uso de memoria y swap.
  - **pids:** Limita el número de procesos.
  - **hugetlb:** Controla el uso de páginas enormes de memoria.
  - **cpuacct:** Genera informes del uso de CPU.
  - **devices:** Controla el acceso a dispositivos.

### 3. Analice qué sucede si se remueve un controlador de cgroups v1 (por ej. Umount /sys/fs/cgroup/rdma).

- Pueden pasar varios escenarios:
  - **Procesos existentes:** Los procesos que ya estaban asignados a ese cgroup del controlador desmontado, continuan ejecutándose sin las restricciones de ese controlador.
  - **Imposibilidad de agregar nuevas tareas:** No se puede agregar nuevos procesos a ese cgroup.
  - **Configuraciones perdidas:** Todas las configuraciones que se aplican a ese controlador se pierden.
  - **Falta de estadísticas:** Si el controlador proporcionaba estadísticas sobre el uso de recursos, ya no será posible obtener esa información.

### 4. Crear dos cgroups dentro del subsistema cpu llamados cpualta y cpubaja. Controlar que se hayan creado tales directorios y ver si tienen algún contenido `# mkdir /sys/fs/cgroup/cpu/"nombre_cgroup"`

- Se crearon los directorios con los siguientes archivos:
  - **cgroup.clone_children** → Controla si los procesos recién creados deben ser añadidos al mismo cgroup que el padre.
  - **cgroup.procs** → Contiene los PIDs que pertenecen al cgroup.
  - **cpuacct.stat** → Estadísticas del uso de CPU acumuladas para el cgroup y sus hijos.
  - **cpuacct.usage** → Muestra el uso de CPU en nanosegundos para el cgroup y sus hijos.
  - **cpuacct.usage_all** → Muestra el uso de CPU en nanosegundos para el cgroup, incluyendo el uso de sus hijos.
  - **cpuacct.usage_percpu** → Muestra el uso de CPU en nanosegundos por cada CPU individual.
  - **cpuacct.usage_percpu_sys** → Muestra el uso de CPU en nanosegundos en modo kernel para el cgroup y sus hijos.
  - **cpuacct.usage_percpu_user** → Muestra el uso de CPU en nanosegundos por cada CPU individual en modo usuario.
  - **cpuacct.usage_sys** → Muestra el uso de CPU en nanosegundos en modo kernel para el cgroup y sus hijos.
  - **cpuacct.usage_user** → Muestra el uso de CPU en nanosegundos en modo usuario para el cgroup y sus hijos.
  - **cpu.cfs_period_us** → Especifica el período de tiempo en microsegundos para el cual se aplica la cuota CPU (cpu.cfs_quota_us).
  - **cpu.cfs_quota_us** → Especifica el total de tiempo en microsegundos que se permitirá a las tareas ejecutarse durante cada período (cpu.cfs_period_us).
  - **cpu.shares** → Especifica el porcentaje máximo de CPU que cada uno puede usar.
  - **cpu.stat** → Proporciona estadísticas de uso de CPU acumuladas para el cgroup.
  - **notify_on_release** → Controla si se generará un evento cuando el último proceso abandone el cgroup.
  - **tasks** → Contiene la lista TIDs que pertenecen a este cgroup.

### 5. En base a lo realizado, ¿qué versión de cgroup se está utilizando?

- Se usa la versión 1 de cgroup, ya que cada controlador tiene su subgrupos de cgroups.

### 6. Indicar a cada uno de los cgroups creados en el paso anterior el porcentaje máximo de CPU que cada uno puede utilizar. El valor de cpu.shares en cada cgroup es 1024. El cgroup cpualta recibirá el 70 % de CPU y cpubaja el 30 %. `# echo 717 > /sys/fs/cgroup/cpu/cpualta/cpu.shares` y `# echo 307 > /sys/fs/cgroup/cpu/cpubaja/cpu.shares`


### 7. Iniciar dos sesiones por ssh a la VM.(Se necesitan dos terminales, por lo cual, también podría ser realizado con dos terminales en un entorno gráfico). Referenciaremos a una terminal como termalta y a la otra, termbaja.

### 8. Usando el comando taskset, que permite ligar un proceso a un core en particular, se iniciará el siguiente proceso en background. Uno en cada terminal. Observar el PID asignado al proceso que es el valor de la columna 2 de la salida del comando. `# taskset -c 0 md5sum /dev/urandom &`

- PID termalta: 2371
- PID terbaja: 2375

### 9. Observar el uso de la CPU por cada uno de los procesos generados (con el comando top en otra terminal). ¿Qué porcentaje de CPU obtiene cada uno aproximadamente?

![top_captura](/Practicas/Practica%204/img/Clipboard03.png)

- termalta usa un 49,3% de la CPU.
- termbaja usa un 49,7% de la CPU.

### 10. En cada una de las terminales agregar el proceso generado en el paso anterior a uno de los cgroup (termalta agregarla en el cgroup cpualta, termbaja en cpubaja. El process_pid es el que obtuvieron después de ejecutar el comando taskset) `# echo "process_pid" > /sys/fs/cgroup/cpu/cpualta/cgroup.procs`


### 11. Desde otra terminal observar cómo se comporta el uso de la CPU. ¿Qué porcentaje de CPU recibe cada uno de los procesos?

![top_captura](/Practicas/Practica%204/img/Clipboard04.png)

- termalta usa un 70% de la CPU.
- termabaja usa un 30% de la CPU.

### 12. En termalta, eliminar el job creado (con el comando jobs ven los trabajos, con kill %1 lo eliminan. No se olviden del %.). ¿Qué sucede con el uso de la CPU?

- `kill 2371`

### 13. Finalizar el otro proceso md5sum.

- `kill 2375` 

### 14. En este paso se agregarán a los cgroups creados los PIDs de las terminales (Importante: si se tienen que agregar los PID desde afuera de la terminal ejecute el comando echo $$ dentro de la terminal para conocer el PID a agregar. Se debe agregar el PID del shell ejecutando en la terminal). `# echo $$ > /sys/fs/cgroup/cpu/cpualta/cgroup.procs` (termalta) y `# echo $$ > /sys/fs/cgroup/cpu/cpubaja/cgroup.procs` (termbaja)

- PID termalta: 1512
- PID termbaja: 2355

### 15. Ejecutar nuevamente el comando `taskset -c 0 md5sum /dev/urandom &` en cada una de las terminales. ¿Qué sucede con el uso de la CPU? ¿Por qué?

- Está igual que antes. Esto es porque agregamos a los cgroups los PID de las terminales (que vendrían a ser los procesos padres), esto hace que sus procesos hijos también formen parte del cgroup y estén atadas a la configuración de cada cgroup.

### 16. Si en termbaja ejecuta el comando: `taskset -c 0md5sum /dev/urandom &` (deben quedar 3 comandos md5 ejecutando a la vez, 2 en el termbaja). ¿Qué sucede con el uso de la CPU? ¿Por qué?


- El uso de la CPU para los procesos de termbaja es de 15,0 cada uno. Esto se debe a que pertenecen al mismo cgroup, y por lo tanto se dividen entre si el uso del core.

## Namespaces

### 1. Explique el concepto de namespaces.

- Es un concepto que permite aislar y virtualizar recursos del sistema para diferentes entornos/procesos.
- Cada namespace actúa como una capa de abstracción separada para un tipo específico de recurso del sistema. Esto permite que los procesos tengan vistas aisladas de esos recursos.
- Los namespaces permiten virtualizar los recursos, lo que significa que un proceso puede tener su propia vista lógica de un recurso.
- Al aislar recursos, los namespaces ayudan a mejorar la seguridad al evitar que los procesos accedan a recursos a los que no deberían tener acceso.
- Los namespaces son una funcionalidad clave en la implementación de contenedores, ya que permiten aislar los recursos para cada contenedor.

### 2. ¿Cuáles son los posibles namespaces disponibles?

- **mnt:** Aisla el árbol de directorios de archivos (incluyendo puntos de montaje).
- **pid:** Aisla los PIDs. Cada proceso puede tener su propio espacio de PIDs.
- **net:** Aisla interfaces de red, tablas de ruteo, etc. Cada namespace tiene su propio stack de red.
- **ipc:** Aisla recursos de comunicación entre procesos (queue message, semáforos, memoria compartida, etc.).
- **uts:** Aisla los identificadores del kernel.
- **user:** Aisla los UID y GID. Cada proceso tiene su propio mapeo de UID y GID.
- **cgroup:** Aisla los grupos de control de recursos (cgroups).
- **time:** Aisla la percepción del tiempo para cada namespace.

### 3. ¿Cuáles son los namespaces de tipo Net, IPC y UTS una vez que inicie el sistema (los que se iniciaron la ejecutar la VM de la cátedra)?

- Podemos listar los namespaces con `lsns`
- Los namespaces Net, IPC y UTS son:
  - net: 4026531992 y 4026532229
  - ipc: 4026531839
  - uts: 4026531838


### 4. ¿Cuáles son los namespaces del proceso cron? Compare los namespaces net, ipc y uts con los del punto anterior, ¿son iguales o diferentes?

- Primero averiguramos el PID del proceso cron: `pidof cron`
- Listamos los namespaces de cron con `lsns -p <pid_cron>`
- Son los mismos que en el punto anterior.


### 5. Usando el comando unshare crear un nuevo namespace de tipo UTS.

#### a. `unshare -–uts sh`

#### b. ¿Cuál es el nombre del host en el nuevo namespace? (comando hostname)

- so2022

#### c. Ejecutar el comando lsns. ¿Qué puede ver con respecto a los namespace?.

- Podemos ver que hay un nuevo namespace de tipo uts (el que creamos).

#### d. Modificar el nombre del host en el nuevo hostname.

- `hostname so2024`

#### e. Abrir otra sesión, ¿cuál es el nombre del host anfitrión?

- so2022

#### f. Salir del namespace (exit). ¿Qué sucedió con el nombre del host anfitrión?

- Cambió. Volvió a ser so2022.

### 6. Usando el comando unshare crear un nuevo namespace de tipo Net.

#### a. `unshare -–pid sh`

#### b. ¿Cuál es el PID del proceso sh en el namespace? ¿Y en el host anfitrión?

- El PID del proceso sh en el namespace es 1395 (con `echo $$ se imprime el PID del proceso actual`).
- En el host, su PID es 1395 (`ps -e | grep "sh"`)

#### c. Ayuda: los PIDs son iguales. Esto se debe a que en el nuevo namespace se sigue viendo el comando ps sigue viendo el /proc del host anfitrión. Para evitar esto (y lograr un comportamiento como los contenedores), ejecutar: `unshare --pid --fork --mount-proc`

#### d. En el nuevo namespace ejecutar `ps -ef`. ¿Qué sucede ahora?

- En el nuevo namespace, su PID es 1.

#### e. Salir del namespace