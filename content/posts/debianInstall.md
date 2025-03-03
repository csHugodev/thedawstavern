---
date: '2025-03-02T21:27:16-06:00'
draft: false
title: 'Guía completa: Cómo instalar Debian 12 con KDE Plasma sin aplicaciones preinstaladas'
tags: ["Instalación", "Debian", "Linux", "Tutorial", "Configuración", "KDE Plasma"]
categories: ["Linux", "Tutoriales"]
---

**Debian es mi distro de favorita para el día a día.** Si quiero algo estable y sin dolores de cabeza, siempre vuelvo a Debian. Pero hay algo que me molesta cada vez que hago una instalación nueva: los escritorios como **KDE Plasma** o **GNOME** vienen cargados con un montón de aplicaciones que nunca voy a usar.

A mucha gente no le importa, porque al final del día no afectan el rendimiento ni nada, pero yo prefiero una instalación más limpia, con lo justo y necesario, para después agregar solo lo que realmente voy a usar. Si te pasa lo mismo, en esta guía te voy a mostrar cómo instalar **Debian 12 Bookworm con KDE Plasma** (mi escritorio favorito), pero sin toda esa suite de aplicaciones preinstaladas.

El objetivo aquí es tener **KDE Plasma funcionando con lo mínimo necesario**, sin todo ese software extra que viene por defecto. Solo instalaremos lo esencial para que el escritorio corra bien, incluyendo las herramientas internas que necesita, pero nada más.

Lo único que necesitas saber es cómo instalar Debian de la forma habitual: crear tu USB booteable, entrar al instalador, hacer tus configuraciones básicas, etc. Si ya tienes eso cubierto, todo lo demás lo veremos paso a paso en esta guía. De todas formas, esta guía sigue siendo para usuarios nuevos, así que explicaré algunas cosas básicas de Linux.

**Nota:** Esto está basado en **Debian 12 Bookworm**, así que si planeas hacerlo en otra versión, tenlo en cuenta.

### **Paso 1: Instalación de Debian (sin escritorio)**

Empieza instalando **Debian 12 Bookworm** con la ISO que prefieras, como lo harías normalmente. Avanza en el proceso hasta llegar a la parte donde eliges el entorno de escritorio.

Aquí es donde hacemos el truco: **NO selecciones ningún escritorio.** Desmarca todas las opciones y deja marcada únicamente la casilla de **"standard system utilities"**. Esto nos asegurará una instalación mínima con las herramientas básicas del Debian, pero sin un entorno gráfico preconfigurado.

Una vez hecho esto, continúa con la instalación hasta que termine y reinicia el sistema en tu nueva instalación.

**Nota 1:** Esta instalación dejará Debian sin entorno gráfico, así que cuando inicies el sistema, lo primero que verás será una terminal en modo texto. No te preocupes, esto es normal y en los siguientes pasos configuraremos el entorno gráfico a nuestro gusto.

**Nota 2:** Algo que **no se instalará** automáticamente es **sudo**. Este comando, cuyo nombre significa _"superuser do"_ (_"superusuario hace..."_), permite ejecutar comandos administrativos desde una cuenta de usuario normal, como `sudo apt update`. Al no estar instalado, por ahora solo podremos ejecutar comandos como superusuario iniciando sesión con `su`. Más adelante nos encargaremos de instalar y configurar `sudo`.

### **Paso 2: Sudo**

Una vez que inicies tu nueva instalación de Debian, lo único que verás será una consola en modo texto. Aquí te pedirá que ingreses tu nombre de usuario y contraseña, según lo que configuraste durante la instalación.

Ahora necesitamos darle permisos de **root** a tu usuario, pero antes debemos convertirnos en **superusuario**. Normalmente, podríamos hacerlo con el comando `su`, pero hay un detalle importante: si lo hacemos así, **no podremos ejecutar ciertos comandos del sistema**, como `usermod`, que es el que necesitamos para asignar permisos de administrador.

¿Por qué pasa esto? Porque algunos comandos importantes están ubicados en `/usr/sbin/`, un directorio que **solo está en el `$PATH` de root**, no en el de un usuario normal. Si usamos `su` sin más, el sistema mantiene el `$PATH` del usuario original, lo que nos impide ejecutar esos comandos.

La solución es usar **`su -` en lugar de `su`**. Ese pequeño guion hace la diferencia, ya que inicia la sesión de root con su entorno completo, incluyendo el `$PATH` correcto.

En resumen, ejecuta: `su -`. 

Te pedirá la contraseña de root (la que configuraste en la instalación). Una vez dentro, ya estarás en modo superusuario con acceso total al sistema.

Ahora que ya eres **root**, lo primero que haremos es instalar `sudo`. Para hacerlo, simplemente ejecuta: `apt install sudo`. Esto instalará `sudo` y todas sus dependencias, permitiéndonos ejecutar comandos administrativos desde un usuario normal sin necesidad de cambiar a root cada vez.

### **Paso 3: Agregar tu usuario al grupo sudo**

Ahora que `sudo` ya está instalado, necesitamos agregar nuestro usuario a la lista de usuarios con permisos de superusuario. Para hacer esto, ejecuta el siguiente comando (sustituyendo `tuusuario` por tu nombre de usuario real):
`usermod -a -G sudo tuusuario`

**¿Qué hicimos aquí?**

Seguramente alguna ves viste un error que dice algo como `"Username is Not in the Sudoers File"`. Esto es un error que nos dice que nuestro usuario no esta en la lista de usuarios que pueden ejecutar comandos de superusuario (la lista de usuarios sudo). Esta lista se encuentra en el archivo `sudoers`. **Nunca edites este archivo directamente**, ya que hacerlo mal puede bloquear el acceso al sistema. Esta caracteristica de tener una lista de usuarios, viene del paquete `sudo` que instalamos anteriormente.

Lo que hicimos con el comando anterior fue **agregar tu usuario al grupo sudo**, lo que le da permisos administrativos.

Desglose del comando:
- `usermod` -> Es el comando para modificar cuentas de usuario en Linux.
- `-a` (append) -> Añade el usuario a un grupo sin eliminar los grupos a los que ya pertenece.
- `-G sudo` -> Especifica que el usuario debe agregarse al grupo sudo.
- `tuusuario` -> Es el nombre del usuario al que se le otorgarán estos permisos.

Una vez agregado nuestro usuario a `sudo`, debemos cerrar sesión para que el cambio sea efectuado. Empieza por salir de root con `exit` para regresar a tu usuario normal. Una vez en tu usuario puedes ejecutar `logout` para cerrar sesión. (O puedes intentar hacer `logout` desde root, solo asegurate de salir por completo de cualquier usuario).

Vuelve a iniciar sesión, y ejecuta algo como `sudo apt update` para ver que este funcionando tu usuario como superusuario. Si no ves errores y el sistema descarga correctamente la lista de paquetes, significa que ya puedes usar `sudo` sin problemas.

### **Paso 4: Configurar internet para KDE**

Para que la configuración de red de **KDE Plasma** funcione correctamente, necesitamos instalar **Network Manager**, que es el gestor de red que KDE usa de forma nativa para integrarse con sus menús y configuraciones.

Para instalarlo, ejecuta:
`sudo apt install network-manager`

**Limpiando la configuración de red anterior**

Debian usa un archivo en `/etc/network/interfaces` para configurar manualmente las interfaces de red, pero como vamos a usar **Network Manager**, necesitamos borrar (o comentar) su contenido.

Puedes editarlo con el editor que prefieras, pero si quieres usar **nano**, ejecuta:
`sudo nano /etc/network/interfaces`

Asegúrate de abrirlo con `sudo`, porque de lo contrario no podrás modificarlo.

Dentro del archivo, elimina o comenta (agregando `#` al inicio de cada línea) todo su contenido. Luego, guarda los cambios con `CTRL + O`, y luego `CTRL + X` para salir.

**Reiniciar el sistema**
Para que los cambios surtan efecto, reinicia el sistema con: `sudo reboot`.

**Nota:** Cuando reinicies, **no tendrás conexión a internet**, pero en el siguiente paso la configuraremos con comandos.

### **Paso 5: Conectarte a internet**

Primero, verifica que Network Manager se esté ejecutando, con el comando: `systemctl status NetworkManager`.

Debería estar activo (aparece en letras verdes), pero si no, lo puedes iniciar con el comando: `sudo systemctl start NetworkManager`. Y esto tambien deberia estar configurado por defecto, pero si te quieres asegurar de que se inicie automaticamente, ejecuta: `sudo systemctl enable NetworkManager`.

**Conectarse a una red WiFi**

Ahora, para ver las conexiones WIFI disponibles, usa:
`nmcli device wifi list`.

Una vez que encuentres la red a la que quieres conectarte, usa el siguiente comando (sustituyendo `"NombreDeRed"` y `"CONTRASEÑA"` con los valores correctos):
`nmcli device wifi connect "NombreDeRed" password "CONTRASEÑA"`
Ejemplo:
`nmcli device wifi connect "Infinitum1234" password "clave123"`

Si estás usando una conexión **Ethernet**, generalmente se conecta automáticamente. Pero si necesitas activarla manualmente, usa:
`nmcli connection up id "nombre_de_la_conexión"`

**Verificar la conexión a internet**

Para confirmar que estás conectado, puedes ejecutar:
`nmcli connection show --active`
O simplemente hacer un **ping** a Google:
`ping -c 4 www.google.com`

### **Paso 6: Instalar KDE Plasma en Debian (Sin Apps Innecesarias)**

¡Por fin! Es momento de instalar el entorno de escritorio **KDE Plasma** sin las aplicaciones que vienen por defecto.

Para instalar solo lo esencial, ejecuta:
`sudo apt install kde-plasma-desktop`.

Este comando instalará **el núcleo de KDE**, sus **dependencias esenciales** y las **bibliotecas necesarias** para que el escritorio funcione sin problemas, pero sin el montón de software adicional que normalmente trae la instalación estándar de KDE.

Una vez que termine la instalación, **reinicia el sistema** con:
`sudo reboot`

**¿Y ahora qué?**

En teoría, con esto ya tienes un escritorio KDE Plasma **limpio y funcional**. Sin embargo, si quieres hacer algunas configuraciones extra que siempre hago, puedes leer los siguientes pasos.

### **Paso 7: Desinstalar aplicaciones por defecto**

Aunque instalamos **KDE Plasma** sin aplicaciones adicionales, todavía hay algunos programas que vienen por defecto, como **Konqueror** (un navegador web) y **Zutty** (una terminal).

Si no los usas y prefieres instalar tus propias herramientas, puedes eliminarlos con este comando:
`sudo apt purge konqueror* zutty*`
**¿Por qué el `*`?**  
El asterisco (`*`) le dice a `apt` que elimine **todo lo relacionado** con esos programas, incluyendo archivos de configuración.

Después de eliminar las apps, es buena práctica limpiar el sistema con:
`sudo apt autopurge`
Esto eliminará cualquier paquete sobrante que ya no sea necesario.

### **Paso 8: Instalar aplicaciones**

Este paso es completamente opcional, pero aquí van algunas aplicaciones que **siempre instalo** en una nueva instalación de Debian con KDE Plasma. Obviamente, puedes elegir las que más te gusten.

**Instalación de Firewall**
```
sudo apt install ufw plasma-firewall
sudo ufw enable
```

**Soporte para impresoras**
```console
sudo apt install cups print-manager
sudo usermod -a -G lpadmin tuusuario
```

**Instalar Flatpak** - Para Obsidian y Zen Browser
```console
sudo apt install flatpak
sudo apt install gnome-software-plugin-flatpak
sudo apt install plasma-discover-backend-flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

**Habilitar Splash Screen**
`sudo nano /etc/default/grub`
Editar la línea correspondiente:
`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"`
Si no hay dualboot y no necesitas ver el grub al iniciar, editar tambien:
`GRUB_TIMEOUT=0`

#### Mis aplicaciones esenciales

- `firefox-esr` → Navegador web estable y mantenido a largo plazo.
- `zen-browser` → Navegador web principal.
- `kitty` → Terminal.
- `obsidian` → Tomar notas en Markdown.
- `pycharm` → Editor de código para proyectos Python.
- `vscode` → Editor de código y texto en general.
- `spectacle` → Capturar pantalla en KDE.

