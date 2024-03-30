# Practica 1 - Compilación del kernel Linux

## A - Conceptos teóricos.

### 1. ¿Qué es el kernel de GNU/Linux? ¿Cuáles son sus funciones principales dentro del Sistema Operativo?

- El kernel de GNU/Linux (Linux) es una porción de código esencial en el SO que permite la comunicación entre el usuario/procesos usuario/software y el hardware adyacente.
- Sus funciones principales son: Administrar memoria, administrar dispositivos, administrar la CPU.

### 2. Explique brevemente la arquitectura del kernel Linux teniendo en cuenta: tipo de kernel, módulos, portabilidad, etc.

- Linux es un kernel tipo monolítico, es decir, todo su código/módulos se ejecutan en modo privilegiado/kernel.
- Pero, también tiene una modalidad híbrida (parte de micro-kernel), ya que cuenta con módulos que se pueden agregar/quitar bajo demanda, de este podemos añadir más funcionalidad.
- Además, Linux es multiplataforma, es compatible con múltiples arquitecturas.

### 3. ¿Cómo se define el versionado de los kernels Linux en la actualidad?

- Actualmente (a partir de la 3.0), el versionado de Linux se compone de 3 números X.Y.Z
  - X -> Versión principal. Se incrementa cuando hay cambios significativos en el kernel.
  - Y -> Versión secundaria. Se incremeta cuando hay nuevas funcionalidades, características o cambios.
  - Z -> Número de revisión. Se incrementa cuando se corrigió algún error, parches de seguridad o actualizaciones menores.
- Además, también pueden tener el sufijo "-rc" para indicar que la versión es estable o de prueba.

### 4. ¿Cuáles son los motivos por los que un usuario/a GNU/Linux puede querer re-compilar el kernel?

- Soportar nuevos dispositivos.
- Agregar nuevas funcionalidades.
- Adaptar el kernel a diferentes ambientes.
- Usar una versión antigua.
- Optimizarlo, testearlo.

### 5. ¿Cuáles son las distintas opciones y menús para realizar la configuración de opciones de compilación de un kernel? Cite diferencias, necesidades (paquetes adicionales de software que se pueden requerir), pro y contras de cada una de ellas.

- **Menuconfig:** Interfaz de configuración que se ejecuta en una terminal. Permite navegar a través de las diferentes opciones mediante un menú interactivo. No requiere paquetes adicionales ya que está incluido en la mayoría de distros.
  - Es sencilla de usar y no requiere una interfaz gráfica. Es rápido y eficiente.
  - Puede ser menos intuitivo para los usuario que no están acostumbrados a la línea de comandos.
- **Xconfig:** Interfaz de condiguración gráfica que usa la biblioteca GUI de X Window System. Necesita un entorno gráfico instalado en el sistema, como Xorg y las bibliotectas de desarrollo asociadas.
  - Tiene una interfaz gráfico intuitiva y fácil de usar.
  - Puede ser más pesado en cuánto a recursos del sistema.
- **Gconfig:** Similar a Xconfig pero usa la bibliotecta GTK para su interfaz gráfica. También requiere un entorno gráfico en el sistema, así como las bibliotectas de desarrollo de GTK.
  - Tiene una interfaz gráfica familiar para aquellos que están acostumbrados a las aplicaciones GTK.
  - Puede requerir recursos adicionales y puede no estar disponible para todas las distros.
- **Nconfig:** Alternativa basada en texto a Menuconfig. Brinda una interfaz de usuario más simple y básica para configurar opciones de compilación. Generalmente no requiere paquetes adicionales.
  - Es más ligero que Xconfig y GConfig. Es adecuado para sistemas con recursos más limitados.
  - Puede carecer de algunas características avanzadas (que si las tiene Menuconfig).

### 6. Indique qué tarea realiza cada uno de los siguientes comandos durante la tarea de configuración/compilación del kernel:

#### a. `make menuconfig`

- Abre una interfaz de configuración basada en texto, donde el usuario puede seleccionar las opciones de configuración del kernel.
- Se pueden habilitar/deshabilitar módulos, características específicas del kernel, configurar drivers, ajustar opciones de rendimiento, etc.

#### b. `make clean`

- Elimina los archivos generados durante el proceso de compilación anterior. Esto incluye eliminar archivos objeto .o, archivos temporales, entre otros.
- Es útil para limpiar el directorio de compilación antes de realizar una compilación limpia.

#### c. `make` (investigue la funcionalidad del parámetro -j)

- Inicia el proceso de compilación del kernel. Compila el código fuente y genera los ejecutables del kernel, incluyendo el núcleo (vmlinux) y otros archivos necesarios para el arranque.
- El parámetro `-j` se usa para especificar el número de trabajos paralelos que se tienen que ejecutar simultaneamente durante la compilación.
  - `make -j4` ejecutaría hasta 4 tareas de compilación simultaneamente, lo que puede acelerar el proceso de compilación.

#### d. `make modules` (utilizado en antiguos kernels, actualmente no es necesario)

- Compila los módulos del kernel. En versiones anteriores, este comando era necesario para compilar los módulos, ahora se realiza automáticamente.

#### e. `make modules_install`

- Instala los módulos del kernel compilados en el directorio de destino especificado. Suelen instalarse en el directorio `/lib/modules/` con una estructura de directorios que corresponde a la versión del kernel y la arquitectura del sistema.

#### f. `make install`

- Instala los archivos del kernel compilado en el sistema. Incluye el núcleo (vmlinuz), los archivos de configuración del kernel y otros archivos necesarios para el arranque del sistema. También puede actualizar el cargador de arranque para que pueda arrancar con el nuevo kernel.

### 7. Una vez que el kernel fue compilado, ¿dónde queda ubicada su imagen? ¿dónde debería ser reubicada? ¿Existe algún comando que realice esta copia en forma automática?

- La imagen del kernel, junto con otros archivos relacionados, se encuentran en el directorio `arch/<arquitectura>/boot/`. El nombre del archivo suele ser `vmlinuz`.
- Es importante reubicar la imgaen a un directorio donde el cargador de arranque pueda encontrarlo durante el booteo del sistema. El directorio típico es `/boot/`.
- El comando `make install` puede realizar esto de manera automática.

### 8. ¿A qué hace referencia el archivo initramfs? ¿Cuál es su funcionalidad? ¿Bajo qué condiciones puede no ser necesario?

- El archivo *initramfs* (Initial RAM File System) es un sistema de archivos tempral que se carga en RAM durante el proceso de booteo de Linux, antes de que el sistema de archivos root esté disponible. Es un paso intermedio para preparar el entorno del sistema antes de que el kernel Linux pase el control al sistema de archivos real.
- Su funcionalidad es permitir la carga de módulos para el correcto funcionamiento del sistema antes de que el filesystem root esté montado.
- Puede no ser necesario cuando el kernel puede acceder directamente al filesystem root sin necesidad de pasos adicionales. Esto puede pasar en sistemas donde el filesystem root está directamente disponible en el momento del booteo.

### 9. ¿Cuál es la razón por la que una vez compilado el nuevo kernel, es necesario reconfigurar el gestor de arranque que tengamos instalado?

- Es necesario reconfigurar el gestor de arranque después de compilar el nuevo kernel para asegurar que pueda arrancar correctamente el SO usando la nueva imágen de kernel.

### 10. ¿Qué es un módulo del kernel? ¿Cuáles son los comandos principales para el manejo de módulos del kernel?

- Es un fragmento de código que puede cargarse/descargarse bajo demanda del mapa de memoria del kernel. Permiten extender su funcionalidad.
- Algunos de los comandos principales para el manejo de módulos son:
  - `lsmod`: Lista todos los módulos cargados actualmente. Muestra nombre del módulo, tamaño y una lista de otros módulos que están siendo usados por él.
  - `modinfo`: Muestra información detallada sobre un módulo (nombre, descripción, autor, versión, parámetros de configuración, etc.)
  - `insmod`: Usado para insertar un módulo específico. Se debe proporcionar la ruta al archivo del módulo como parámetro (`/lib/modules/version`)
  - `rmmod`: Se usa para descargar un módulo. Se debe proporcionar el nombre del módulo como parámetro.
  - `modprobe`: Se usa para administrar módulos de manera más avanzada. Permite cargar, descargar, configurar módulos, y manejar interdependencias (carga automáticamente los módulos que necesita uno).
  - `depmod`: Genera/actualiza el archivo de dependencias del kernel (`/lib/modules/version/modules.dep`). Ese archivo lista las dependencias entre los módulos, es usado por el **modprobe**.

### 11. ¿Qué es un parche del kernel? ¿Cuáles son las razones principales por las cuáles se deberían aplicar parches en el kernel? ¿A través de qué comando se realiza la aplicación de parches en el kernel?

- Un parche es un archivo que contiene actualizaciones (pequeñas) del código del kernel.
- Las razones principales para aplicar parches son: corrección de bugs, mejoras de rendimiento/seguridad, nuevas características, ajustes de configuración, etc.
- La aplicación de parches en el kernel se aplica con el comando `patch`, la sintaxis básica es:

```bash
patch -p<nivel> < archivo-parche
```

- `<nivel>` indica el nivel de directorio a eliminar de las rutas de archivo contenidas en el archivo de parche y `<archivo-parche>` es el archivo parche en cuestión.
- El parámetro `--dry-run` es usado para simular la aplicación de cambios, esto nos da la oportunidad de revisar los cambios antes de aplicarlos.

### 12. Investigue la característica Energy-aware Scheduling incorporada en el kernel 5.0 y explique brevemente con sus palabras:

- La Energy-aware Scheduling (EAS) está diseñada para optimizar el uso de energía en sistemas con procesadores ARM big.LITTLE

#### a. ¿Qué característica principal tiene un procesador ARM big.LITTLE?

- La característica principal de un procesador ARM big.LITTLE es su diseño heterogéneo que consta de dos tipos de núcleos:
  - Los núcleos "big" de alto rendimiento, para tareas intensivas de cómputo.
  - Los núcleos "LITTLE" de bajo consumo energético, para cargas de trabajo más livianas y ayudan a conservar la energía.
  - Está arquitectura brinda una combinación eficiente de rendimiento y eficiencia energética.

#### b. En un procesador ARM big.LITTLE y con esta característica habilitada. Cuando se despierta un proceso ¿a qué procesador lo asigna el scheduler?

- Cuando se despierta un proceso, el scheduler del kernel selecciona el núcleo apropiado para ejecutarlo, en función de la carga de trabajo y eficiencia energética.

#### c. ¿A qué tipo de dispositivos opinás que beneficia más esta característica? Ver https://docs.kernel.org/scheduler/sched-energy.html

- Beneficia especialmente a dispositivos móviles que usan procesadores ARM big.LITTLE. También puede ser beneficioso para dispositivos embebidos y sistemas integrados que operan con restricciones de energía, como dispositivos IoT, dispositivos médicos portátiles, etc.

### 13. Investigue la system call memfd_secret() incorporada en el kernel 5.14 y explique brevemente con sus palabras

#### a. ¿Cuál es su propósito?

- Su propósito principal es crear regiones de memoria anónimas y secretas. Dichas regiones son opacas para el filesystem y son inaccesibles para otros procesos, lo que las hace adecuadas para almacenar datos sensibles/confidenciales.

#### b. ¿Para qué puede ser utilizada?

- Puede ser usada para manejar información confidencial (claves de cifrado, contraseñas, tokens, etc.)

#### c. ¿El kernel puede acceder al contenido de regiones de memoria creadas con esta system call? El siguiente artículo contiene bastante información al respecto: https://lwn.net/Articles/865256/

- No, estas regiones están protegidas y son inaccesibles para otros procesos (incluido el propio kernel). Solo el proceso creador puede acceder a su contenido.
