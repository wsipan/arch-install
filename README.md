# Arch Linux | Guía de instalación

Esta es una guía para instalar Arch Linux en el modo de arranque UEFI.

## Pre instalación

A partir de acá asumiré que ya se encuentran en el prompt de instalación.

### Chequear el modo de arranque

Para comprobar si es un sistema UEFI/EFI, ejecutar:
```sh
ls /sys/firmware/efi/efivars
```
Si el directorio no existe, problablemente es un sistema BIOS. (No UEFI)

### Conexión a internet
Para comprobar la conexión a internet, ejecuta:
```sh
ping -c 4 archlinux.org
```

### Cambiar la distribución del teclado
Si tenemos la distribución de latam, ejecutamos:
```sh
loadkeys la-latin1
```
Si tenemos la distribución española, ejecutamos:
```sh
loadkeys es
```

### Cambiar el idioma del live environment
```sh
echo "es_ES.UTF-8 UTF-8" > /etc/locale.gen
locale-gen
export LANG=es_ES.UTF-8
pacman -Sy
```

### Actualizar el reloj del sistema
```sh
timedatectl set-ntp true
```

### Particionado
Para listar los discos disponibles, ejecutamos:
```sh
fdisk -l
```
Mi disco virtual tiene `100GB`, necesitamos crear las particiones para la instalación de Arch.

#### Tabla de particiones

| Nombre | Montar | Tamaño |  Tipo |
| :----: | :----: | :----: | :---: |
| /dev/sdax | `/boot/efi` | 150M | Sistema EFI |
| /dev/sdax | `swap` | 4GB | Linux swap |
| /dev/sdax | `/` |  Espacio restante | Sistema de ficheros Linux |

#### Creación de particiones

Formateamos nuestro disco duro.
> [!CAUTION]
> No ejecutar el siguiente comando si queremos instalar un `DUAL BOOT`.
```sh
sgdisk --zap-all /dev/sda
```

Para crear las particiones usaremos `cfdisk`, ejecutaremos el siguiente comando y crearemos la tabla de particiones de la parte superior:
```sh
cfdisk /dev/sda
```

#### Formatear particiones
Una vez creadas las particiones, cada una (excepto `swap`) debe ser formateada con un sistema de ficheros apropiado, para eso ejecutamos:
```sh
mkfs.fat -F32 -n "boot-efi" /dev/sda1        #- UEFI partition
mkfs.ext4 -n "root" /dev/sda3                #- root partition
```

Para formatear la partición de intercambio, ejecutamos:
```sh
mkswap /dev/sda2
swapon /dev/sda2
```

#### Montar particiones
Montar la partición `root`:
```sh
mount /dev/sda3 /mnt
```
Montar la partición de arranque `UEFI`:
```sh
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```
---
## Instalación

### Instalar los paquetes base
```sh
pacstrap /mnt base base-devel linux linux-firmware linux-headers mkinitcpiio
```

### Generar el archivo `fstab`
```sh
genfstab -p /mnt >> /mnt/etc/fstab
```

### Cambiar raíz
Cambiamos la raíz a nuestro sistema:
```sh
arch-chroot /mnt
```

### Instalación de paquetes básicos
```sh
pacman -S nano man-db man-pages
```

### Cambiar el idioma del sistema
```sh
nano /etc/locale.gen

# Descomentar es_PE.UTF-8 UTF-8
locale-gen
export LANG=es_PE.UTF-8

# Indicar el idioma del sistema a la hora de iniciar
echo "LANG=es_PE.UTF-8" > /etc/locale.conf

# Indicar la layout del teclado sólo para la TTY
echo "KEYMAP=es" > /etc/vconsole.conf
```

### Cambiamos la zona horaria
```sh
ln -sf /usr/share/zoneinfo/America/Lima /etc/localtime
```
Sincronizamos la hora con el hardware:
```sh
hwclock -w
```

### Redes

#### Definir el nombre del equipo
```bash
echo "<Nombre Equipo>" > /etc/hostname
```

#### Instalar los servicios de Internet
```sh
pacman -S dhcp dhcpcd networkmanager iwd
```
Activamos los servicios:
```sh
systemctl enable dhcpd NetworkManager
```

### Creación de usuarios
#### Crear clave para el usuario administrador (root)
```bash
passwd
```

#### Crear usuario del sistema
```sh
useradd -m <Nombre Usuario>
```
Asignamos una contraseña al usuario creado:
```sh
passwd <Nombre Usuario>

```
Asignamos los grupos al usuario creado:
```sh
usermod -aG wheel,storage,audio,video <Nombre Usuario>
```
```sh
# Hacer que el usuario pueda utilizar sudo
nano /etc/sudoers

# Buscar "User privilege specification"
# Agregar "<Nombre Usuario> ALL=(ALL:ALL) ALL
```

### Instalación de GRUB
Instalamos los paquetes necesarios:
```sh
pacman -S grub efitbootmgr #os-prober
```
Instalamos GRUB en el sistema:
```sh
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Arch
grub-install --target=x86_64-efi --efi-directory=/boot/efi --removable
```
Generamos la configuración de GRUB:
```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

### Instalación de herramientas para el sistema
```sh
pacman -S neofetch lsb-release git wget curl unzip xclip zsh neovim wireless_tools
```

### Instalación de drivers de video
```sh
pacman -S xf86-video-vesa xf86-video-fbdev
```

### Instalación de drivers de controladores de audio
```sh
pacman -S alsa alsa-firmware alsa-utils alsa-plugins
```

### Reiniciar el sistema
```sh
exit
umount -R /mnt
#shutdown now
reboot
```
