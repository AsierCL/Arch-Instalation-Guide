# Dual Boot de Windows + Arch Linux

### Preparación previa de Windows

Para hacer un dual boot, la recomendación desde la wiki de Arch Linux, es instalar primero Windows, y despues Arch.

Antes de empezar con la instalacion en si misma, vamos a hacer un par de ajustes en Windows para evitar errores futuros. El primero, es abrir un CMD como administrador y pegar el siguietne comando, que se puede encontar buscando Arch Time, primera entrada de la wiki de Arch, en el apartado 4.1.

```
reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation" /v RealTimeIsUniversal /d 1 /t REG_DWORD /f
```
Este comando permite que la hora del equipo sea gestionada por Arch, dado que de lo contrario, es probable que se produzcan desajustes en ella.

El segundo paso, será ir a las opciones de energía, y desactivar el arranque rápido y la hibernación en windows.

Cuando windows se instala, crea 3 particiones.
- La primera, de unos 200MB, es la partición de arranque de Windows, la cual reutilizaremos para nuestra particion de arranque con grub.
- La segunda, es el disco duro C:, donde esta todo lo relacionado con windows (SO, archivos, etcetc)
- La tercera, de unos 700MB, es la particion de recuperación de Windows.

Lo primero que debemos hacer, es desde el particionador de discos de Windows, reducir la segunda partición, para tener sitio para Arch.
Deberia quedar un espacio como **No asignado**, que será el que usemos para la raíz de Arch.

Ahora podemos entrar con el sistema live de Arch, siguiendo las indicaciones generales, hasta llegar a el particionado de discos, y seguir esta guía.


### 3.- Particionado del disco

Ejecutamos el comando lsblk para ver las particiones que existen en el disco.
Es probable que tengamos un nuevo esquema con 4 particiones, y dos espacios de memoria libres, de la siguiente forma:
- Arranque de Windows (la reutilizaremos)
- Reservado para datos de windows
- Datos de Windows (C:)
- Espacio libre (el que usaremos para instalar Arch)
- Entorno de recuperacion de Windows
- Espacio libre (se genera al reducir volumenes, se pierde (un par de megas unicamente))

> [!CAUTION]
> Los cambios hechos en este apartado son irreversibles, debes estar seguro de lo que estas a punto de hacer.

Abrimos nuestro disco duro con cfdisk /dev/sda (o el nombre que tenga tu disco duro, nvme0n1, vda, hda...)

Para hacer el tutorial más breve, no haremos partición de swap, pero si se deseara, se puede hacer de la forma indicada en la guía general.

Damos enter en el espacio libre en el cual vamos a instalar Arch, y enter otra vez para confirmar el tamaño de la nueva partición (todo el disponible)

Para salir y guardar, vamos a write y escribimos "yes". Por último, le damos en Quit.

Ahora si ejecutamos _fdisk -l_ debemos ver las 5 particiones (6 si hay swap)

Debemos darle formato a nuestra nueva partición:
```
mkfs.ext4 /dev/sda5
```

Montamos nuestra partición root
```
mount /dev/sda5 /mnt
```

Y montamos la particion de arranque
```
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```

Ahora, prosigue con la guia general, hasta llegar a la parte de instalacion de grub.


### 7.- Instalación de GRUB
Si hacemos un _ls /boot/efi/EFI, debemos ver las carpetas Boot y Microsoft.
```
pacman -S grub efibootmgr os-prober ntfs-3g
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Arch
grub-install --target=x86_64-efi --efi-directory=/boot/efi --removable
```
Si repetimos el ls, deberiamos ver ahora una nueva carpeta llamada Arch.

Debemos editar el archivo /etc/default.grub, y buscar la linea _GRUB_DISABLE_OS_PROBER=false_, y descomentarla.

Ahora, ejecutamos
```
os-prober
grub-mkconfig -o /boot/grub/grub.cfg
```
Si el os-prober no muestra nada, puede ser debido a estar corriendo en un entorno chroot, simplemente, repetimos este paso al reiniciar el ordenador y entrar sin el pendrive.
```
exit
umount -R /mnt
reboot now
```
Ya podemos proseguir con la guia general.

### 10.- Instalación de mi entorno

```
pacman -S bspwm polybar sxhkd kitty picom dunst rofi keyd feh brightnessctl bluez-utils openssh p7zip pavucontrol playerctl pulseaudio rclone tree usbutils wakeonlan xclip xorg zsh bat lsd

git clone https://github.com/AsierCL/Dotfiles
```
