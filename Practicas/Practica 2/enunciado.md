# Practica 2 - Syscalls, Módulos y Drivers

## System Calls - Conceptos generales

### 1. ¿Qué es una System Call? ¿Para qué se utiliza?

- Es la forma en que un programa se comunica con el kernel para solicitar un servicio.
- Pueden usarse para manejar archivos, gestionar procesos, comunicarse con otros procesos, asignación de memoria y manejo de direcciones, etc. (todas tareas privilegiadas)

### 2. ¿Para qué sirve la macro `syscall`? Describa el propósito de cada uno de sus parámetros.

- La macro `syscall` se usa en SO tipo _Unix_ para invocar una System Call desde un programa.
- Su sintaxis general es:

```c
syscall(kernel_call_number, arg1, arg2, arg3, arg4, arg5, arg6);
```

- Parámetros:
  - `kernel_call_number`: Integer que representa la System Call a invocar. Cada System Call está asociada a un número único.
  - `arg1, ..., arg6`: Argumentos que se le pasan a la System Call. Depende de cada una, no todas usan los 6 parámetros, y también cada una define su propia cantidad y tipos.
- Ejemplo: La System Call `open()` se usa así:
```c
#include <unistd.h>
#include <sys/syscall.h>
#include <fcntl.h>

int main() {
    char *filename = "/path/to/file.txt";
    int flags = O_RDONLY;
    int mode = 0;
    int fd = syscall(SYS_open, filename, flags, mode);
    return 0;
}
```

### 3. ¿Para qué sirven los siguientes archivos?

- Ambos son archivos de tabla de System Calls, en arquitectuas de 32 y 64 bits. Definen la relación entre los números de System Calls y las funciones que las implementan.

#### a. `<kernel_code>/arch/x86/entry/syscalls/syscall_32.tbl`

- Contiene la tabla de System Calls para arquitectura de 32 bits.
- Cada línea define una System Call, el primero campo es el número y el segundo la función. Por ejemplo:
  - `249 sys_mbind`

#### b. `<kernel_code>/arch/x86/entry/syscalls/syscall_64.tbl`

- Su estructura y propósito son similares a `syscall_32.tbl`, pero está orientado a arquitectura de 64 bits.

### 4. ¿Para qué sirve la macro asmlinkage?

- La macro `asmlinkage` se usa para definir las funciones del kernel que implementan las System Calls.
- Deben seguir una convención específica para garantizar compatibilidad y portabilidad.
- Se define en `include/linux/kernel.h` del código fuente del kernel (aunque su definición varía según la arquitectura y compilador usado). Su propósito es que las funciones que implementan las System Calls sigan dichas convenciones.
- Ejemplo:
  - `sys_write` es la implementación de la System Call `write`.
```c
asmlinkage long sys_write(unsigned int fd, const char __user *buf, size_t count);
```

### 5. ¿Para qué sirve la herramienta `strace`? ¿Cómo se usa?

- Herramienta de SOs tipo _Unix_ que se usa para monitorear las System Calls y las señales recibidas por un proceso.
- Su propósito es:
  - Mostrar todas las System Calls realizadas por un proceso. Nos serviría para ver cómo un programa interactúa con el SO.
  - Mostrar las señales recibidas por el proceso, como **SIGINT** y **SIGSEGV**.
  - Brindar información detallada de las operaciones E/S, uso de memoria y otros eventos del sistema. Lo que facilita la depuración.
- Para usarla, se ejecuta el comando seguido del programa/comando a rastrear, por ejemplo: `strace ls`
- Algunos de sus parámetros son:
  - `-e trace=<syscalls>` -> Rastrea solos las syscalls específicadas (se separan por comas).
  - `-o <file>` -> Envía la salida a un archivo.
  - `-f` -> Rastrea los procesos hijos creados por el proceso.
  - `-c` -> Cuenta el número de veces que se llama cada System Call.
  - `-p <pid>` -> Adjunta `strace`a un proceso ya en ejecución.

## Agregamos una nueva System call

### 1. Añadir una nueva entrada al final de la syscall table.

- Nuestro sistema es x86_64. La tabla está en `/home/so/kernel/linux-6.8/arch/x86/entry/syscalls/syscall_64.tbl`
- Vamos hasta la última syscall (la 461). Agregamos una nueva entrada:
  - `462 common rqinfo sys_rqinfo`
- La estructura general es `number abi name entry_point`
  - `number` -> Número **único** de System Call.
  - `ABI` -> Interfaz que define cómo se comunican los programas con el kernel a través de las System Calls.
  - `name` -> Texto que representa el nombre de la System Call, la cuál apunta a la función real de la implementación.
  - `entry_point` -> Nombre de la función en el código fuente.

### 2. Añadir declaración de la System CAll en los headers del kernel.
- Añadimos el header en el archivo `/home/so/kernel/linux-6.8/include/linux/syscalls.h`
- Añadimos la línea `asmlinkage long sys_rqinfo(unsigned long *ubuff, long len);` después de la línea `asmlinkage long sys_ni_syscall(void);`

### 3. Incluir el código de la syscall.

- Esto se puede hacer añadiendo un nuevo archivo al conjunto del kernel o poner el código en un archivo existente.
  - En la primera opción tenemos que modifcar los makefiles del kernel para incluir el nuevo archivo.
- Añadimos la función de la syscall al archivo `/home/so/kernel/linux-6.8/kernel/sched/core.c`, después del último `#endif`.
  - En este archivo se implementan otras syscalls relacionadas con el scheduler de CPU y SO.
  - Además muchas funciones que necesita nuestra función están en este archivo.
- Nuestra syscall accederá a la estructura rq (runqueue), definida en `/home/so/kernel/linux-6.8/kernel/sched/sched.h`. Tenemos una rq por procesador y cada proceso estará ligado a una rq.
  - Esa estructura mantiene información adicional por cada procesador, la cuál será el objetivo de la syscall.
  - El valor `current` de la función es una macro que retorna la estructura que representa el proceso que llamó a la syscall. Esa estructura se llama `task_struct` y está en `/home/so/kernel/linux-6.8/include/linux/sched.h`
  - Bloqueamos la esturctura rq antes de acceder a ella (ya que es compartida).
  - La función `cpoy_to_user` sirve para exportar los datos obtenidos al programa que usará la syscall.
  - Intentará acceder a dos datos almacenados en los campos de la rq.
    - `nr_running`: Tareas en ejecución para el procesador.
    - `nr_uninterruptible`: Tareas en estado no interrumpible.

### 4. Re-compular el Kernel.

1. Posicionarnos en el directorio del kernel `cd ~/kernel/linux-6.8`
2. Compilar el kernel -> `make -jx`
3. Instalar módulos -> `sudo make modules_install`
4. Re-instalar -> `make install`

## Monitoreando System Calls

### 1. strace con HolaMundo.c

- Las System Calls invocadas son:
  - `execve`
  - `brk`
  - `mmap`
  - `access`
  - `openat`
  - `newfstatat`
  - `mmap`
  - `close`
  - `openat`
  - `read`
  - `pread64`
  - `newfstatat`
  - `pred64`
  - `mmap`
  - `close`
  - `arch_prctl`
  - `set_tid_address`
  - `set_robust_list`
  - `rseq`
  - `mprotect`
  - `prlimit64`
  - `munmap`
  - `getrandom`
  - `brk`
  - `write`
  - `exit_group`

### 2. Comparación de strace con programa que imprime PID.

- La salida de ambos programas es la misma. Generalmente se suele usar más la `libc`, mientras que `syscall` es usado para tareas de más bajo nivel.


## Módulos y Drivers - Conceptos generales

### 1. ¿Cómo se denomina en GNU/Linux a la porción de código que se agrega al kernel en tiempo de ejecución? ¿Es necesario reiniciar el sistema al cargarlo? Si no se pudiera utilizar esto. ¿Cómo deberíamos hacer para proveer la misma funcionalidad en Gnu/Linux?

- En GNU/Linux, la porción de código que se agrega al kernel dinámicamente es conocida como **módulo**.
- No es necesario reiniciar el sistema para cargarlo, ya que pueden ser cargados/descargados a demanda.
- Si no pudieramos usar esto, para brindar la misma funcionalidad deberíamos hacerlo a través de System Calls.

### 2. ¿Qué es un driver? ¿Para qué se utiliza?

- Un driver es un software implementado cómo módulo, con el cuál el SO se comunica e interactúa con los dispositivos adyacentes.

### 3. ¿Por qué es necesario escribir drivers?

- Los drivers nos brindan una capa de abstracción de acceso a los dispositivos. Además, nos permiten aprovechar el funcionamiento del dispositivo, mejorar su rendimiento, solucionar problemas, etc.

### 4. ¿Cuál es la relación entre módulo y driver en GNU/Linux?

- Los drivers se empaquetan y cargan en el kernel cómo módulos, lo cúal provee un mayor dinamismo, independencia, modularidad y abstracción.

### 5. ¿Qué implicancias puede tener un bug en un driver o módulo?

- Puede tener implicancias graves, pero al ser cargables/descargables dinámicamente, si ocurre un bug podríamos simplemente descargar el módulo (no cómo en las SysCalls que nos puede dar un error).

### 6. ¿Qué tipos de drivers existen en GNU/Linux?

- Drivers de bloque: Controlan dispositivos de almacenamiento de bloques (HDD, USB, CD, DVD, etc.).
- Drivers de caracteres: Manejan dispositivos de transmisión de caracteres (terminales, modems, impresoras, etc.). Transfieren datos secuencialmente de a un byte por vez.
- Drivers de red: Controlan dispositivos de interfaz de red (tarjetas Ethernet, WiFi, etc.)

### 7. ¿Qué hay en el directorio /dev? ¿Qué tipos de archivo encontramos en esa ubicación?

- En el directorio `/dev` podemos encontrar archivos especiales que representan dispositivos. Permiten que los programas puedan acceder y comunicarse con los recursos del sistema.
- También podemos encontrar recursos del kernel (`/dev/null`, `/dev/zero`, `/dev/random`), enlaces simbólicos a otros dispositivos, nodos para recursos de memoria, puertos, etc.
- Los archivos que representan a los dispositivos pueden ser:
  - Dispositivos de caracteres (`c`): Manejan la transferencia de caracteres secuencialmente (`/dev/tty`, `/dev/pts`, `/dev/lp0`, etc.).
  - Dispositivos de bloque (`b`): Acceden a dispositivos de almacenamiento en bloque de datos (`/dev/sda`, `/dev/nvme0n1`, `/dev/loop0`).

### 8. ¿Para qué sirven el archivo `/lib/modules/<version>/modules.dep` utilizado por el comando `modprobe`?

- El uso de `modules.dep` por parte de `modprobe` facilita la gestión de dependencias de módulos (ya que `modules.dep` mapea y mantiene un registro de las dependencias entre módulos), así evitando que el usuario cargue manualmente los módulos.

### 9. ¿En qué momento/s se genera o actualiza un initramfs?

- Un initramfs (inital ramfs - sistema de archivos RAM inicial) se genera o actualiza:
  - Cuando se instala el SO, donde se genera un initramfs que tiene los módulos y drivers necesarios para bootear.
  - Después de instalar/actualizar el kernel, es necesario regenerar el initramfs para asegurar que tenga los módulos correctos.
  - Después de instalar/actualizar drivers, el initramfs debe actualizarse para reflejar esos cambios.
  - Cambios en la configuración de arranque.

### 10. ¿Qué módulos y drivers deberá tener un initramfs mínimamente para cumplir su objetivo?

- Módulos para montar y acceder al root filesystem. Lo que incluye módulos como ext4, xfs, btrfs, etc. (dependiendo del filesystem).
- Drivers de dispositivos de almacenamiento, para reconocer y accederlos (así accedemos a `/`). Se incluyen módulos SATA/PATA/SCSI, drivers de dispositivos USB, drivers de raid por software, etc.
- Drivers específicos del hardware (placa base, procesador, etc.) para que el booteo funcione correctamente.
- Herramientas básicas como `sh`, `mkdir`, `mount`, etc.
- También se podrían necesitar drivers de red (si el root filesystem está en una ubicación de red) o módulos de cifrado (si el root filesystem está cifrado).


## Desarrollando un módulo simple para Linux

### 1. Crear el archivo `memory.c`.

### 2. Crear el archivo `Makefile`

#### a. Explique brevemente cual es la utilidad del archivo Makefile.

- Es un archivo de construcción, su función es automatizar el proceso de compilación de programas/módulos. Contiene instrucciones y reglas que indican cómo compilar, enlazar y generar ejecutables o módulos a partir de archivos fuente.
  - Al ejecutar el comando `make` en un directorio que contiene un Makefile, se leen y ejecutan las intrucciones definidas en el.
  - Esto simplifica y estandariza el proceso de compilación.

#### b. ¿Para qué sirve la macro MODULE_LICENSE? ¿Es obligatoria?

- Sirve para especificar la licencia bajo la cuál se distribuye el módulo.
- Es obligatoria y tiene que estar presente en todos los módulos. Sino, el kernel genera un aviso y no permite cargarlo.
  - En nuestro caso se usa `MODULE_LICENSE("Dual BSD/GPL")`, que sirve para módulos que pueden ser distribuidos bajo la GPL o la licencia BSD.

### 3. Compilar módulo (en el directorio donde creamos el `Makefile` y `memory.c`)

#### a. ¿Cuál es la salida del comando anterior?

- La salida del comando anterior es:
```bash
CC [M] /home/so/modulos/memory.o
MODPOST /home/so/modulos/Module.symvers
CC [M] /home/so/modulos/memory.mod.o
CC [M] /home/so/modulos/memory.ko
```

#### b. ¿Qué tipos de archivo se generan? Explique para qué sirve cada uno.

- `memory.o`: Archivo objeto compilado de nuestro código fuente. Contiene el código máquina.
- `Module.sysmvers`: Contiene información sobre la versión del kernel para la cuál se compiló el módulo.
- `memory.mod.o`: Archivo objeto que tiene información adicional sobre el módulo (símbolos exportados, dependencias, etc.)
- `memory.ko`: Archivo del módulo de kernel compilado, listo para ser cargado. El sufijo `.ko` (kernel object) es el formato estándar para los módulos.

### 4. Agregar nuestro módulo

- `insmod`: Herramienta usada para cargar módulos de kernel en Linux.
- `modprobe`: Además de cargar módulos, puede deshabilitarlos y manejar las dependencias entre módulos. Es más seguro que `insmod`porque puede manejar las dependencias entre módulos

### 5. Verificamos que el módulo este agregado con `lsmod`

#### a. ¿Cuál es la salida del comando? Explique cuál es la utilidad del comando lsmod.

- `lsmod` sirve apra mostrar los módulos cargados en el sistema. Además brinda información como su nombre, tamaño y su uso.

#### b. ¿Qué información encuentra en el archivo /proc/modules?

- El archivo `/proc/modules` tiene información de los módulos cargados. La información que brinda es:
  - Nombre.
  - Tamaño.
  - Dependencias.
  - Número de referencias.
  - Información del autor y licencia.

#### c. Ejecutando `more /proc/modules` obtenemos los siguientes fragmentos.

```bash
memory 8192 0 - Live 0x0000000000000000 (OE)
binfmt_misc 24576 1 - Live 0x0000000000000000
intel_rapl_msr 16384 0 - Live 0x0000000000000000
intel_rapl_common 32768 1 intel_rapl_msr, Live 0x0000000000000000
```

- Cada línea representa a un módulo diferente. En cada uno tenemos información de:
  - Nombre.
  - Tamaño del módulo en bytes.
  - Número de referencias.
  - Si está en uso o no.
  - Dirección de inicio del módulo en memoria.
  - Tipo de módulo

#### d. ¿Con qué comando descargamos el módulo de la memoria?

- Podemos descargar el módulo con el comando `rmmod` o `modprobe -r`.


### 6. Descargar el módulo memory y verificar que efectivamente se descargó.

- `rmmmod`
- `lsmod | grep memory`

### 7. Modificar el módulo memory

- Compilar y cargar.
- Invocar `dmesg` -> Al final de la salida del comando aparece el _Hello World!_ del módulo memory.
- Descargar e invocar `dmesg`. -> Al final de la salida del comando aparece el _Bye cruel world_ del módulo memory.


### 8. Responder:

#### a. ¿Para qué sirven las funciones module_init y module_exit?. ¿Cómo haría para ver la información del log que arrojan las mismas?.

- `module_init`: Se usa para especificar la función que se ejecuta al cargar el módulo. Sirve para inicialziar el módulo.
- `module_exit`: Se usa para definir la función que se ejecutará al descargar el módulo. La podríamos usar para liberar recursos, hacer limpieza, etc.
- Para ver la información del log que arrojan, podemos usar el comando `dmesg` que visualiza los mensajes relevantes en el buffer de mensajes del sistema. 

#### b. Hasta aquí hemos desarrollado, compilado, cargado y descargado un módulo en nuestro kernel. En este punto y sin mirar lo que sigue. ¿Qué nos falta para tener un driver completo?.

- Saber el major number de nuestro dispositivo.
- Definir las operaciones de lectura, escritura, apertura y liberación en un `struct` llamado `file_operations`.
- Además tenemos que registrar y des-registrar nuestro driver.

#### c. Clasifique los tipos de dispositivos en Linux. Explique las características de cada uno.

- Dispositivos de acceso aleatorio: Permiten acceder a los datos de forma aleatoria (HDD, CD/DVD, USB, etc.)
- Dispositivos seriales: Permiten acceder a los datos secuencialmente (puertos serie, modems, impresoras, etc.)

## Desarrollando un Driver

### 1. Modificar el archivo memory.c

### 2. Responder:

#### a. ¿Para qué sirve la estructura ssize_t y memory_fops? ¿Y las funciones register_chrdev y unregister_chrdev?

- `ssize_t`: Usado para representar el tamaño de un objeto en bytes.
- `memory_fops`: Define las operaciones permitidas en un archivo o dispositivo (abrir, leer, escribir, cerrar).
- Las funciones `register_chrdev` y `unregister_chrdev` se usan para registrar y desregistrar un driver de caracteres.

#### b. ¿Cómo sabe el kernel que funciones del driver invocar para leer y escribir al dispositivo?

- El kernel sabe qué funciones del driver invocar a través de la estructura `file_operations`. Dicha estructura tiene punteros a funciones que implementan las operaciones permitidas.

#### c. ¿Cómo se accede desde el espacio de usuario a los dispositivos en Linux?

- Se accede a los dispositivos en Linux a través de archivos especiales en el directorio `/dev`. Esos archivos representan a los dispositivos y se crean cuando se registra un driver de dispositivo.

#### d. ¿Cómo se asocia el módulo que implementa nuestro driver con el dispositivo?

- El módulo que implementa nuestro driver se asocia con un dispisitivo a través del `register_chrdev` y el major number del dispositico, que crea un archivo en `/dev` y lo asocia al driver.

#### e. ¿Qué hacen las funciones copy_to_user y copy_from_user?(https://developer.ibm.com/technologies/linux/articles/l-kernel-memory-access/).

- Se usan para transferir datos de manera segura entre el espacio del kernel y el espacio de usuario.
- `copy_to_user`: Copia datos desde el espacio del kernel al espacio de usuario. Se usa cuando un driver necesita enviar datos a una aplicación.
- `copy_from_user`: Copia datos desde el espacio de usuario al espacio del kernel. Se usa cuando un driver necesita recibir datos de una aplicación.

### 3. Creamos el archivo `/dev/memory`

- Con el comando `mknod` que sirve para crear archivos especiales (dispositivos, nodos, etc.).
- Con el parámetro `c` indicamos que el archivo está asociado a un dispositivo de caracteres (si fuera de bloque sería una `b`).
- El 60 y el 0 son los major y minor number.

### 4. Cargamos el módulo del driver.

#### ii. ¿Qué son el “major” y el “minor” number? ¿Qué referencian cada uno?

- Son números enteros usados por el kernel para identificar y gestionar los dispositivos univocamente.
- **Major Number**: Identifica el driver o tipo de dispositivo. Cada driver en el kernel tiene asignado un único major number.
- **Minor Number**: Identifica una instancia específica de un dispositivo controlado por un driver determinado. Un mismo driver (con un único major number) puede gestionar múltiples dispositivos, cada uno identificado con un minor number.

### 5. Escribimos y leemos del dispositivo en el dispositivo

### 7. Responder:

#### a. ¿Qué salida tiene el anterior comando?, ¿Porque? (ayuda: siga la ejecución de las funciones memory_read y memory_write y verifique con dmesg)

- Usando `dmesg`, podemos ver que tuvimos 6 llamados a `memory_write`, esto es porque se escriben 6 caracteres (`echo -n abcdef /dev/memory`).
- Preguntar por qué aparece solo una f.

#### e. En el caso de un driver que lee un dispositivo como puede ser un file system, un dispositivo usb,etc. ¿Qué otros aspectos deberíamos considerar que aquí hemos omitido? ayuda: semáforos, ioctl, inb,outb.

- Los semáforos serían esenciales para sincronizar el acceso a estructuras de datos, buffers usados para leer y escribir desde/hacia el dispositivo.
- La operación `ioctl` permite a un driver brindar una interfaz (a través de comandos) para configurar y controlar el comportamiento del dispositivo, lo que nos da más flexibilidad y control sobre el dispositivo.
- En algunos casos, los dispositivos pueden comunicarse a través de puertos E/S. La función `inb` se usa para leer datos de un puerto E/S y la función `outb` se usa para escribir datos en un puerto de E/S.
- También pueden haber dispositivos que usen interrupciones y que el driver deberá tener un manejador de interrupciones adecuado. Y que el driver maneje adecuadamente los estados de suspención y reanudación de los dispositivos.

