# Instalación de ARCH LINUX

A continuación se muestran los pasos necesarios para instalar el sistema de Arch Linux en un equipo manualmente, a la vez que se explica brevemente para que sirve cada paso. 

### Requisitos previos

Antes de empezar con la instalación, necesitaremos un pendrive flasheado con la imagen de Arch Linux. Esta puede ser encontrada en la web oficial de [Arch Linux](https://archlinux.org/download/).

Para ello, debemos usar un equipo funcional, en el que descargaremos algún software para flashear la imagen .iso. En lo personal, uso [Rufus](https://rufus.ie/es/), pero sirve cualquier otro como [Balena Etcher](https://etcher.balena.io/).

Una vez flasheada la unidad USB, conectamos la unidad flash al ordenador, y en el programa que vayamos a usar, seleccionamos la imagen iso y el pendrive, y dejamos todas las opciones predeterminadas. Una vez que termine, podremos apagar y empezar con la instalación.

### 1.- Arrancamos desde el entorno live de Arch

Para este primer paso, debemos conectar el pendrive al ordenador en el que vamos a instalar Arch, con el equipo apagado. Una vez conectado, debemos encender el ordenador, y durante el arranque, presionar repetidamente la tecla de selector de arranque. Esto depende del fabricante de la placa base, pueden consultarlo en la web de este, pero las más comunes son F2, F9 y F12. Ya en el gestor de arranque, debemos seleccionar nuestra unidad USB. Saldrá la pantalla de instalación de Arch, y seleccionamos la primera opción.

La unidad USB contiene la información necesaria para montar un "sistema funcional" en la memoria ram del ordenador, de forma que podremos cargar este sistema siempre que necesitemos, y arrancará el 100% de las veces. Esto es útil si a futuro el sistema se corrompe por algún cambio erróneo que impide arrancar el sistema operativo, ya que con este método podremos entrar desde el entorno live para corregirlo, o para recuperar información que tengamos en el sistema.

### 2.- Preparación del entorno live

Lo primero que veremos una vez cargue el entorno live, será una consola de comandos, en la que somos usuario root. Según que teclado estés usando, podrías notar que los símbolos como guiones, barras o comillas no se corresponden con tu teclado. Para ello cargaremos la distribución de teclado que uses, en mi caso "es". El comando es el siguiente:
```
loadkeys es
```
Ahora vamos a conectar el dispositivo a internet. Lo más recomendable es tenerlo conectado por un cable LAN, ya que automáticamente debería detectar la interfaz de red, y el router automáticamente asignaría una dirección IP por DHCP. Para conectarnos por Wifi, usaremos el siguiente comando:

Entramos en el programa iwd (iNet wireless daemon)
```
iwctl
```
Una vez ejecutemos el comando, aparecerá una nueva interfaz. Listamos nuestras interfaces de red:
```
device list
```
En mi caso, la interfaz de red es wlan0, pero puede variar en función de la marca y modelo del dispositivo.

Para escanear las redes disponibles, ejecutamos dos comandos:
```
station wlan0 scan
station wlan0 get-networks
```
y nos conectamos a la red correspondiente con el comando:
```
station wlan0 connect NOMBREDELARED
```

Por último, verificamos la conectividad, con un el comando, el cual debe devolver exitosamente 4 paquetes.
```
ping google.com -c 4
```

### 3.- Particionado del disco

Antes de empezar el particionado, debemos conocer como debe ser la estructura de las particiones, y el nombre de nuestro disco duro.
- Particion 1: Será donde instalaremos GRUB para poder arrancar cualquier sistema del ordenador. El tipo de partición debe ser EFI System, y debe de estar formateada en FAT32.
- Partición 2: Será la partición que usaremos para el swap. El tipo de partición es Linux filesystem, y debe estar formateada en formato swap.
- Partición 3: Será la partición principal, en la cual instalemos Arch Linux. El tipo de partición es Linux filesystem, y debe estar en ext4.

Para saber el nombre de nuestro disco, simplemente ejecutamos el comando _cfdisk_, y arriba de todo, veremos el nombre del disco de la siguiente forma /dev/NOMBRE. En mi caso, sda.

#### DISCLAIMER. Los cambios hechos en este apartado son irreversibles, debes estar seguro de lo que estas a punto de hacer.

Si tenemos particiones ya creadas, que será lo habitual, saldremos de _cfdisk_, y ejecutaremos el siguiente comando para eliminarlas:
```
sgdisk --zap-all /dev/sda
```
Ahora, podemos comenzar con la creación de las particiones. Volvemos a ejecutar _cfdisk_, y seleccionamos GPT.

En el menú de cfdisk, nos desplazaremos con las flechas hasta el apartado new, y creamos la primera partición, de 300M. (sda1).

Movemos el cursor hacia abajo para ubicarnos encima de el espacio libre, y volvemos a darle a write, para crear la segunda partición. Para esta partición, recomiendo poner de tamaño referencia el 50% de nuestra memoria RAM. Yo tengo 8GB de RAM, por lo que asignaré 4GB de swap. (En cfdisk debes poner 4G, sin la B de byte). (sda2)

Repetimos el paso para la última partición, y no modificamos el tamaño por defecto, para que ocupe todo el espacio restante del disco duro. (sda3). 

Ahora modificamos el tipo de cada una de ellas, seleccionándolas con arriba y abajo, y usando Type. La primera debe ser EFI System, la segunda Linux swap, y la tercera Linux filesystem.

Para salir y guardar, vamos a write y escribimos "yes". Por último, le damos en Quit.

Ahora si ejecutamos 
```
fdisk -l
```
debemos ver las 3 particiones (y otro disco b, que es el pendrive del entorno live).

Vamos con el formato de cada partición. Para la primera, en mi caso (/dev/sda1), ejecutamos
```
mkfs.fat -F32 /dev/sda1
```
Para la segunda,
```
mkswap /dev/sda2
swapon /dev/sda2
```
Para la última
```
mkfs.ext4 /dev/sda3
```

Ahora, debemos montar las particiones, usando el comando _mount_. A efectos prácticos, debemos entender que el directorio /mnt del entorno live es equivalente a la que será la raíz del sistema una vez iniciado.

La primera partición que debemos montar, será la raíz o root, que corresponde a la partición sda3. Para ello, ejecutamos el comando
```
mount /dev/sda3 /mnt
```

Para montar la partición de boot, debemos crear la carpeta /boot/efi, y luego montarla. Esta ruta es importante, ya que será donde más adelante instalaremos el GRUB en nuestro sistema.
```
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```


### 4.- Instalación del sistema

Para instalar todos los archivos necesarios, usaremos el comando _pacstrap_. Todo lo que instalemos debe de estar en /mnt, (será la raíz del sistema). Para ello, ejecutamos:
```
pacstrap /mnt base base-devel linux linux-firmware linux-headers mkinitcpio nano
```

##### Utilidades de cada paquete:
- base: Incluye los componentes esenciales necesarios para que nuestro SO funcione. Entre ellos, tenemos Bash, que es la shell por defecto, pacman, que es el gestor de paquetes de arch, o systemd que es el sistema de inicio y gestor de servicios
- base-devel: Incluye los componentes básicos para el desarrollo y la compilación de código, entre ellos Gcc, Gdb o Make. Esto es esencial para instalar aplicaciones en las que tengamos que compilar nosotros el código fuente.
- linux: Instala el kernel de linux estable. El archivo principal es vmlinuz-linux
- linux-firmware: Descarga una serie de "drivers genéricos" que permiten que componentes de hardware como las tarjetas de red funcionen adecuadamente.
- linux-headers: Instala los archivos necesarios para complilar módulos del kernel y módulos que interaccionan directamente con el. Estos archivos son, principalmente, archivos de encabezado .h y makefiles o scripts de construcción.
- nano: Un editor de texto básico y sencillo que funciona en la terminal. No es imprescindible, ya que en base se instala vi

Una vez instalado todo esto, podemos hacer una serie de comprobaciones. Si listamos los paquetes de nuestra futura raíz:
```
ls /mnt
```
donde antes no había nada, ahora observamos la estructura de archivos de linux.
Si listamos la carpeta /boot:
```
ls /mnt/boot
```
vemos los archivos del kernel que usará GRUB para arrancar el sistema.

### 5.- Generar fstab

Hasta ahora, hemos estado trabajando encima de nuestras 3 particiones (sda1, sda2, sda3). Si recordamos, para "colocar cada una en su sitio", hemos usado el comando _mount_.

Para que el SO sepa donde montar cada partición cuando arranque, sin necesidad de que nosotros se lo indiquemos, cuenta con un archivo en la ruta /etc, llamado fstab. En el, puedes escribir a mano como debe montarse, pero es una tarea un poco tediosa, por lo que usaremos una herramienta incluida en el paquete "util-linux", que se incluye en el grupo de paquetes "base".
Para ello, escribimos:
```
genfstab -p /mnt >> /mnt/etc/fstab
```
Una vez ejecutado, podremos ejecutar 
```
cat /mnt/etc/fstab
```
y veremos las 3 particiones de nuestro disco y el punto de montaje de cada una.

### 6.- Configuración del sistema

Hasta ahora, estábamos trabajando desde nuestro arch live. Ahora, ya estamos listos para entrar al sistema "real".

Para ello, usamos
```
arch-chroot /mnt
```
Vemos que el prompt (el inicio de nuestra linea de comandos) ha cambiado, asi como si hacemos un _ls /mnt_ no tenemos el sistema.

Ahora, procedemos a hacer una serie de configuraciones, empezando por el idioma.

```
nano /etc/locale.gen
```
y en el editor, tenemos que descomentar la línea del idioma que queramos. En mi caso, descomento es_ES UTF-8 UTF-8. Las primeras 2 letras son el idioma, y las siguientes 2 el país. Recomiendo escoger siempre la que pone UTF-8.

Guardamos con ctrl+o, damos a enter, y salimos con ctrl+x.

Ahora ejecutamos los dos siguientes comandos. El segundo depende del idioma que hayas descomentado.
```
locale-gen
export LANG=es_ES.UTF-8
echo "LANG=es_ES.UTF-8" > /etc/locale.conf
```

Ahora, establecemos la zona horaria:
```
ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
hwclock --systohc
```
Sustituye Europe/Madrid por tu continente/capital.

Configuramos la distribución de teclado:
```
echo KEYMAP=es > /etc/vconsole.conf
```
Definimos el nombre de nuestro equipo (reemplazar NombreDelEquipo):
```
echo NombreDelEquipo > /etc/hostname
```

#### Configuración de usuarios.

Modificamos la contraseña del usuario root:
```
passwd root
```

Y creamos nuestro usuario, lo agregamos al grupo users, y le damos permisos para usar _sudo_ añadiéndolo al grupo wheel, y le ponemos contraseña:
```
useradd -m -g users -G wheel -s /bin/bash NombreUsuario
passwd NombreUsuario
```

Ahora, para poder usar _sudo_ con el nuevo usuario, editaremos el siguiente archivo:
```
nano /etc/sudoers
```
Justo en la línea debajo de donde pone root ALL=(ALL:ALL) ALL
, escribimos lo mismo, pero cambiando root por el nombre del usuario que acabas de crear. Guardamos y cerramos como en pasos anteriores.

#### Configuración de red

Para configurar la conexión red de nuestro equipo, debemos instalar una serie de paquetes.
```
pacman -Syu
pacman -S dhcp dhcpcd networkmanager
systemctl enable dhcpcd NetworkManager
```

##### Utilidades de cada paquete:
- dhcp: Proporciona tanto el cliente como el servidor DHCP. En resumen, permite obtener direcciones IP de forma automática al conectarse a una red.
- dhcpcd: Es el Daemon, es decir, se ejecuta en segundo plano, para mantener la configuracion de red.
- Networkmanager: Es un gestor de red avanzado y versatil, que proporciona tanto una GUI como una CLI para configurar la red.

### 7.- Instalación de GRUB
Ahora vamos a instalar grub para permitir a nuestro sistema UEFI arrancar. Para ello, ejecutamos:
```
pacman -S grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Arch
grub-install --target=x86_64-efi --efi-directory=/boot/efi --removable
grub-mkconfig -o /boot/grub/grub.cfg
```
Si se modifica algún parámetro en el GRUB, se debe ejecutar el último comando de la lista. Este comando actualiza el archivo de configuración de GRUB con los cambios recientes, como nuevos kernels o sistemas operativos detectados.

En este punto, estamos listos para reiniciar el ordenador sin el pendrive con el entorno live.

```
exit
umount -R /mnt
swapoff /dev/sda2
reboot now
```
Ya podemos desenchufar el pendrive y arrancar nuestro sistema.


### 8.- Instalación de paquetes básicos

Al reiniciar, nos logueamos con el usuario root, para seguir instalando paquetes y no necesitar el comando _sudo_.

Procedemos a instalar paquetes que casi al 100% de las veces vamos a necesitar.
```
pacman -Syu
pacman -S neofetch lsb-release curl wget git xf86-video-fbdev xf86-video-vesa
```

### 9.- Instalación del entorno gráfico
Finalmente ya podríamos instalar un entorno gráfico y tener un dispositivo completamente funcional. Para ello, debemos instalar el entorno gráfico (Desktop enviroment) y su correspondiente administrador de pantalla (Display manager). A veces, instalando el entorno, ya viene su administrador de pantalla.

La función del administrador de pantalla es ofrecer el inicio de sesión, y en caso de un login exitoso, lanzar el entorno gráfico. También gestiona las sesiones y los cambios de usuarios.

Para este tutorial, usaremos el entorno gnome, ya que es sencillo de instalar. Para ello, ejecutamos los siguientes comandos:
```
pacman -S gnome
```
y aceptamos todo;
```
systemctl enable gdm.service
```
activamos el administrador de pantalla.

Ahora podemos reiniciar el sistema, y deberíamos tener el entorno listo y funcionando.
```
reboot now
```
