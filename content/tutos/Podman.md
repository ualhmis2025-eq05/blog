+++
date = '2025-04-02T12:03:27+02:00'
draft = true
title = 'Podman'
+++


# Guía de Práctica: Contenerización con Podman y Podman Desktop en Windows

## Introducción a la contenerización y Podman

La **contenerización** es una tecnología que permite empaquetar una aplicación junto con todas sus dependencias en una unidad ligera llamada **contenedor**. Los contenedores aíslan la aplicación del sistema host, asegurando que se ejecute de forma consistente en cualquier entorno (desarrollo, pruebas, producción), ya que incluyen todo lo necesario (bibliotecas, configuraciones, etc.). A diferencia de las máquinas virtuales, los contenedores comparten el kernel del sistema operativo host y son mucho más livianos en términos de recursos y tiempo de arranque.

**Podman** (abreviatura de *Pod Manager*) es una herramienta de línea de comando de código abierto para gestionar contenedores y es una alternativa popular a Docker. A diferencia de Docker, Podman es *daemonless* (no requiere ejecutar un servicio demonio central) y permite ejecutar contenedores en modo *rootless*, es decir, sin privilegios de superusuario ([How to install and use Podman Desktop on Windows | Red Hat Developer](https://developers.redhat.com/articles/2023/09/27/how-install-and-use-podman-desktop-windows#:~:text=Podman%20is%20a%20popular%20tool,supports%20both%20Podman%20and%20Docker)). Esto mejora la seguridad, ya que los procesos en contenedores no necesitan permisos de root en el host. Podman es compatible con el formato de contenedores OCI, lo que significa que puede ejecutar imágenes de contenedor de Docker Hub u otros registries de la misma forma que Docker. De hecho, la sintaxis de comandos de Podman es muy similar a la de Docker, facilitando la migración (incluso es común usar `alias docker=podman` para usar comandos Docker con Podman).

Una característica destacada de Podman son los **Pods**. Al igual que en Kubernetes, un *pod* en Podman es un grupo de uno o más contenedores que comparten red y espacio de nombres. Esto permite ejecutar servicios relacionados juntos (por ejemplo, una aplicación web y su base de datos) de manera aislada en un mismo pod. Podman puede generar directamente ficheros YAML de Kubernetes a partir de contenedores o pods creados localmente, lo que facilita la transición de un entorno local a uno orquestado con Kubernetes.

Como complemento visual, tenemos **Podman Desktop**. Podman Desktop es una aplicación de interfaz gráfica multiplataforma (Windows, macOS, Linux) para facilitar la gestión de contenedores y pods ([How to install and use Podman Desktop on Windows | Red Hat Developer](https://developers.redhat.com/articles/2023/09/27/how-install-and-use-podman-desktop-windows#:~:text=With%20containerization%20becoming%20essential%20to,or%20local%20ones%20using%20Kind)). Ofrece una GUI para crear, ejecutar, detener contenedores, gestionar pods e incluso interactuar con entornos Kubernetes locales (por ejemplo, Kind u OpenShift). Podman Desktop soporta tanto Podman como Docker, por lo que se puede usar como alternativa a Docker Desktop, con la ventaja de ser gratuita y de código abierto. En esta práctica utilizaremos Podman Desktop en Windows para demostrar cómo manejar contenedores de forma visual, complementando el uso de la línea de comandos de Podman.

**¿Por qué usar Podman en lugar de Docker?** Además de la ausencia de un demonio central y el modo rootless ya mencionados, Podman no requiere privilegios de administrador en el sistema anfitrión para ejecutar contenedores. Esto elimina la necesidad de ejecutar un servicio privilegiado constantemente (Docker Engine típicamente corre como root). Cada contenedor lanzado con Podman es un proceso hijo del usuario que lo ejecuta, lo que encaja mejor con el modelo de seguridad de Linux. En entornos multiusuario, cada desarrollador puede ejecutar sus contenedores sin interferir con otros. Asimismo, Podman está desarrollado por Red Hat y forma parte del ecosistema de herramientas que incluyen Buildah (para construir imágenes) y Skopeo (para manejar imágenes en registries), integrándose bien con flujos de integración continua. En resumen, Podman ofrece **mayor seguridad y flexibilidad**, manteniendo compatibilidad con los contenedores existentes de Docker ([How to install and use Podman Desktop on Windows | Red Hat Developer](https://developers.redhat.com/articles/2023/09/27/how-install-and-use-podman-desktop-windows#:~:text=Podman%20is%20a%20popular%20tool,supports%20both%20Podman%20and%20Docker)).

En esta guía, aprenderemos a instalar Podman en Windows (usando WSL2 como backend) junto con Podman Desktop, y realizaremos ejercicios prácticos: desde ejecutar un contenedor sencillo, hasta construir una mini aplicación FastAPI con PostgreSQL en contenedores dentro de un pod, simulando un entorno de microservicios. Se cubrirán paso a paso las operaciones básicas (construir, ejecutar, listar, detener contenedores), el uso de pods en Podman, el beneficio de contenedores rootless, el uso de Podman Desktop para visualizar nuestros contenedores, y una simulación de un pipeline CI/CD local. Al final, integraremos todo en un pequeño proyecto práctico.

## 1. Instalación de Podman y Podman Desktop en Windows (WSL2)

Para ejecutar Podman en Windows utilizaremos **Windows Subsystem for Linux 2 (WSL2)**. Podman es una herramienta nativa de Linux, por lo que en Windows se ejecuta dentro de una máquina virtual ligera de Linux proporcionada por WSL2 o Hyper-V ([How to install and use Podman Desktop on Windows | Red Hat Developer](https://developers.redhat.com/articles/2023/09/27/how-install-and-use-podman-desktop-windows#:~:text=Let%27s%20install%20the%20Podman%20container,the%20Windows%20Subsystem%20for%20Linux)). Podman Desktop simplifica enormemente la instalación, ya que puede habilitar WSL2 e instalar Podman automáticamente en una distribución Linux (Fedora) dentro de WSL ([How to install and use Podman Desktop on Windows | Red Hat Developer](https://developers.redhat.com/articles/2023/09/27/how-install-and-use-podman-desktop-windows#:~:text=Let%27s%20install%20the%20Podman%20container,the%20Windows%20Subsystem%20for%20Linux)). A continuación, se detallan los pasos de instalación:

### 1.1. Prerrequisitos y configuración inicial

- **Sistema Operativo**: Windows 10 (build 19043 o superior) o Windows 11. Se recomienda tener al menos 6 GB de RAM disponibles para la máquina virtual de contenedores ([Windows | Podman Desktop](https://podman-desktop.io/docs/installation/windows-install#:~:text=Check%20that%20your%20environment%20has%3A)) ([Windows | Podman Desktop](https://podman-desktop.io/docs/installation/windows-install#:~:text=,machine%3A%20%20120%20Nested%20Virtualization)).
- **Virtualización**: Asegúrate de que la virtualización esté habilitada en el BIOS/UEFI de tu equipo (necesario para WSL2).  
- **Permisos**: Necesitarás privilegios de administrador en Windows para instalar WSL2 y Podman.

Antes de instalar Podman Desktop, habilitaremos manualmente las características de WSL2, aunque Podman Desktop puede hacerlo automáticamente, es útil conocer el proceso. Abre una terminal de **PowerShell como Administrador** y ejecuta los siguientes comandos:

```powershell
# Actualizar WSL por si ya está instalado
wsl --update

# Instalar WSL2 sin distribución por defecto (no instala Ubuntu u otra distro de inmediato)
wsl --install --no-distribution
```

**Explicación:**  
- `wsl --install --no-distribution` habilita la característica Windows Subsystem for Linux 2 en Windows sin instalar una distribución Linux por defecto (normalmente, `wsl --install` instalaría Ubuntu automáticamente). Usamos `--no-distribution` porque Podman Desktop luego instalará una distribución optimizada (Fedora) para Podman.  
- Si es la primera vez que habilitas WSL, es posible que Windows te solicite reiniciar el sistema. Realiza el reinicio antes de continuar.

> **Nota:** En Windows 10 puede ser necesario habilitar también la característica "Virtual Machine Platform" antes de WSL2. Esto se puede hacer con el comando de PowerShell:  
> ```powershell
> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
> ```  
> seguido de un reinicio. En Windows 11 normalmente `wsl --install` se encarga de habilitar todo automáticamente. Si tienes problemas instalando WSL2, consulta la documentación de Microsoft ([How to install and use Podman Desktop on Windows | Red Hat Developer](https://developers.redhat.com/articles/2023/09/27/how-install-and-use-podman-desktop-windows#:~:text=shown%20in%20Figure%205,and%20run%20the%20following%20command)) ([How to install and use Podman Desktop on Windows | Red Hat Developer](https://developers.redhat.com/articles/2023/09/27/how-install-and-use-podman-desktop-windows#:~:text=command%20prompt%20using%20the%20following,Linux%3B%20Podman%20will%20handle%20that)).

### 1.2. Instalación de Podman Desktop

Una vez configurado WSL2, procedemos a instalar **Podman Desktop**:

1. **Descarga el instalador**: Ve al sitio oficial de Podman Desktop (https://podman-desktop.io) y descarga el instalador para Windows. Alternativamente, puedes usar un gestor de paquetes como **Winget** desde PowerShell: 

   ```powershell
   winget install -e --id RedHat.Podman-Desktop
   ```
   Esto descargará e instalará la última versión de Podman Desktop en tu sistema ([Windows | Podman Desktop](https://podman-desktop.io/docs/installation/windows-install#:~:text=)).

2. **Ejecuta el instalador**: Si descargaste el `.exe` manualmente, haz doble click y sigue las indicaciones del asistente. La instalación es sencilla: básicamente acepta la licencia y elige el directorio de instalación (puedes dejar el predeterminado). Al finalizar, marca la opción para **Iniciar Podman Desktop**.

3. **Primer inicio y configuración**: Al arrancar Podman Desktop por primera vez, es posible que te muestre una pantalla de bienvenida y realice detecciones del entorno. Si WSL2 no estaba habilitado previamente, Podman Desktop te guiará para habilitarlo (como ya lo hicimos manualmente, deberías pasar rápidamente esta etapa) ([How to install and use Podman Desktop on Windows | Red Hat Developer](https://developers.redhat.com/articles/2023/09/27/how-install-and-use-podman-desktop-windows#:~:text=As%20the%20installation%20begins%2C%20Podman,the%20process%20of%20enabling%20WSL)). 

   En la **pantalla principal del Panel (Dashboard)** de Podman Desktop, verás una sección indicando que **Podman necesita ser configurado** (por ejemplo, un botón "Setup" o "Instalar Podman"). Esto se debe a que, aunque Podman Desktop está instalado, falta instalar el motor de contenedores Podman en sí dentro de WSL.

4. **Instalar Podman (motor de contenedores)**: Desde Podman Desktop, haz clic en el botón **"Setup"** o **"Install Podman"** que aparece en el panel principal. Podman Desktop comprobará los prerrequisitos (WSL habilitado, RAM disponible, etc.) ([How to install and use Podman Desktop on Windows | Red Hat Developer](https://developers.redhat.com/articles/2023/09/27/how-install-and-use-podman-desktop-windows#:~:text=As%20the%20installation%20begins%2C%20Podman,the%20process%20of%20enabling%20WSL)) y luego instalará automáticamente una distribución Linux ligera (Fedora) en WSL2 con Podman preinstalado ([How to install and use Podman Desktop on Windows | Red Hat Developer](https://developers.redhat.com/articles/2023/09/27/how-install-and-use-podman-desktop-windows#:~:text=Let%27s%20install%20the%20Podman%20container,the%20Windows%20Subsystem%20for%20Linux)). Este proceso puede tardar unos minutos la primera vez, ya que descargará la imagen de Fedora y la configurará. 

   A mitad del proceso, Podman Desktop puede solicitar confirmación para descargar e instalar Podman (por ejemplo, mostrará un diálogo para instalar la última versión de Podman); acepta y continúa ([How to install and use Podman Desktop on Windows | Red Hat Developer](https://developers.redhat.com/articles/2023/09/27/how-install-and-use-podman-desktop-windows#:~:text=Click%C2%A0Install%20one%20last%20time%20from,Let%27s%20select%C2%A0Yes)) ([How to install and use Podman Desktop on Windows | Red Hat Developer](https://developers.redhat.com/articles/2023/09/27/how-install-and-use-podman-desktop-windows#:~:text=This%20will%20automatically%20download%20the%C2%A0installer,Figure%208)). Al finalizar, debería indicarte que **Podman está instalado correctamente**.

5. **Iniciar Podman machine**: Tras la instalación, Podman Desktop creará y arrancará automáticamente una máquina virtual de Podman (a veces llamada *Podman machine*). Básicamente, esto es la instancia de Fedora Linux en WSL donde corre el motor Podman. En la interfaz de Podman Desktop, debería aparecer ahora un recuadro o sección indicando que Podman (engine) está *running* (en ejecución) ([Windows | Podman Desktop](https://podman-desktop.io/docs/installation/windows-install#:~:text=To%20verify%20that%20Podman%20is,set%20up)). Si no estuviera corriendo, podrías encontrar un botón "Start" para iniciarlo. En la barra inferior o en *Settings > Resources* puedes ver detalles de la máquina (como memoria asignada) según Podman Desktop ([Windows | Podman Desktop](https://podman-desktop.io/docs/installation/windows-install#:~:text=)).

Ahora hemos instalado con éxito Podman Desktop y configurado Podman en Windows utilizando WSL2.

### 1.3. Verificación de la instalación

Es importante verificar que todo esté funcionando correctamente:

- **Desde Podman Desktop**: En el *Dashboard* de Podman Desktop, busca la tarjeta o indicación de estado de Podman. Debería decir "Podman is running" (Podman está en ejecución) ([Windows | Podman Desktop](https://podman-desktop.io/docs/installation/windows-install#:~:text=To%20verify%20that%20Podman%20is,set%20up)). Esto confirma que el motor de contenedores se está ejecutando en la VM de WSL.

- **Usando la línea de comandos**: Podemos usar la CLI de Podman para asegurarnos de que podemos interactuar con el motor. Abre una terminal de **PowerShell (puede ser normal, no necesitas admin a partir de ahora)** o la nueva **Terminal de Windows**. Podman Desktop agrega el binario de `podman` a tu PATH de Windows, de modo que puedes ejecutar comandos Podman directamente desde Windows y este los dirigirá a la máquina de Podman en WSL ([Run Podman on Windows: How-to instructions - Red Hat](https://www.redhat.com/en/blog/run-podman-windows#:~:text=Run%20Podman%20on%20Windows%3A%20How,handle%20the%20commands%20on%20Windows)). 

  Ejecuta el comando de versión para comprobar que Podman responde:

  ```powershell
  podman --version
  ```

  Deberías obtener una salida indicando la versión de Podman instalada, por ejemplo:
  ```plaintext
  podman version 4.x.x
  ```
  Esto confirma que la CLI de Podman está disponible.

- **Probando un contenedor de ejemplo**: Vamos a ejecutar un contenedor de prueba para verificar que todo funciona end-to-end. Usaremos la imagen de hello world estándar. Ejecuta en la terminal:

  ```powershell
  podman run --rm hello-world
  ```

  **Explicación:** Este comando intentará descargar la imagen `hello-world` desde Docker Hub (por defecto) y ejecutarla. La opción `--rm` indica que elimine el contenedor automáticamente al finalizar. La imagen `hello-world` simplemente imprime un mensaje de saludo y termina. 

  Si todo está bien, deberías ver en la salida un mensaje como:
  ```plaintext
  Hello from Docker!
  This message shows that your installation appears to be working correctly.
  ...
  ```
  (El mensaje menciona Docker porque la imagen es la misma que usa Docker, pero ejecutándose con Podman). Si ves este mensaje, **¡felicidades!** Ya tienes Podman funcionando en tu entorno Windows a través de WSL2. Podman Desktop también debería reflejar que un contenedor se creó y eliminó (puedes mirar la sección de *Containers* en la GUI para confirmarlo, quizás aparezca brevemente o en la lista de contenedores históricos).

> **Nota:** Si el comando de Podman falla en PowerShell, es posible que debas reiniciar la sesión o tu PC para que la variable PATH se actualice tras la instalación. Alternativamente, Podman Desktop tiene integrada una terminal en la sección de *Terminal* (o puedes usar una ventana de WSL manualmente). Por ejemplo, si prefieres usar WSL directamente, puedes abrir "Ubuntu" o la distro que haya configurado Podman (suele ser una distro Fedora llamada `podman-desktop` o similar) desde el menú inicio, y ejecutar los mismos comandos dentro de la shell de Linux.

Con la instalación verificada, estamos listos para comenzar a usar Podman.

## 2. Construir y ejecutar un contenedor simple (Primeros pasos)

Comenzaremos creando un contenedor sencillo para familiarizarnos con el flujo **Dockerfile/Containerfile -> Imagen -> Contenedor** utilizando Podman. Para este ejercicio, crearemos una pequeña aplicación Python que imprime un mensaje en bucle. Esto nos permitirá practicar la construcción de una imagen a partir de un archivo de contenedor (equivalente a un Dockerfile) y ejecutar un contenedor basado en esa imagen.

### 2.1. Crear la aplicación de ejemplo

En una carpeta de tu elección (por ejemplo, `C:\podman-practica\hola-mundo` o dentro de tu distribución WSL en `~/podman-practica/hola-mundo`), crea un archivo llamado `main.py` con el siguiente contenido:

```python
import time

while True:
    print("Hola, mundo desde un contenedor de Podman!")
    time.sleep(5)
```

**Explicación del código:** Este simple script de Python imprime un mensaje cada 5 segundos indefinidamente. De esta manera, al ejecutar el contenedor, tendremos una salida continua que podremos observar, y el contenedor permanecerá en ejecución (no termina inmediatamente).

### 2.2. Escribir un Containerfile (Dockerfile)

En la misma carpeta, crearemos ahora el archivo de instrucciones para construir la imagen de contenedor. Podman puede usar ficheros denominados **Dockerfile** o **Containerfile** indistintamente. Usaremos el término Containerfile para remarcar que es independiente de Docker.

Crea un archivo llamado `Containerfile` (sin extensión) en la carpeta, con el siguiente contenido:

```dockerfile
# Usar la imagen base oficial de Python (versión 3.11 slim)
FROM python:3.11-slim

# Establecer el directorio de trabajo dentro de la imagen
WORKDIR /app

# Copiar el script de la carpeta actual del host al sistema de ficheros de la imagen
COPY main.py .

# Comando por defecto al ejecutar un contenedor de esta imagen
CMD ["python", "main.py"]
```

**Explicación del Containerfile:**  
- `FROM python:3.11-slim` indica que partimos de la imagen base de Python 3.11 (variante slim, que es más ligera). Esto nos proporciona un entorno con Python ya instalado.  
- `WORKDIR /app` fija el directorio de trabajo dentro del contenedor a `/app`. Así, las siguientes instrucciones como COPY o CMD se ejecutarán en ese contexto, y si se ejecuta un contenedor, empezará en ese directorio.  
- `COPY main.py .` copia el archivo `main.py` desde el contexto de construcción (la carpeta actual en la que ejecutemos el build) al directorio actual del contenedor (por efecto de WORKDIR, `/app`). Esto incluye nuestro script dentro de la imagen.  
- `CMD ["python", "main.py"]` define el comando por defecto que correrá cuando lancemos un contenedor de esta imagen. En este caso, ejecutará Python con nuestro script, iniciando el bucle de impresión.

### 2.3. Construir la imagen con Podman

Ahora vamos a construir una imagen a partir del Containerfile. Abre una terminal en la carpeta donde tienes `Containerfile` y `main.py` (puede ser una terminal de Windows en esa ruta, o navega con `cd` en WSL/PowerShell). Luego ejecuta:

```bash
podman build -t holamundo-py:1.0 .
```

**Explicación del comando:**  
- `podman build` inicia el proceso de construcción de la imagen.  
- `-t holamundo-py:1.0` le da un nombre y etiqueta a la imagen resultante. En este caso, llamamos a la imagen `holamundo-py` y le asignamos la etiqueta `1.0` (si no especificamos etiqueta, por defecto sería `latest`). Este nombre nos permitirá referenciar la imagen más fácilmente más adelante.  
- El `.` final indica que la *build context* es el directorio actual (donde reside el Containerfile). Podman empaquetará el contenido de este directorio (a menos que esté excluido por un `.dockerignore`) y ejecutará las instrucciones del Containerfile sobre la imagen base.

Durante la ejecución de este comando, deberías ver cómo Podman (Buildah internamente) va descargando la imagen base `python:3.11-slim` de Docker Hub (si no la tenías ya) y luego copia el archivo, finalmente configurando el CMD. Al terminar, verás un mensaje de éxito indicando que la imagen fue creada, por ejemplo con un ID de imagen o el nombre que le dimos:

```plaintext
Successfully tagged holamundo-py:1.0
```

Podemos verificar que la imagen exista ejecutando:

```bash
podman images
```

Este comando lista las imágenes locales. Debería aparecer en la lista `holamundo-py` con la etiqueta `1.0`, mostrando también el tamaño de la imagen, etc.

### 2.4. Ejecutar el contenedor de la nueva imagen

Con la imagen construida, lancemos un contenedor para probarla. Usaremos Podman para ejecutar la imagen **en segundo plano** (detached mode), de forma que podamos ver que realmente está imprimiendo mensajes:

```bash
podman run -d --name holamundo_contenedor holamundo-py:1.0
```

**Explicación del comando:**  
- `podman run` crea y lanza un nuevo contenedor.  
- `-d` indica *detached mode*, es decir, que el contenedor se ejecute en segundo plano. Así el comando `podman run` no bloqueará la terminal esperando a que el contenedor termine (dado que nuestro contenedor nunca termina por sí solo debido al bucle infinito, este modo es necesario para recuperar el control de la consola).  
- `--name holamundo_contenedor` asigna un nombre amigable al contenedor en ejecución. Si no usamos `--name`, Podman generará un nombre aleatorio. Usar nombres hace más fácil referirse al contenedor luego en otros comandos. Aquí lo llamamos "holamundo_contenedor".  
- `holamundo-py:1.0` especifica la imagen a ejecutar y su etiqueta. Podman buscará esa imagen localmente (que acabamos de construir). Si no la encontrara local, intentaría descargarla de un repositorio remoto, pero en este caso existe localmente.

Al ejecutar este comando, el contenedor arrancará inmediatamente. Podemos comprobar su estado y salida de varias formas:

- Verificar que el contenedor está **corriendo**:

  ```bash
  podman ps
  ```

  Este comando lista los contenedores en ejecución (similar a `docker ps`). Deberías ver una línea listando `holamundo_contenedor`, con el nombre de la imagen `holamundo-py:1.0`, su ID, estado "Up" o "Running" y el comando que se está ejecutando (`python main.py`). También mostrará hace cuánto está corriendo.

- Ver los **logs** (salida estándar) del contenedor:

  ```bash
  podman logs holamundo_contenedor
  ```

  Esto mostrará los mensajes que nuestro contenedor ha impreso hasta ahora. Deberías ver algo como:

  ```plaintext
  Hola, mundo desde un contenedor de Podman!
  Hola, mundo desde un contenedor de Podman!
  ... (repetido cada 5 segundos)
  ```

  Cada línea corresponde a una iteración del bucle de nuestro script imprimiendo el mensaje. Puedes ejecutar `podman logs -f holamundo_contenedor` si quieres seguir (follow) las logs en tiempo real.

¡Enhorabuena! Acabas de construir tu propia imagen de contenedor y ejecutar un contenedor con Podman. Tienes un servicio (en este caso, un bucle imprimidor) corriendo de manera aislada.

En este punto, hemos aprendido a usar `podman build` para construir imágenes y `podman run` para ejecutar contenedores. En la siguiente sección aprenderemos a gestionar contenedores: listarlos, detenerlos, eliminarlos, etc., utilizando Podman.

## 3. Gestión de contenedores e imágenes con Podman

Ahora que tenemos un contenedor en ejecución, es importante saber cómo administrarlo y cómo manejar las imágenes disponibles en nuestro sistema. Veremos comandos para listar contenedores (activos o detenidos), detener un contenedor, eliminarlo, y listar/eliminar imágenes.

### 3.1. Listar contenedores en ejecución y todos los contenedores

- **Listar contenedores en ejecución:** Ya usamos `podman ps` para ver contenedores activos. Si lo ejecutas de nuevo:

  ```bash
  podman ps
  ```

  obtendrás la lista de contenedores que están *up* (corriendo). En nuestro caso debería seguir apareciendo `holamundo_contenedor`. Este comando por defecto omite los contenedores detenidos.

- **Listar todos los contenedores (incluyendo detenidos):** Para ver también contenedores que ya finalizaron o se detuvieron, usamos:

  ```bash
  podman ps -a
  ```

  La opción `-a` (all) muestra *todos* los contenedores existentes en el sistema, sin importar su estado. Si no has detenido ningún contenedor aún, la diferencia será poca. Pero más adelante, cuando hayamos detenido/eliminado contenedores, este comando permitirá ver históricos o contenedores inactivos que siguen definidos en el sistema.

### 3.2. Detener un contenedor

Detengamos nuestro contenedor en ejecución. Para ello usamos el comando `podman stop`. Este envía una señal de parada (SIGTERM seguido de SIGKILL tras un tiempo de gracia) al proceso principal del contenedor:

```bash
podman stop holamundo_contenedor
```

**Explicación:** Aquí indicamos el nombre del contenedor que queremos detener (podríamos usar el ID del contenedor también). El comando stop esperará unos segundos a que el contenedor termine (nuestro script no tiene manejo de la señal, pero el proceso de Python terminará al recibir SIGTERM). 

Tras ejecutar esto, si vuelves a listar contenedores en ejecución con `podman ps`, verás que ya no aparece `holamundo_contenedor`. En cambio, con `podman ps -a` sí debería listarlo, pero con estado **Exited (0)** indicando que salió correctamente con código 0.

### 3.3. Eliminar un contenedor

Un contenedor detenido todavía consume un poco de espacio (registro en la lista de contenedores, sus recursos de sistema de ficheros si tenía alguna modificación, etc.). Para eliminar completamente un contenedor (borrar su "rastro"), usamos `podman rm`. 

Antes de eliminarlo, listemos todos los contenedores para identificarlo con su ID o nombre si no lo tenemos ya:

```bash
podman ps -a
```

Deberías ver `holamundo_contenedor` con estado Exited. Ahora puedes eliminarlo:

```bash
podman rm holamundo_contenedor
```

**Explicación:** Esto eliminará el contenedor de nombre `holamundo_contenedor` de la lista de contenedores. Ya no aparecerá ni siquiera con `podman ps -a`. Ten en cuenta que si el contenedor estuviera corriendo, tendríamos que detenerlo antes de eliminarlo (o usar `podman rm -f` para forzar eliminación). En general es más limpio hacer stop primero como hicimos.

> **Nota:** Podman (igual que Docker) distingue entre **imágenes** y **contenedores**. Una **imagen** es el paquete inmutable con el sistema de ficheros y la configuración predeterminada. Un **contenedor** es una instancia en ejecución (o ya ejecutada) de una imagen, que puede tener cambios en su propio sistema de ficheros (esas capas de contenedor se descartan al eliminarlo si no se cometieron a una nueva imagen). El comando `podman rm` elimina contenedores, mientras que las imágenes permanecen intactas hasta que decidamos limpiarlas.

### 3.4. Listar y gestionar imágenes

Para ver qué imágenes tenemos en nuestro sistema local, utilizamos:

```bash
podman images
```

Esto mostrará una tabla con las imágenes, incluyendo la columna **REPOSITORY** (nombre), **TAG** (etiqueta), **IMAGE ID** (identificador único) y **SIZE**. Deberías ver al menos la imagen `holamundo-py` con tag `1.0` que construimos, y probablemente también la imagen base `python:3.11-slim` (ésta fue descargada como dependencia durante el build). Es posible que aparezcan listadas como `<none>` en repository si son intermedias o si la base quedó sin etiqueta local; lo importante es reconocer nuestras imágenes.

Si quisiéramos **eliminar una imagen** (por ejemplo, para liberar espacio o porque ya no la necesitamos), usaríamos `podman rmi`. Por ejemplo, para eliminar nuestra imagen de hola mundo:

```bash
podman rmi holamundo-py:1.0
```

**Explicación:** Con esto, borramos la imagen de nombre `holamundo-py` tag `1.0` del almacen local de imágenes. Si hay algún contenedor (activo o detenido) que fue creado a partir de esa imagen, el comando fallará por seguridad (no se puede eliminar una imagen en uso). En nuestro caso ya eliminamos el contenedor, así que debería proceder. La próxima vez que ejecutes `podman images`, esa imagen ya no aparecerá. (Aún podría aparecer la base de Python; podrías eliminarla también con `podman rmi python:3.11-slim` si no la necesitas, aunque puede ser caché útil para futuros builds).

> **Nota:** Podman no mantiene un demonio corriendo, pero sí lleva un registro de imágenes descargadas en tu sistema (normalmente bajo `~/.local/share/containers` en Linux para rootless, o una ruta gestionada dentro de WSL). A diferencia de Docker, no hay un servicio que consuma RAM cuando no usas contenedores; Podman solo consume recursos cuando estás ejecutando contenedores o construyendo imágenes. Por esta razón, no es necesario "cerrar Podman" — de hecho, Podman termina cuando la última operación acaba, aunque en nuestro caso la máquina WSL con Podman puede seguir en marcha para estar lista ante nuevos comandos.

Hemos cubierto las operaciones básicas: crear imágenes (`podman build`), ejecutar contenedores (`podman run`), listar (`podman ps/images`), detener (`podman stop`), y eliminar (`podman rm/rmi`). Con estas bases, pasaremos a un concepto más avanzado de Podman: el uso de **pods** para agrupar contenedores.

## 4. Uso de Pods en Podman: Múltiples contenedores (API + Base de datos)

Una de las características distintivas de Podman es la capacidad de manejar pods de contenedores, similar al modelo de pods de Kubernetes. Un **pod** es un entorno compartido donde múltiples contenedores pueden ejecutarse conjuntamente compartiendo ciertos recursos, principalmente la red (stack de puertos, localhost común) y el espacio de nombres de IPC, etc. Esto significa que los contenedores dentro de un pod pueden comunicarse entre sí vía `localhost` como si estuvieran en la misma máquina (porque efectivamente comparten la interfaz de red). También comparten puertos: un puerto expuesto al host por el pod es compartido por los contenedores del pod (por lo que cada puerto asignado debe ser único dentro del pod, no pueden dos contenedores del mismo pod usar el mismo puerto a la vez).

Vamos a ilustrar el uso de pods lanzando dos contenedores en el mismo pod: una aplicación **API (FastAPI)** y una base de datos **PostgreSQL**. Esto simulará un escenario de microservicio donde la API web consulta a la base de datos, todo ejecutándose localmente en contenedores. Para simplificar, en este paso inicial no construiremos nosotros la imagen de FastAPI (lo haremos en el proyecto final); en su lugar usaremos un contenedor de ejemplo que provea un endpoint web. Así nos centramos en cómo crear el pod y cómo se relacionan los contenedores. (Luego, en el proyecto final, sí crearemos nuestra propia imagen de FastAPI.)

### 4.1. Crear un pod con puertos compartidos

Primero, crearemos un pod. Usaremos el comando `podman pod create` para ello:

```bash
podman pod create -n mypod -p 8000:8000 -p 5432:5432
```

**Explicación:**  
- `podman pod create` crea un nuevo pod vacío (sin contenedores).  
- `-n mypod` asigna el nombre "mypod" al pod, para referirnos a él fácilmente.  
- `-p 8000:8000 -p 5432:5432` mapea puertos del pod al host, similar a como lo haríamos con `podman run -p` para un contenedor individual. Aquí estamos diciendo: expón el puerto 8000 del pod al puerto 8000 del host (usaremos 8000 para la API FastAPI), y expón el puerto 5432 del pod al 5432 del host (el puerto estándar de PostgreSQL). Mapeamos ambos porque quizás querremos probar tanto el acceso a la API vía navegador como (opcionalmente) conectarnos a la base de datos desde el host para verificar.  

El resultado de este comando es la creación de un pod. Si ejecutas `podman pod ps`, verás listado el pod **mypod** con un ID, número de contenedores (por ahora 0) y los puertos asignados. También `podman ps -a` ahora podría listar un contenedor "infra" asociado al pod (Podman crea automáticamente un contenedor de infraestructura dentro del pod que se encarga de mantener los namespaces compartidos, similar al "pause container" en Kubernetes).

> **Nota:** El contenedor infra es de la imagen `k8s.gcr.io/pause:3.x` y aparece con nombre `<podname>-infra`. No interactuamos con él directamente; Podman lo gestiona para proveer el entorno compartido. Ocupa muy poca memoria.

### 4.2. Añadir un contenedor de base de datos (PostgreSQL) al pod

Con el pod creado, vamos a lanzar ahora la base de datos PostgreSQL dentro de ese pod. Usaremos la imagen oficial de PostgreSQL de Docker Hub. Ejecuta:

```bash
podman run -d --pod mypod --name mydb \
  -e POSTGRES_USER=fastapi -e POSTGRES_PASSWORD=fastapi -e POSTGRES_DB=fastapidb \
  postgres:15
```

Vamos a desglosar este comando por partes:

- `podman run -d` hasta ahora igual que antes: ejecuta un contenedor en modo detach.
- `--pod mypod` es la parte clave: en lugar de crear un contenedor con su propia red aislada, le indicamos que se **una al pod** existente llamado "mypod". Esto significa que este contenedor no tendrá su propia interfaz de red separada, sino que usará la del pod (compartida con otros contenedores del pod).
- `--name mydb` damos nombre al contenedor de la base de datos, "mydb".
- `-e POSTGRES_USER=fastapi -e POSTGRES_PASSWORD=fastapi -e POSTGRES_DB=fastapidb` son variables de entorno para PostgreSQL:
  - `POSTGRES_USER` crea un usuario administrador con ese nombre (además del usuario `postgres` por defecto).
  - `POSTGRES_PASSWORD` la contraseña para ese usuario.
  - `POSTGRES_DB` crea una base de datos inicial con ese nombre (y otorga acceso al usuario especificado).
  
  En este caso, estamos creando un usuario "fastapi" con contraseña "fastapi", y una base de datos llamada "fastapidb". Esto será útil luego para que nuestra aplicación FastAPI se conecte usando esas credenciales. (Si no especificáramos estos, por defecto tendríamos usuario `postgres` sin contraseña dentro del contenedor, lo cual también podríamos usar, pero es buena práctica definir un usuario específico).
- `postgres:15` es la imagen de PostgreSQL versión 15. Si no la tienes, la descargará automáticamente de Docker Hub.

Tras ejecutar el comando, el contenedor de PostgreSQL se iniciará dentro del pod "mypod". Podemos verificar su estado con `podman ps`:

```bash
podman ps
```

La salida debería mostrar dos contenedores ahora:
- El contenedor `mydb` corriendo la imagen `postgres:15`.
- Posiblemente también enlistado el contenedor infra del pod (aunque `podman ps` suele omitirlo a menos que uses `--all`, pero podrías ver `mypod-infra` listado con STATUS Up).

PostgreSQL tarda unos segundos en inicializar la base de datos en el primer arranque. Podemos revisar los logs para ver cuándo está listo:

```bash
podman logs -f mydb
```

Verás mensajes de PostgreSQL configurándose. Espera hasta ver líneas como `database system is ready to accept connections` (el log está en inglés). Cuando aparezca, significa que PostgreSQL está listo y escuchando en el puerto 5432 dentro del pod.

### 4.3. Añadir un contenedor de API (FastAPI simulado) al pod

Ahora añadiremos la aplicación API. En un escenario real, este sería nuestro contenedor con FastAPI. Para propósitos de la práctica sin escribir aún el código, usaremos un contenedor de ejemplo que provee un servicio HTTP sencillo. Existe una pequeña imagen de Hashicorp llamada **http-echo** que simplemente devuelve un texto fijo para cualquier petición HTTP (útil para pruebas). La usaremos para simular la respuesta de una API FastAPI, devolviendo un saludo.

Ejecuta el siguiente comando para lanzar el contenedor de la API en el mismo pod:

```bash
podman run -d --pod mypod --name myapi \
  hashicorp/http-echo:0.2.3 -listen=:8000 -text="¡Hola desde FastAPI en Podman!"
```

**Explicación:**  
- `--pod mypod` de nuevo indica que este contenedor se une al pod "mypod", compartiendo la red con `mydb`.  
- `--name myapi` nombra el contenedor como "myapi".  
- La imagen que usamos es `hashicorp/http-echo:0.2.3`. Esta imagen al ejecutarse inicia un servidor HTTP mínimo.  
- Los parámetros que siguen después de la imagen (`-listen=:8000 -text="..."`) son argumentos pasados al proceso http-echo dentro del contenedor:
  - `-listen=:8000` le indica que escuche en el puerto 8000 (en todas las interfaces, por eso `:8000`). Esto es importante porque nuestro pod tiene asignado el puerto 8000 para exponer al host; necesitamos que el servicio de este contenedor escuche en ese puerto interno.
  - `-text="¡Hola desde FastAPI en Podman!"` es el texto que devolverá en el cuerpo de la respuesta HTTP para cualquier petición. Hemos puesto un mensaje de ejemplo que simula la respuesta de una API FastAPI.

Después de ejecutar esto, puedes comprobar nuevamente `podman ps` y verás el contenedor `myapi` corriendo junto a `mydb`. Ambos tienen la columna POD apuntando a `mypod` (Podman muestra en `podman ps` una columna POD para indicar si un contenedor pertenece a un pod y a cuál).

Ahora, **probemos la API y la comunicación dentro del pod**:

- Desde el host (tu Windows), intenta abrir en un navegador web la URL [http://localhost:8000](http://localhost:8000). Como mapeamos el puerto 8000 del pod al host, deberías recibir una página con el texto `¡Hola desde FastAPI en Podman!` que proviene del contenedor `myapi`. Si lo ves, ¡genial! Estás accediendo a la aplicación contenedorizada.
- La API (simulada) y la base de datos comparten la red interna. En una aplicación real, tu código FastAPI podría conectarse a PostgreSQL simplemente usando `localhost:5432` como host de la DB, ya que dentro del pod *localhost* es común. Podemos verificar esta conectividad con una pequeña prueba: ejecutar un comando `psql` dentro del contenedor de API o similar. Para simplificar, en lugar de instalar un cliente psql en el contenedor de API, hagamos que el host (Windows) se conecte al Postgres contenedor via localhost:
  
  Si tienes un cliente PostgreSQL instalado en Windows (por ejemplo, `psql` en tu PATH) o puedes instalar uno rápidamente, intenta:  
  ```powershell
  psql -h localhost -p 5432 -U fastapi -d fastapidb
  ```  
  Te pedirá la contraseña (que es `fastapi`). Si se conecta correctamente, podrás ejecutar por ejemplo `\dt` (listar tablas, no habrá ninguna aún) o `SELECT 1;` para probar. Esto confirma que PostgreSQL contenedor funciona y está accesible en el puerto host 5432. Sal con `\q`.  
  (Si no tienes psql a mano, este paso es opcional; lo principal es saber que la API podría acceder al DB en `localhost:5432` dentro del pod).

- Observa que no tuvimos que hacer ningún tipo de configuración de red manual entre los contenedores. Al estar en el mismo pod, la **resolución DNS interna por nombre de contenedor** también funciona: por ejemplo, el contenedor `myapi` podría resolver el hostname `mydb` y obtendría la IP interna del pod donde está PostgreSQL (y viceversa). Podman configura automáticamente un DNS interno en el pod para que los contenedores puedan encontrarse por nombre. Así que alternativamente, la app FastAPI podría conectarse a "mydb:5432". Sin embargo, usar `localhost` es sencillo porque comparten namespace de red.

Resumiendo, hemos creado un pod `mypod` y desplegado dos servicios dentro de él: una base de datos PostgreSQL y una aplicación (simulada) FastAPI. Este patrón es útil para desarrollar múltiples servicios vinculados sin necesidad de un orquestador externo; los pods de Podman te permiten agruparlos lógicamente. 

> **Nota:** En producción, normalmente se usaría Kubernetes u otro orquestador para administrar múltiples contenedores, pero Podman pods te permiten probar localmente una arquitectura de microservicios pequeña. También podrías usar `podman-compose` (una herramienta separada) para levantar servicios definidos en un estilo Docker Compose, pero conocer los pods nativos de Podman es valioso. 

En las próximas secciones, exploraremos cómo visualizar estos contenedores y pods usando Podman Desktop, y luego pasaremos a hablar de contenedores rootless y pipeline de CI/CD local. Después culminaremos construyendo la mini aplicación FastAPI+PostgreSQL con nuestras propias imágenes.

## 5. Visualización con Podman Desktop

Hasta ahora hemos interactuado con Podman principalmente a través de la línea de comandos. Sin embargo, Podman Desktop nos ofrece una interfaz gráfica útil para observar y gestionar nuestros contenedores y pods de forma más intuitiva. Vamos a utilizar Podman Desktop para inspeccionar lo que acabamos de crear y familiarizarnos con su interfaz.

Abre la aplicación **Podman Desktop** (si no la tenías ya abierta). En el panel lateral izquierdo, deberías ver varias secciones: *Containers*, *Images*, *Pods*, *Volumes*, *Networks*, etc. Nos centraremos en **Containers** y **Pods**.

 ([Podman Desktop | Podman Desktop](https://podman-desktop.io/features)) *Figura: Vista de la pestaña de contenedores en Podman Desktop, listando contenedores y pods.* 

- Haz clic en **Containers** (icono de contenedor) en la barra lateral. Aparecerá la lista de contenedores actuales. En la captura de ejemplo superior, se muestran múltiples contenedores, algunos pertenecientes a pods (indicados con el subtítulo "pod" bajo el nombre). En tu caso, deberías ver al menos los contenedores `myapi` y `mydb`. Probablemente Podman Desktop los agrupe visualmente por pod, mostrando que ambos están dentro de **mypod**. Es posible que `mypod-infra` también aparezca pero marcado como infraestructura.

- La columna **STATUS** indicará que ambos están corriendo (*Running*). La columna **PORT** podría mostrar los puertos mapeados (8000, 5432) para el pod/contendores. Y en **ACTIONS** verás botones para acciones rápidas:
  - Un botón de "Stop" ( ⏹ ) para detener el contenedor.
  - Un botón de "Resume/Start" ( ▶ ) si estuviera detenido.
  - Un botón de eliminar ( 🗑️ ) para remover el contenedor.
  - Tres puntos verticales para más acciones (inspeccionar, ver logs, etc).

- Si haces clic sobre el nombre de un contenedor en la lista (por ejemplo, `myapi`), Podman Desktop abrirá un panel de detalle. Allí puedes ver varias pestañas: **Summary** (resumen con información general, variables de entorno, comandos), **Logs** (donde verás la salida de ese contenedor; en el caso de `myapi`, cada vez que alguien accede al puerto 8000, debería registrar una petición recibida), **Inspect** (datos JSON detallados), **Terminal** (te permite abrir una consola *dentro* del contenedor, muy útil para depuración) y posiblemente **Stats** (consumo de recursos).

- Prueba en la pestaña **Logs** del contenedor `myapi`: abre [http://localhost:8000](http://localhost:8000) de nuevo en tu navegador para generar tráfico, y observa cómo las logs reflejan probablemente una línea por cada petición (http-echo podría estar registrando algo para cada hit).

- Ahora haz clic en la sección **Pods** en la barra lateral de Podman Desktop. Verás listado el pod **mypod**. Al seleccionarlo, deberías ver los contenedores que contiene y opciones para manejar el pod completo. Podman Desktop permite por ejemplo **detener o eliminar el pod entero** con sus contenedores dentro (usando botones similares a los de contenedores). También hay opciones para crear pods nuevos o incluso "Play Kubernetes YAML" que serviría para desplegar un pod a partir de un manifiesto de Kubernetes.
 ([Podman Desktop | Podman Desktop](https://podman-desktop.io/features))】 *Figura: Vista de la pestaña de pods en Podman Desktop, mostrando varios pods con sus contenedores.* 

En la imagen de ejemplo, se ven pods listados con sus nombres, cantidad de contenedores y estado. En tu caso solo habrá uno (`mypod`). Podman Desktop te da una visión clara de qué contenedores están agrupados en cada pod, cosa que en la CLI debes inferir con la columna POD de `podman ps`.

- También puedes explorar la sección **Images** en la GUI, donde aparecerán las imágenes descargadas o construidas (por ejemplo, deberías ver `postgres:15`, `hashicorp/http-echo:0.2.3`, quizás tu `holamundo-py:1.0` si no la eliminaste, etc.). Desde la pestaña Images, Podman Desktop permite construir nuevas imágenes (botón "Build") o eliminar imágenes fácilmente, e incluso **pull** (descargar) desde registries remotos.

- Podman Desktop tiene un menú de **Settings** donde puedes ver recursos asignados a la máquina (memoria, CPUs), administrar la máquina de Podman (WSL). Por defecto, asignó 6 GB de RAM a la VM de Podman; puedes ajustarlo si lo requieres.

En general, Podman Desktop es muy útil para visualizar rápidamente el estado de tus contenedores y pods sin tener que recordar comandos, y para inspeccionar detalles de configuración. No obstante, es bueno combinar ambos enfoques: la **CLI** para operaciones rápidas o scripting, y la **GUI** para observar estados y realizar algunas acciones manuales. 

Por ejemplo, podrías detener el pod `mypod` desde Podman Desktop ahora haciendo click en el botón Stop del pod (lo detendrá por completo, parando `myapi` y `mydb` juntos). Inténtalo si deseas: verás que ambos contenedores pasan a estado *Exited*. Luego puedes iniciarlo de nuevo con el botón Start (vuelve a arrancar los contenedores en el pod). Esta es una manera rápida de reiniciar todo un grupo de servicios.

> **Nota:** Ten en cuenta que Podman Desktop no reemplaza a herramientas de orquestación: es principalmente local. No "recuerda" qué contenedores estaban corriendo tras un reinicio de sistema, a menos que los marques para autoarranque. Si cierras Podman Desktop o apagas la máquina, al volver a encender debes iniciar nuevamente el pod (ya sea manualmente con `podman pod start mypod` o usando la GUI). En un entorno de producción, usarías un orquestador o un servicio para mantener contenedores en ejecución. En desarrollo, esto normalmente no es un problema.

Hemos visto cómo Podman Desktop complementa la CLI para manejar contenedores y pods de forma visual. A continuación, profundizaremos en un concepto importante de Podman: la ejecución *rootless* de contenedores y por qué es ventajosa.

## 6. Contenedores Rootless con Podman

Una de las grandes ventajas de Podman es su arquitectura **rootless**. Esto significa que **no se requieren privilegios de superusuario (root) para ejecutar contenedores**. En otras palabras, un usuario normal puede lanzar contenedores, y esos contenedores no tendrán más privilegios que los del usuario que los ejecuta en el host.

¿Por qué es esto importante? Principalmente por **seguridad**. Tradicionalmente, Docker funciona con un demonio que corre como root en el host, y los contenedores mismos típicamente se ejecutan como root dentro de su espacio aislado. Si bien hay aislamiento, ha habido vulnerabilidades que permiten escapar del contenedor; si un contenedor en Docker escapa y está corriendo como root, comprometería el sistema anfitrión con privilegios máximos. En Podman rootless, incluso si una aplicación dentro de un contenedor se escapa, se encontraría con que en el host solo tiene los permisos de un usuario sin privilegios, limitando drásticamente el posible daño.

¿Cómo logra Podman esto? Utiliza características del kernel de Linux como **user namespaces**. Cuando lanzas un contenedor rootless, Podman mapea el usuario root dentro del contenedor a un UID no privilegiado en el host (típicamente tu UID de usuario host o un rango asignado). Así, el proceso dentro del contenedor *cree* que es root (UID 0 dentro del contenedor), pero en realidad en el host podría ser, por ejemplo, UID 1000 (tu usuario) o un sub-UID asignado. Esto permite que muchas aplicaciones dentro del contenedor que normalmente requieren ser root (por ejemplo, un servicio que abre puertos <1024, o que cambia de usuario) funcionen, pero sin otorgar privilegios reales en el host.

En nuestro entorno con WSL2, cuando instalamos Podman Desktop, la máquina Linux creada para Podman funciona en modo rootless por defect ([How to install and use Podman Desktop on Windows | Red Hat Developer](https://developers.redhat.com/articles/2023/09/27/how-install-and-use-podman-desktop-windows#:~:text=daemon%20to%20be%20running%20in,that%20supports%20both%20Podman%20and))】. De hecho, si inspeccionas los procesos, verás que los contenedores están siendo ejecutados por un usuario sin privilegios en la VM de WSL (no por root). 

Podemos comprobar cierta información con el comando `podman info`:

```bash
podman info
```

Busca en la salida la sección `host` -> `security`. Debería haber un campo `"rootless": true` indicando que estamos en modo sin root. También suele indicar el UID mapeado. Si ejecutaras dentro de un contenedor un comando para obtener el UID real del host, verías que no es 0.

Otra ventaja práctica: en Linux, para que un usuario no-root pueda ejecutar contenedores rootless, su configuración de subuids/subgids debe estar preparada. Podman Desktop en Fedora/WSL hace esto automáticamente. En una instalación manual, podrías necesitar asegurarte de que tu usuario tenga rangos de UIDs asignados en `/etc/subuid` y `/etc/subgid`. Todo esto fue resuelto por Podman Desktop, por eso no tuvimos que preocuparnos.

**¿Y en Windows?** En nuestro caso Windows no tiene usuarios root de la misma manera, pero el motor Podman corre en Linux (WSL) como usuario sin privilegios. Desde la perspectiva de Windows, todo está aislado en la VM. Esto significa que incluso en Windows, usar Podman es seguro ya que no hay ningún servicio con privilegios elevados que pueda afectar al sistema Windows directamente.

Además de la seguridad, hay otra ventaja: **multiusuario**. Podman permite que diferentes usuarios en la misma máquina lancen sus propios contenedores sin pisarse. No hay un demonio global al que todos deban acceder. Cada usuario tiene su instancia o podman system service si lo levanta, con sus propias imágenes y contenedores (aunque las imágenes se pueden compartir a nivel del sistema si los permisos lo permiten, pero generalmente cada rootless tiene su storage separado).

> **Nota:** Docker ha introducido modos rootless también en años recientes, pero Podman fue construido con este concepto desde el inicio, por lo que la experiencia tiende a ser más fluida en Podman para escenarios rootless.

En resumen, **contenedores rootless** proporcionan una capa extra de aislamiento y reducen la necesidad de permisos administrativos para trabajar con contenedores. Como desarrolladores, esto nos permite probar aplicaciones en contenedores sin pedir permisos especiales en máquinas de la empresa, por ejemplo, y sin exponer tanto el sistema. Siempre es posible que un contenedor rootless necesite algún ajuste (por ejemplo, ciertas capacidades del kernel pueden no estar disponibles sin root), pero la mayoría de las aplicaciones funcionan perfectamente.

Para nuestra práctica, simplemente ten en cuenta que estás usando Podman en modo rootless. Si intentas hacer algo que requiera privilegios especiales del kernel, podrías necesitar opciones adicionales (como `--cap-add` para añadir capacidades específicas al contenedor, o en último caso correr Podman como root dentro de WSL, lo cual normalmente no hará falta). Hasta ahora, nuestros escenarios (servidores web, bases de datos) funcionan sin problemas en rootless.

## 7. Simulación de un pipeline CI/CD local (Construcción y prueba de imágenes)

En un entorno de **Integración Continua/Despliegue Continuo (CI/CD)**, típicamente se automatizan las tareas de construir imágenes de contenedor, ejecutar pruebas en ellas y finalmente desplegarlas o publicarlas en un registro. Podman se puede integrar en pipelines CI/CD igual que Docker (por ejemplo, usando Podman en GitHub Actions, GitLab CI, Jenkins, etc., en lugar de Docker). Pero aquí nos centraremos en **simular ese proceso localmente**, para entender cómo sería el flujo de una manera manual.

Imaginemos que cada vez que hacemos un cambio en nuestra aplicación, queremos asegurarnos de que la imagen de contenedor se construye correctamente y que la aplicación funciona (al menos pasa unos tests básicos). Los pasos clave de un pipeline de contenedores podrían ser:

1. **Construir la imagen** a partir del código actualizado.
2. **Lanzar contenedores de prueba** para verificar que la imagen funciona. Esto puede incluir:
   - Ejecutar tests automatizados dentro del contenedor (por ejemplo, pruebas unitarias).
   - Desplegar el contenedor en un entorno de prueba y ejecutar pruebas de integración (por ejemplo, hacer solicitudes HTTP a la API y comprobar respuestas).
3. **Opcional: Analizar la imagen** (escaneo de vulnerabilidades, tamaño, etc.) y **publicar la imagen** en un registro si todo va bien.
4. **Limpiar** los contenedores de prueba.

Vamos a hacer una versión simplificada: usaremos nuestra aplicación en contenedor para demostrar cómo comprobar su funcionamiento automáticamente.

Sigamos con el ejemplo del pod `mypod` que creamos (FastAPI + PostgreSQL). Supongamos que esa es nuestra aplicación. Un pipeline local sencillo podría:

- Reconstruir la imagen de la API (en caso de cambios).
- Arrancar el pod (o contenedores) con la nueva imagen.
- Ejecutar una comprobación: por ejemplo, hacer una petición HTTP a la API y esperar cierta respuesta.
- Detener/eliminar los contenedores de prueba.

Vamos a simular esto con comandos. Cerremos/eliminemos el pod anterior para empezar "limpios":

```bash
# Detener y eliminar el pod mypod si sigue existiendo
podman pod stop mypod
podman pod rm mypod
```

Asumamos que tenemos el código de la API y el Containerfile listos (en el siguiente apartado lo prepararemos). De momento, usemos el mismo `hashicorp/http-echo` como si fuera "nuestra" imagen de app para la prueba CI/CD.

### 7.1. Construcción de imagen (fase Build)

Si hubiéramos hecho cambios en el código de la API, aquí reconstruiríamos la imagen. Por ejemplo, más adelante construiremos `fastapi-app:1.0`. En un pipeline podríamos hacer:

```bash
podman build -t fastapi-app:1.0 .
```

(Suponiendo el Containerfile correcto en el dir actual). Esto generaría la nueva imagen con los cambios.

### 7.2. Despliegue en entorno de prueba (fase Deploy/Test)

Ahora, lanzaríamos contenedores con la imagen para probar. Podríamos recrear el pod:

```bash
podman pod create -n testpod -p 8000:8000 -p 5432:5432
```

Luego lanzar los servicios:

```bash
# Base de datos
podman run -d --pod testpod --name dbtest \
  -e POSTGRES_USER=fastapi -e POSTGRES_PASSWORD=fastapi -e POSTGRES_DB=fastapidb \
  postgres:15

# API (aquí usaríamos la imagen recien construida; como no la tenemos en este instante, usamos http-echo a modo de ejemplo)
podman run -d --pod testpod --name apitest \
  hashicorp/http-echo:0.2.3 -listen=:8000 -text="CI Test OK"
```

En un pipeline real, en vez de `hashicorp/http-echo...` sería algo como:
```bash
podman run -d --pod testpod --name apitest fastapi-app:1.0
```
para lanzar la imagen recién construida de nuestra aplicación FastAPI.

Luego, el pipeline esperaría un momento a que los servicios se inicien (especialmente la DB) y ejecutaría pruebas. Podría ser tan sencillo como hacer un curl desde el host hacia la API:

```bash
curl -s http://localhost:8000
```

**Explicación:** Usamos `-s` (silent) para que curl no muestre progreso, solo la respuesta. Este comando devolverá el cuerpo HTTP de la respuesta de la API. En nuestro ejemplo, debería devolver "CI Test OK". En una API real FastAPI, quizás devuelva un JSON `{"message": "Hello, world"}` o algo conocido.

Podríamos integrar esto en un script y verificar el contenido. Por ejemplo, en pseudocódigo bash:

```bash
RESPONSE=$(curl -s http://localhost:8000)
if [[ "$RESPONSE" == *"CI Test OK"* ]]; then
  echo "Prueba pasada: la respuesta de la API es la esperada."
  EXIT_CODE=0
else
  echo "Prueba fallida: la respuesta de la API no es la esperada."
  EXIT_CODE=1
fi
```

Si tuviéramos endpoints más específicos o tests unitarios, podríamos ejecutarlos aquí. Por ejemplo, podríamos tener un contenedor separado que corra `pytest` contra la aplicación (si la imagen incluye los tests), o podríamos usar `podman exec` para ejecutar comandos dentro del contenedor en marcha.

- *Ejemplo con `podman exec`:* Si nuestra imagen de FastAPI tuviera pruebas unitarias en el código, podríamos hacer:  
  `podman exec apitest pytest /app/tests` (esto ejecutaría pytest dentro del contenedor `apitest`). El resultado (exit code) de ese comando nos diría si pasaron las pruebas unitarias.

Para mantener las cosas simples, nos quedamos con la idea del curl como test de integración básico.

### 7.3. Limpieza post-prueba

Una vez realizadas las pruebas, el pipeline apagaría y eliminaría los contenedores/pod para limpiar recursos:

```bash
podman pod stop testpod
podman pod rm testpod
```

Y si la construcción y pruebas fueron exitosas, se pasaría quizás a publicar la imagen (ej: `podman push fastapi-app:1.0 <registro>`). Si fallaron, se aborta el pipeline y no se publica nada.

### 7.4. Automatización

Podman puede integrarse en scripts Shell, Makefiles o archivos de pipeline CI fácilmente. No tiene un demonio separado, así que los comandos `podman build` y `podman run` se pueden ejecutar en cualquier entorno que tenga Podman instalado (incluyendo contenedores de builder en GitHub Actions, etc.). 

En local, podrías escribir un **script de prueba** que haga todo lo anterior de forma automática. Por ejemplo, un script `test_app.sh` que construya la imagen, levante los contenedores de test, ejecute curl, analice la respuesta y al final limpie. Este script podría usarse para validar cambios rápidamente antes de commitear, simulando lo que hará el CI.

> **Nota:** Asegúrate de no dejar contenedores colgando en caso de falla. Podman permite usar `podman run --rm` para que contenedores se autodestruyan al terminar, aunque en nuestro caso los contenedores de servicios no terminan solos. Otra técnica es usar `trap` en bash para que al recibir SIGINT/SIGTERM se limpien los recursos (útil si abortas manualmente). Estas prácticas de scripting aseguran que siempre partas de un estado limpio.

Con esto, hemos simulado cómo sería un pipeline CI/CD local para nuestras aplicaciones contenerizadas. En la práctica, herramientas como **GitHub Actions** podrían ejecutar comandos Podman en runners de Linux (requiriendo instalar Podman en el runner, o usando acciones específicas). La ventaja de Podman es que al no necesitar privilegios, es más sencillo ejecutarlo en entornos limitados sin dar permisos de root (muchos servicios CI limitan la capacidad de usar Docker por temas de privilegios; Podman puede funcionar en rootless mode ahí).

## 8. Proyecto Final: Aplicación FastAPI + PostgreSQL en contenedores con Podman

Llegamos al ejercicio integrador. Ahora construiremos una mini aplicación utilizando **FastAPI** (un framework web en Python) que interactúe con una base de datos **PostgreSQL**, todo contenerizado con Podman. Organizaremos la aplicación en dos contenedores dentro de un pod (similar a lo que hicimos en la sección de pods, pero esta vez creando nuestra propia imagen para la aplicación FastAPI en lugar de usar un contenedor de ejemplo). El objetivo es repasar todos los pasos: escribir un Dockerfile/Containerfile para la app, construir la imagen, ejecutar los servicios en un pod, probar la funcionalidad, y finalmente ver todo funcionando quizá a través de Podman Desktop también.

### 8.1. Preparar la aplicación FastAPI

Primero, creemos una sencilla aplicación FastAPI. Esta aplicación tendrá un par de endpoints: uno de saludo para verificar que funciona y otro que intente leer algo de la base de datos (por ejemplo, la versión de PostgreSQL o una tabla ficticia) para comprobar la integración.

**Archivos de la aplicación:**

- `main.py` – Código principal de la aplicación FastAPI.
- `requirements.txt` – Lista de dependencias de Python.
- Opcionalmente, podríamos tener un script SQL para inicializar DB, pero para simplicidad usaremos la DB tal cual.

Crea una carpeta de proyecto, por ejemplo `podman-practica/fastapi-app` y dentro crea los archivos:

**`requirements.txt`:**

```text
fastapi
uvicorn[standard]
psycopg2-binary
```

**Explicación:** Especificamos las dependencias:
- `fastapi` es el framework web.
- `uvicorn[standard]` es el servidor ASGI para correr la app (la versión "standard" incluye extras como autoreload, etc., aunque no necesariamente usaremos autoreload en contenedor).
- `psycopg2-binary` es el driver Postgres para Python, para poder conectar a PostgreSQL desde FastAPI.

**`main.py`:**

```python
from fastapi import FastAPI
import os
import psycopg2

app = FastAPI()

# Configuración de conexión a la base de datos desde variables de entorno
DB_HOST = os.getenv("DB_HOST", "localhost")
DB_NAME = os.getenv("POSTGRES_DB", "fastapidb")
DB_USER = os.getenv("POSTGRES_USER", "fastapi")
DB_PASSWORD = os.getenv("POSTGRES_PASSWORD", "fastapi")

@app.get("/")
def read_root():
    return {"message": "Hola desde FastAPI en Podman!"}

@app.get("/db-version")
def get_db_version():
    try:
        conn = psycopg2.connect(host=DB_HOST, database=DB_NAME, user=DB_USER, password=DB_PASSWORD)
        cur = conn.cursor()
        cur.execute("SELECT version();")
        version = cur.fetchone()[0]
        cur.close()
        conn.close()
        return {"postgres_version": version}
    except Exception as e:
        # En caso de error (por ejemplo, DB no accesible)
        return {"error": str(e)}
```

**Explicación del código:**  
- Leemos las variables de entorno `DB_HOST`, `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, proporcionando valores por defecto que coinciden con lo que usaremos (localhost, fastapidb, fastapi, fastapi).
- Definimos la app FastAPI con dos rutas:
  - `GET /` – retorna un mensaje de saludo para comprobar que la API responde.
  - `GET /db-version` – intenta conectarse a la base de datos PostgreSQL usando psycopg2 y ejecutar `SELECT version();` (una consulta que devuelve la versión del servidor PostgreSQL). Retorna esa versión en un JSON. Si hay algún error (por ejemplo, conexión rechazada), captura la excepción y la devuelve en el campo "error". Esto nos sirve para verificar la comunicación con la base de datos.
- Notar que estamos usando la interfaz sincrónica de psycopg2 dentro de una función FastAPI regular (no async). Esto está bien para una prueba sencilla, aunque en producción uno podría usar asyncpg o SQLAlchemy async. Pero mantengámoslo simple.

### 8.2. Escribir el Dockerfile/Containerfile para la aplicación FastAPI

En el mismo directorio `fastapi-app`, crearemos un `Containerfile` (o Dockerfile) para empaquetar nuestra aplicación en una imagen de contenedor:

```dockerfile
# Imagen base de Python (usaremos slim para tener entorno reducido)
FROM python:3.11-slim

# Instalar dependencias del sistema que puedan hacer falta (psycopg2 necesita libpq)
RUN apt-get update && apt-get install -y libpq-dev && rm -rf /var/lib/apt/lists/*

# Crear directorio de la aplicación
WORKDIR /app

# Copiar requisitos e instalarlos
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar el código de la aplicación
COPY main.py .

# Exponer el puerto (no estrictamente necesario con Podman, pero informativo)
EXPOSE 8000

# Comando para lanzar la aplicación FastAPI con Uvicorn
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Explicación del Containerfile:**  
- Partimos de `python:3.11-slim` como base, que es similar a la que usamos antes. 
- Instalamos `libpq-dev` porque `psycopg2` (no puro Python) requiere las librerías de PostgreSQL (libpq) para compilar o para runtime. En muchas imágenes base de Python esto no está incluido, así que lo añadimos via apt. Tras instalar, limpiamos la lista de paquetes para no dejar basura (`rm -rf /var/lib/apt/lists/*`).
- Establecemos `/app` como directorio de trabajo.
- Copiamos el archivo de requisitos y luego instalamos las dependencias con pip. Usamos `--no-cache-dir` para no dejar caché de pip (menos espacio).
- Luego copiamos el código `main.py` al contenedor.
- `EXPOSE 8000` documenta que el contenedor expone el puerto 8000 (esto no hace el mapeo, pero algunas herramientas pueden leerlo; Podman Desktop por ejemplo lo muestra).
- Por último, definimos el CMD para arrancar Uvicorn server sirviendo nuestra app (`main:app` refiere al objeto `app` en el módulo `main.py`). Ponemos host 0.0.0.0 para que Uvicorn escuche en todas las interfaces dentro del contenedor (necesario para poder atender conexiones externas al contenedor), y puerto 8000.

### 8.3. Construir la imagen de la aplicación FastAPI

Con todos los archivos preparados (`requirements.txt`, `main.py`, `Containerfile`), procedemos a construir la imagen:

Asegúrate de estar en el directorio `fastapi-app` en la terminal y ejecuta:

```bash
podman build -t fastapi-app:1.0 .
```

Esto iniciará la construcción:
- Descargará python:3.11-slim si no lo tienes.
- Ejecutará apt-get para libpq (que descargará unos paquetes, unos pocos MB).
- Instalará las dependencias Python (fastapi, uvicorn, psycopg2...). Esto podría tardar un poco la primera vez (psycopg2-binary puede instalar binarios precompilados, lo cual acelera).
- Finalmente, copiará nuestro código y definirá el CMD.

Si todo sale bien, obtendrás un mensaje de éxito con la imagen `fastapi-app:1.0` creada.

Verifica con `podman images` que `fastapi-app` aparece en la lista.

### 8.4. Ejecutar la aplicación y base de datos en un pod con Podman

Ahora vamos a desplegar nuestra solución completa. Creamos un nuevo pod (por ejemplo, "myapppod") y lanzamos PostgreSQL y nuestra app FastAPI en él.

```bash
# Crear un pod para la app con puertos DB y API expuestos
podman pod create -n myapppod -p 8000:8000 -p 5432:5432
```

Ahora, contenedor de base de datos (Postgres):

```bash
podman run -d --pod myapppod --name myapppg \
  -e POSTGRES_USER=fastapi -e POSTGRES_PASSWORD=fastapi -e POSTGRES_DB=fastapidb \
  postgres:15
```

Igual que antes, esto inicia Postgres. Espera unos segundos para que inicialice. Podemos seguir logs o simplemente dar un pequeño sleep antes de lanzar la app para asegurarnos de que DB arranque (en un script podrías hacer un loop intentando conectar, pero aquí manual con unos segundos suele bastar).

Finalmente, contenedor de la aplicación FastAPI:

```bash
podman run -d --pod myapppod --name myapi \
  -e POSTGRES_USER=fastapi -e POSTGRES_PASSWORD=fastapi -e POSTGRES_DB=fastapidb -e DB_HOST=localhost \
  fastapi-app:1.0
```

**Explicación:**  
Estamos pasando las variables de entorno necesarias a la app:
- `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` para que el contenedor de la app conozca las mismas credenciales que usamos en la DB.
- `DB_HOST=localhost` porque dentro del pod, la DB es accesible en "localhost" (comparten red). Alternativamente, podríamos poner `DB_HOST=myapppg` y funcionaría también porque el DNS interno del pod resolvería `myapppg` al IP del contenedor DB, pero es sencillo usar localhost en pods.
  
Nuestra app leerá estas variables en `main.py` y usará psycopg2 para conectarse.

Cuando ejecutamos este comando, se lanza la aplicación FastAPI. Dado que en el Containerfile definimos CMD con uvicorn, el contenedor debería iniciar Uvicorn y comenzar a escuchar en `0.0.0.0:8000` dentro del pod. 

Podemos comprobar:
- Ejecuta `podman ps` para ver que `myapi` está "Up" junto con `myapppg`.
- Ve las logs de la app: 
  ```bash
  podman logs -f myapi
  ```
  Allí deberías ver algo como:
  ```
  INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
  INFO:     Started reloader process [...]
  INFO:     Started server process [...]
  INFO:     Waiting for application startup.
  INFO:     Application startup complete.
  ```
  Esto indicaría que FastAPI (Uvicorn) levantó correctamente. Si ves algún error en logs (por ejemplo de conexión a DB), revisa que la DB esté levantada y las env vars sean correctas. Si la DB tardó un poco más y el app dio error de conexión al inicio, FastAPI no va a detenerse por ello - nuestro código simplemente devolverá error en la ruta /db-version pero el servidor sigue vivo. Puedes ignorar un error inicial, o reiniciar el contenedor de la app para reintentar. En entornos reales se maneja con reintentos de conexión.

- Ahora probemos los endpoints de la API desde el host (Windows):
  - Abre un navegador e ingresa [http://localhost:8000](http://localhost:8000). Nuestra aplicación debería responder con el JSON `{"message": "Hola desde FastAPI en Podman!"}`. Si lo ves, ¡la API está funcionando! 🎉
  - Ahora prueba [http://localhost:8000/db-version](http://localhost:8000/db-version). La primera vez, es posible que obtengas un JSON `{"postgres_version": "PostgreSQL 15.x ..."}` con la versión exacta de PostgreSQL que está ejecutándose en el contenedor. Esto confirmaría que la aplicación pudo conectarse a la base de datos y obtener la versión. Si en su lugar ves `{"error": "..."}`, podría ser que el contenedor de la app no pudo conectar (posiblemente por orden de arranque). Si este es el caso, espera unos segundos y refresca /db-version; eventualmente debería conectar. (Nuestro código no implementa un retraso/reintento robusto, por simplicidad).
  
  - También puedes probar la API usando `curl` en terminal:
    ```bash
    curl http://localhost:8000
    curl http://localhost:8000/db-version
    ```
    Verás las respuestas en texto.

- Por último, veamos en **Podman Desktop**. Abre la sección *Pods* y verifica que **myapppod** aparece, con 2 contenedores (`myapppg` y `myapi`) corriendo. En *Containers*, deberías verlos listados también. Puedes inspeccionar la CPU/mem que consumen (FastAPI con nada de carga debería consumir muy poco CPU; memoria tal vez ~50-100MB por el Python runtime, PostgreSQL quizá unos 100MB).
  
  También podrías usar Podman Desktop para, por ejemplo, reiniciar el contenedor de la app si quisieras (Stop en `myapi` y luego Start de nuevo) o ver sus logs en la interfaz.

¡Felicidades! Has creado y ejecutado una mini aplicación de múltiples componentes usando Podman. Recapitulando el logro:
- Construimos una imagen de nuestra aplicación FastAPI con un Containerfile personalizado.
- Ejecutamos un servicio de base de datos usando una imagen existente.
- Agrupamos ambos en un pod para facilitar su comunicación y gestión.
- Expusimos los puertos necesarios al host para poder interactuar con la API (y DB si fuera necesario).
- Probamos que la API responde y puede acceder a la base de datos.
- Utilizamos Podman Desktop para visualizar y administrar los recursos de forma gráfica.

### 8.5. Limpieza y consideraciones finales

Para cerrar la práctica, puedes detener y eliminar el pod `myapppod` cuando ya no lo necesites:

```bash
podman pod stop myapppod
podman pod rm myapppod
```

Esto apagará tanto la app como la base de datos y removerá el pod. Las imágenes `fastapi-app:1.0` y `postgres:15` permanecerán en tu sistema local hasta que decidas borrarlas (`podman rmi fastapi-app:1.0 postgres:15` si quisieras).

En un contexto real, podrías extender esta mini app: por ejemplo añadiendo más endpoints, modelos de datos, incluso otro microservicio en el pod (aunque usualmente en arquitectura de microservicios cada servicio tendría su propio pod, aquí solo juntamos API+DB por conveniencia). También podrías usar **volúmenes** de Podman para persistir los datos de PostgreSQL (ahora mismo, la DB se guarda dentro del contenedor; si eliminas el contenedor, los datos se pierden. Un volumen montado permitiría que los datos sobrevivan). Podman maneja volúmenes de forma similar a Docker (`podman volume create` y `-v volume_name:/path` en run).

Otro punto: **Kubernetes YAML**. Podman permite generar un manifiesto YAML de Kubernetes desde un pod existente con `podman generate kube <pod>` que produce la especificación (Deployment/Pod/Service) correspondiente. Esto es útil si deseas migrar lo que has probado localmente a un cluster real. Podman Desktop incluso tiene un botón "Play Kubernetes YAML" para aplicar un YAML (aunque localmente lo ejecutaría en Podman, o en Kind si lo tienes configurado).

**Resumen**: En esta guía de ~2 horas, cubrimos la teoría de contenedores, instalamos Podman y Podman Desktop en Windows, construimos contenedores simples, manejamos contenedores y pods con Podman CLI, visualizamos recursos con Podman Desktop, entendimos la importancia de rootless, simulamos un pipeline CI/CD, y finalmente desarrollamos una pequeña aplicación FastAPI con su base de datos en contenedores, todo ello paso a paso y con explicaciones detalladas. 

Esperamos que esta práctica te haya dado una comprensión sólida de cómo trabajar con contenedores usando Podman, preparándote para usar estas herramientas en proyectos futuros y explorando alternativas modernas a Docker en entornos de desarrollo y despliegue. ¡Happy Container Hacking! 🐳 (bueno, o mejor dicho 🐋🚫😄, ¡porque con Podman ya no necesitamos al daemon de Docker!)