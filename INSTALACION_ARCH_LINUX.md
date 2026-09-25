# Instalación de Arch Linux con LVM y entorno MATE

Esta guía documenta el proceso completo de instalación de **Arch Linux** en una máquina virtual, utilizando **LVM (Logical Volume Manager)** para la administración del almacenamiento y **MATE** como entorno gráfico de escritorio.

> **Nota:** En esta instalación el disco principal fue identificado como `/dev/sda`. Antes de ejecutar comandos sobre discos, se debe comprobar el nombre real del dispositivo con `lsblk`.

---

## 1. Preparación

Iniciar desde el ISO de Arch Linux en modo UEFI y realizar las comprobaciones iniciales:

```bash
# Verificar que el sistema arrancó en modo UEFI
ls /sys/firmware/efi/efivars

# Comprobar la conexión a Internet
ping -c 3 archlinux.org

# Identificar los discos disponibles
lsblk

# Sincronizar la hora del sistema
timedatectl set-ntp true
```

## 2. Particionamiento del disco

Abrir `cfdisk` sobre el disco principal:

```bash
cfdisk /dev/sda
```

Crear una tabla de particiones **GPT** con la siguiente distribución:

| Partición | Tipo GPT | Tamaño |
|---|---|---:|
| `/dev/sda1` | EFI System | 512 MiB |
| `/dev/sda2` | Linux filesystem | 1 GiB |
| `/dev/sda3` | Linux LVM | Resto del disco |

Al finalizar, seleccionar **Write**, confirmar con `yes` y seleccionar **Quit**.

## 3. Formateo de las particiones fuera de LVM

```bash
# Partición EFI
mkfs.fat -F32 /dev/sda1

# Partición /boot
mkfs.ext4 /dev/sda2
```

## 4. Creación de la estructura LVM

Crear el volumen físico y el grupo de volúmenes:

```bash
pvcreate /dev/sda3
vgcreate vgarch /dev/sda3
```

Crear los volúmenes lógicos:

```bash
lvcreate -L 15G -n root vgarch
lvcreate -L 2G -n swap vgarch
lvcreate -L 5G -n var vgarch
lvcreate -L 2G -n tmp vgarch
lvcreate -l 100%FREE -n home vgarch
```

Verificar la estructura creada:

```bash
lvs
```

Si se crea un volumen lógico con un nombre incorrecto, puede eliminarse antes de continuar. Por ejemplo:

```bash
lvremove /dev/vgarch/homre
lvs
lvcreate -l 100%FREE -n home vgarch
```

Formatear los volúmenes lógicos y preparar el área swap:

```bash
mkfs.ext4 /dev/vgarch/root
mkfs.ext4 /dev/vgarch/var
mkfs.ext4 /dev/vgarch/tmp
mkfs.ext4 /dev/vgarch/home
mkswap /dev/vgarch/swap
```

## 5. Montaje de sistemas de archivos

Montar primero la raíz y luego el resto de los sistemas de archivos:

```bash
mount /dev/vgarch/root /mnt

mkdir -p /mnt/boot
mount /dev/sda2 /mnt/boot

mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi

mkdir -p /mnt/var /mnt/tmp /mnt/home
mount /dev/vgarch/var /mnt/var
mount /dev/vgarch/tmp /mnt/tmp
mount /dev/vgarch/home /mnt/home

swapon /dev/vgarch/swap
```

Verificar los montajes:

```bash
findmnt
lsblk
```

## 6. Instalación base del sistema

Instalar el sistema base, kernel, firmware y herramientas necesarias:

```bash
pacstrap -K /mnt base linux linux-firmware lvm2 vim nano networkmanager sudo grub efibootmgr
```

## 7. Generación de fstab

Generar el archivo que define los sistemas de archivos que se montarán durante el arranque:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

Comprobar que aparezcan los puntos de montaje `/`, `/boot`, `/boot/efi`, `/var`, `/tmp` y `/home`.

## 8. Acceso al sistema instalado

```bash
arch-chroot /mnt
```

## 9. Configuración básica

Configurar la zona horaria de Ecuador:

```bash
ln -sf /usr/share/zoneinfo/America/Guayaquil /etc/localtime
hwclock --systohc
```

Habilitar la localización `en_US.UTF-8` en `/etc/locale.gen`, generar las localizaciones y establecer el idioma predeterminado:

```bash
sed -i 's/^#en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

Definir el nombre del equipo:

```bash
echo "archlvm" > /etc/hostname
```

Configurar `/etc/hosts`:

```text
127.0.0.1 localhost
::1       localhost
127.0.1.1 archlvm.localdomain archlvm
```

## 10. Configuración de mkinitcpio para LVM

Este paso es necesario para que el sistema pueda activar los volúmenes lógicos durante el arranque.

Editar `/etc/mkinitcpio.conf` y comprobar que el hook `lvm2` se encuentre antes de `filesystems` dentro de la línea `HOOKS`.

Después, regenerar la imagen initramfs:

```bash
mkinitcpio -P
```

## 11. Configuración de usuarios y sudo

Establecer la contraseña de `root`:

```bash
passwd
```

Crear el usuario `netadmin`, agregarlo al grupo `wheel` y establecer su contraseña:

```bash
useradd -m -G wheel -s /bin/bash netadmin
passwd netadmin
```

Abrir el archivo `sudoers` de forma segura:

```bash
EDITOR=nano visudo
```

Descomentar la siguiente línea:

```text
%wheel ALL=(ALL:ALL) ALL
```

## 12. Instalación de GRUB en UEFI

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

## 13. Habilitación de la red

```bash
systemctl enable NetworkManager
```

## 14. Salida, desmontaje y reinicio

```bash
exit
umount -R /mnt
swapoff -a
reboot
```

Antes del nuevo arranque, retirar o desconectar el ISO de instalación para que la máquina virtual arranque desde el disco instalado.

## 15. Actualización del sistema

Una vez iniciado Arch Linux:

```bash
sudo pacman -Syu
```

## 16. Instalación del entorno gráfico MATE

Instalar Xorg y el entorno de escritorio MATE:

```bash
sudo pacman -S xorg-server mate mate-extra
```

Instalar LightDM y su greeter GTK:

```bash
sudo pacman -S lightdm lightdm-gtk-greeter
```

Habilitar los servicios necesarios y reiniciar:

```bash
sudo systemctl enable lightdm
sudo systemctl enable NetworkManager
reboot
```

Después del reinicio deberá aparecer la pantalla de LightDM. Iniciar sesión con el usuario `netadmin` y seleccionar la sesión **MATE**.

## 17. Verificación de la instalación

Verificar discos, particiones y sistemas de archivos:

```bash
lsblk -f
findmnt
sudo fdisk -l
```

Verificar la estructura LVM:

```bash
pvs
vgs
lvs
```

Verificar específicamente `/home`:

```bash
findmnt /home
df -h /home
```

Verificar el sistema y el usuario:

```bash
uname -a
id netadmin
groups netadmin
pacman -Q | head
```

Verificar el gestor de inicio de sesión gráfico:

```bash
systemctl status lightdm
```

## 18. Ampliación posterior de `/home` con un segundo disco virtual

Después de agregar un segundo disco virtual a la máquina, identificar su nombre:

```bash
lsblk
```

En esta práctica, el segundo disco corresponde a `/dev/sdb`. Convertirlo en un Physical Volume:

```bash
pvcreate /dev/sdb
```

Agregar el nuevo PV al Volume Group existente:

```bash
vgextend vgarch /dev/sdb
```

Extender el Logical Volume de `/home` utilizando todo el espacio libre disponible:

```bash
lvextend -l +100%FREE /dev/vgarch/home
```

Redimensionar el sistema de archivos `ext4` para utilizar el espacio agregado:

```bash
resize2fs /dev/vgarch/home
```

Verificar el resultado:

```bash
df -h /home
lvs
```

> Es recomendable documentar la salida de `lvs` y `df -h /home` antes y después de la ampliación para evidenciar el incremento del almacenamiento.

---

## Resultado

La práctica permitió completar la instalación de **Arch Linux**, estructurar el almacenamiento mediante **LVM**, configurar el entorno gráfico **MATE** y ampliar posteriormente el volumen lógico destinado a `/home` mediante la incorporación de un segundo disco virtual.

##Evidencia
<img width="1305" height="871" alt="image" src="https://github.com/user-attachments/assets/5ebbd14a-099a-4778-9567-baba634f787b" />

