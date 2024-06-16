<div align="center">
<img src="res/archlinux.png" />
<h6>Csekme Krisztián</h6>
<h1>Arch linux telepítési kézikönyv</h1>
<h3>1.1 verzió</h3>
<h4>2024. június</h4>
</div>

- [Jogi nyilatkozat](#jogi-nyilatkozat)
- [1. Alaprendszer telepítése](#1-alaprendszer-telepítése)
  - [1.1 Partícionálás](#11-partícionálás)
    - [1.1.a Titkosított partícióval](#11a-titkosított-partícióval)
    - [1.1.b Normál módon](#11b-normál-módon)
  - [1.2 Telepítsük az alap rendszert](#12-telepítsük-az-alap-rendszert)
  - [1.3 Konfiguráljuk a rendszert](#13-konfiguráljuk-a-rendszert)
  - [1.4 Felhasználók beállítása](#14-felhasználók-beállítása)
  - [1.5 Initramfs és bootloader beállítása](#15-initramfs-és-bootloader-beállítása)
    - [1.5.a Titkosított systemd-boot partíció esetén](#15a-titkosított-systemd-boot-partíció-esetén)
    - [1.5.b Normal GRUB loader telepítés esetén](#15b-normal-grub-loader-telepítés-esetén)
  - [1.6 Gép újraindítása](#16-gép-újraindítása)
- [2. Felhasználói GUI telepítése](#2-felhasználói-gui-telepítése)
- [2.a GNOME telepítése](#2a-gnome-telepítése)
  - [2.a.2.1 SSH askpass](#2a21-ssh-askpass)
- [2.b KDE telepítése](#2b-kde-telepítése)
- [3 POST install](#3-post-install)
  - [3.1 AUR (yay) csomagkezelő](#31-aur-yay-csomagkezelő)
- [4 Pacman](#4-pacman)
  - [4.1 Néhány hasznos pacman parancs](#41-néhány-hasznos-pacman-parancs)
    - [4.1.1 Zst csomagok telepítése](#411-zst-csomagok-telepítése)
    - [4.1.2 Csomagok eltávolítása](#412-csomagok-eltávolítása)
- [5 ISSUES](#5-issues)
  - [5.1 Python issues](#51-python-issues)
    - [Error: To install Python packages system-wide, try 'pacman -S](#error-to-install-python-packages-system-wide-try-pacman--s)
  - [5.2 Postman](#52-postman)
  - [5.3 Visual studio](#53-visual-studio)
  - [5.4 Flameshot](#54-flameshot)
  - [5.5 Rendszer monitor](#55-rendszer-monitor)
- [6 Egyéb csomagok telepítése](#6-egyéb-csomagok-telepítése)

## Jogi nyilatkozat
<img width="64px" src="res/cc.svg" />
<h4>Nevezd meg! - Ne add el! 2.5 Magyarország</h4>

A következőket teheted a művel:
- szabadon másolhatod, terjesztheted, bemutathatod és előadhatod a művet
- származékos műveket (feldolgozásokat) hozhatsz létre

Az alábbi feltételekkel:
- Jelöld meg! A szerző vagy a jogosult által meghatározott módon kell meg-
jelölni a művet (pl. a szerző és a cím feltüntetésével).
- Ne add el! Ezt a művet nem használhatod fel kereskedelmi célokra

A szerzői jogok tulajdonosának írásos engedélyével bármelyik fenti feltételtől eltérhetsz.
A fentiek nem befolyásolják a szabad felhasználáshoz fűződő, illetve az egyéb jogokat

Ez a Legal Code (Jogi változat, vagyis a teljes licenc) szövegének közérthető nyelven
megfogalmazott kivonata.
Ez a kivonat a http://creativecommons.org/licenses/by-nc/2.5/hu/ oldalon is olvasha-
tó. A teljes licensz a http://creativecommons.org/licenses/by-nc/2.5/hu/legalcode olda-
lon érhető el.

E jegyzet hivatalos honlapjáról (https://github.com/csekme/arch_install_hun) tölthető le a mindenkori legfris-
sebb verzió.

Ez a jegyzet tartalmaz OpenAI ChatGPT válaszokat is, melyek helyességét magam ellenőriztem.

Ez a jegyzet forrásként használja továbbá az Arch linux [wiki](https://wiki.archlinux.org/) oldalát.


## 1. Alaprendszer telepítése

Bootolható USB létrehozása
```bash
$ sudo dd if=archlinux-2024.03.01-x86_64.iso of=/dev/sda bs=515b status=progress
$ sync
```

Miután egy USB, vagy egyéb forrásból származó telepítő segítségével betöltöttük az Arch Linux telepítőt, megkapjuk a konzolt. Elsőként célszerű a billentyűzet kiosztást beállítani hogy könnyű dolgunk legyen a gépeléssel.

Billentyűzet kiosztás beállítása

```zsh title="Magyar billentyűzet beállítása"  
$ loadkeys hu
```

Csatlakozás az internethez

Szükségünk lesz internet eléréshez hogy a csomagokat telepíteni tudjuk. WiFi esetén segítségünkre lesz az iNet wireless daemon (iwd) mely szolgáltat egy parancssoros kis programot  az **iwctl**-t, segítségével tudunk csatlakozni egy routerhez.  
Ha beütjük az iwctl parancsot majd egy entert ütünk akkor az iwd interaktív promptját kapjuk meg. 

```bash
$ iwctl
[iwd]#
```

Ahhoz hogy csatlakozni tudjunk szükségünk lesz az eszköz nevére (wireless kártya) illetve az ssid-ra, a hálozat nevére. A device list segítségével megkereshetjük az eszközünket.

```bash
[iwd]# device list
#
#                            Devices
# -------------------------------------------------------------
#   Name          Address          Powered    Adapter    Mode
# -------------------------------------------------------------
#   wlan0         ...              on         ...        ...
```

A példa alapján a wlan0 az eszközünk. Most már csatlakozhatunk is

```bash
[iwd]# station wlan0 connect wifid_ssid-ja
```

Lehetőségünk van interaktív prompt helyett egyből csatlakozni a router-hez az iwctl programocskával. 

```
# Wireless esetén
$ iwctl stations wlan0 connect <ssid>
```

Rendszeróra frissítése

```bash
$ timedatectl set-ntp true
```

### 1.1 Partícionálás

A kézikönyv kétféle telepítést mutat be, egy egyszerű telepítést grub bootloader segítségével, illetve egy titkosított partícióra való telepítést systemd-boot segítségével. Későbbiekben csak normál illetve titkosított-ként fogok rá hivatkozni.

#### 1.1.a Titkosított partícióval

A LUKS (Linux Unified Key Setup) egy lemez titkosítási szabvány, amelyet a Linux rendszer használ a lemezek és partíciók biztonságos titkosítására. A LUKS célja, hogy egységes és hordozható formátumot biztosítson a különböző disztribúciók és eszközök között, és a felhasználók számára egyszerűbbé tegye a titkosítás beállítását és használatát.

A LUKS közvetlenül támogatott a Linux kernelben, és felhasználói (user space)eszközökkel, mint például a cryptsetup, kezelhető.


```bash
# kilistázzuk a partíciókat
$ fdisk -l
# partícionálás cfdisk segítségével
$ cfdisk /dev/...
```

Hozz létre egy GPT partíciós táblát

A [GPT](/GPT.md) (GUID Partition Table) egy modern partíciós tábla formátum, amelyet az MBR (Master Boot Record) korlátainak kiküszöbölésére fejlesztettek ki. A GPT az UEFI (Unified Extensible Firmware Interface) specifikáció része, és számos előnyt kínál az MBR-rel szemben.


- EFI rendszerpartíció (ESP) (512-1024 MiB, `type: EFI System`)
- LUKS titkosított partíció (a maradék terület)

Formázd az ESP partíciót FAT32-re:

```bash
mkfs.fat -F32 /dev/sdX1
```

LUKS titkosítás és LVM

Titkosítsd a LUKS partíciót
```bash
cryptsetup luksFormat /dev/sdX2
```

Nyisd meg a LUKS konténert

```bash
cryptsetup open /dev/sdX2 cryptlvm
```

Hozz létre egy LVM fizikai kötetet

```bash
pvcreate /dev/mapper/cryptlvm
```

 Hozz létre egy LVM volume groupot
 
```bash
vgcreate vg0 /dev/mapper/cryptlvm
```

Hozz létre logikai köteteket a root és swap számára

Root (például 20G):

```bash
lvcreate -L 20G vg0 -n root
```

Swap (például 2G):

```bash
lvcreate -L 2G vg0 -n swap
```

A fennmaradó hely (opcionális home partíció):

```bash
lvcreate -l 100%FREE vg0 -n home
```

Fájlrendszerek létrehozása

Formázd a logikai köteteket

Root:

```bash
mkfs.ext4 /dev/vg0/root
```

Swap:

```bash
mkswap /dev/vg0/swap
```

Home (opcionális):

```bash
mkfs.ext4 /dev/vg0/home
```


Mountold a root fájlrendszert

```bash
mount /dev/vg0/root /mnt
```

Hozd létre a szükséges könyvtárakat és mountold azokat

EFI rendszerpartíció:

```bash
mkdir /mnt/boot
mount /dev/sdX1 /mnt/boot
```

 
Home (ha van):

```bash
mkdir /mnt/home
mount /dev/vg0/home /mnt/home
```

Aktiváld a swapot

```bash
swapon /dev/vg0/swap
```

#### 1.1.b Normál módon

```bash
# kilistázzuk a partíciókat
$ fdisk -l
# partícionálás cfdisk segítségével
$ cfdisk /dev/...
```

cfdisk

Select label type:  gpt/**dos**/sgi/sun

```
    /dev/sda1 - 1G - for /boot EFI
    /dev/sda2 - 5G - for root  Linux 
    /dev/sda3 - 1G - for swap  Linux swap / Solaris (érdemes a ram kétszeresét megadni, pl.: 16GB esetén 32GB-ot) 
```

formázzuk meg a létrehozott partiíciókat

```bash
# Az efi partíció specifikáció szerint kötelezően előírja a FAT partíciós táblát ezért azt FAT-ra formázzuk

$ mkfs.fat -F 32 /dev/sdxy

# A root partíció ext4
$ mkfs.ext4 /dev/sdxy

# A swap partíció pedig
$ mkswap /dev/sdxy

```

csatoljuk a partíciókat

```bash
# A root partíciót a /mnt helyre
$ mount /dev/root_partition /mnt
# Az efi részére hozzuk létre az /mnt/efi helyet és csatoljuk fel
$ mkdir /mnt/efi
$ mount /dev/sdxy /mnt/efi
# csatoljuk a swap partíciót is
$ swapon /dev/sdxy
```

### 1.2 Telepítsük az alap rendszert

```bash
$ pacman -Syy
$ pacstrap /mnt base base-devel linux linux-headers linux-firmware vim dhcpcd
```

### 1.3 Konfiguráljuk a rendszert

```bash
# Fstab: Generate an fstab file (use -U or -L to define by UUID or labels, respectively):
$ genfstab -U /mnt >> /mnt/etc/fstab
```

Váltsunk root-ot az új rendszerre:
```bash
$ arch-chroot /mnt
```

Időzóna beállítása

```bash
# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
$ ln -sf /usr/share/zoneinfo/Europe/Budapest /etc/localtime

$ hwclock --systohc
```

Lokalizáció beállítása

Szerkesszük meg a */etc/locale.gen' fájlt és vegyük ki a kommentezést a megfelelő bejegyzés mögül.
```bash
$ vim /etc/locale.gen
----------------------
...
# hu_HU.UTF-8 UTF-8
...
# Ezt követően generáljuk le
$ locale-gen
```
Hozzunk létre a locale.conf fájlt és adjuk meg a LANG értékét:
```
$ nano /etc/locale.conf
-----------------------
LANG=hu_HU.UTF-8
```

```bash
echo "LANG=hu_HU.UTF-8" > /etc/locale.conf
```


Hozzuk létre a konzolos billetnyűzet beállítását és adjuk meg a KEYMAP értékét:
```bash
$ nano /etc/vconsole.conf
-------------------------
KEYMAP=hu
```

```bash
echo "KEYMAP=hu" > /etc/vconsole.conf
```

Hálózat konfiguráció

Hozzuk létre a hostname fájlt és állítsuk be a hostnevet:
```bash
$ nano /etc/hostname
--------------------
myhostname
```

Állítsuk be a hosts fájlt

```bash
echo "127.0.0.1 localhost" >> /etc/hosts
echo "::1 localhost" >> /etc/hosts
```

Állítsuk be a DHCP szolgáltatását
```
$ systemctl enable dhcpcd
```

### 1.4 Felhasználók beállítása
A root jelszó megadása
```bash
$ passwd
```

Hozzunk létre új felhasználót sudo jogokkal
```bash
$ useradd -m -g users -G wheel -s /bin/bash username
$ passwd username

# szerkesszük meg a sudoers fájlt
$ nano /etc/sudoers
# a root ALL=(ALL) ALL alá adjuk hozzá az új felhasználónkat

username ALL=(ALL) ALL

# illetve ha a weel csoport tagjait engedjük akkor az alábbi bejegyzést
%wheel ALL=(ALL) ALL
```

### 1.5 Initramfs és bootloader beállítása

Az [initramfs](/INITRAMFS.md) (initial RAM filesystem) egy ideiglenes fájlrendszer, amelyet a Linux kernel használ a rendszer indítási folyamatának korai szakaszában. Az initramfs biztosítja a szükséges környezetet ahhoz, hogy a kernel betölthesse a rendszer fő fájlrendszerét és elindíthassa az init folyamatot. Az initramfs tartalmazza azokat az eszközmeghajtókat, modulokat és segédprogramokat, amelyek szükségesek az alapvető rendszerindítási feladatok elvégzéséhez.

#### 1.5.a Titkosított systemd-boot partíció esetén

**Állítsd be a megfelelő jogosultságokat:** Győződj meg róla, hogy csak a root felhasználó férhet hozzá írási joggal A /boot mappához, míg más felhasználók csak olvasási joggal rendelkeznek:

```
chmod 755 /boot
TODO: fmask=0077 dmask=0077 az fstab-ban a boot-ra

```

Telepítsük a szükséges csomagokat

```bash
 pacman -Syu efibootmgr cryptsetup lvm2
```

Initramfs beállítása
Szerkeszd a `/etc/mkinitcpio.conf` fájlt, és add hozzá a `keyboard`, `keymap`, `encrypt` és `lvm2` hookokat a `HOOKS` sorban:

```
HOOKS=(base udev autodetect modconf block keyboard keymap encrypt lvm2 filesystems)
```

Generáld újra az initramfs

```bash
mkinitcpio -p linux
```

Telepítsd a systemd-boot bootloadert

```
bootctl install
```

Konfiguráld a bootloadert:
Hozz létre egy `loader.conf` fájlt az `/boot/loader` könyvtárban és add hozzá a következő sorokat:
```sh
default arch
timeout 5
editor 0
```

Hozd létre az Arch Linux boot bejegyzést /boot/loader/entries/arch.conf:

```
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options cryptdevice=/dev/sdX2:cryptlvm root=/dev/vg0/root rw
```


#### 1.5.b Normal GRUB loader telepítés esetén

```bash
 pacman -Sy grub efibootmgr dosfstools os-prober mtools
```


```bash
# létrehozzuk az EFI boot direktory-t
$ mkdir /boot/EFI
# felcsatoljuk a boot partíciót
$ mount /dev/sdxy /boot/EFI
# telepítjük a grubot EFI módon
$ grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
# telepítjük a grubhoz a nyelvi fájlokat
$ cp /usr/share/locale/hu/LC_MESSAGES/grub.mo /boot/grub/locale/hu.mo
# kiírjuk a GRUB configot
$ grub-mkconfig -o /boot/grub/grub.cfg
```

### 1.6 Gép újraindítása

Kilépés a chrootból és újraindítás
```bash
exit
umount -R /mnt
swapoff -a
reboot
```

## 2. Felhasználói GUI telepítése

## 2.a GNOME telepítése
```bash
# Next, Install X Window System (xorg) using command:
$ sudo pacman -S xorg xorg-server
# install gnome
$ sudo pacman -S gnome gnome-tweaks gnome-nettool gdm archlinux-wallpaper networkmanager firefox xdg-desktop-portal-gnome xdg-desktop-portal gcr4

$ sudo systemctl enable gdm.service
$ systemctl enable NetworkManager
$ systemctl enable --user gcr-ssh-agent.socket

# indítsuk újra a gépet

# Ha problémák adódnának a hanggal
$ sudo pacman -S pulseaudio pulseaudio-alsa
```

A telepítést követően néhány fontos beállítás, utó telepítés.
Régebbi GTK téma telepítése a mostmár legacy programokhoz. Illetve Flatpak programokhoz. Töltsd le az alábbi repót és telepítsd a témát:
https://github.com/lassekongo83/adw-gtk3

Ha flatpak alkalmazásokat használsz célszerű lehet telepíteni az alábbi témákat flatpakon keresztül is.
```bash
$ flatpak install org.gtk.Gtk3theme.adw-gtk3 org.gtk.Gtk3theme.adw-gtk3-dark
```

Engedélyezni tudod az adw-gtk3 témát a `gnome-tweaks`-ben, vagy manuál terminálból:
```bash
# Change the theme to adw-gtk3 light
$ gsettings set org.gnome.desktop.interface gtk-theme 'adw-gtk3' && gsettings set org.gnome.desktop.interface color-scheme 'default'0
# Change the theme to adw-gtk3-dark
$ gsettings set org.gnome.desktop.interface gtk-theme 'adw-gtk3-dark' && gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'
# Revert to GNOME's default theme
$ gsettings set org.gnome.desktop.interface gtk-theme 'Adwaita' && gsettings set org.gnome.desktop.interface color-scheme 'default'
```

GDM billentyűzet kiosztás beállítása
```bash
# GDM language
$ localectl set-x11-keymap hu
```
 

### 2.a.2.1 SSH askpass

dependencies:

```bash
sudo pacman -Sy gcr4
```

zsh export:
```bash
export SSH_AUTH_SOCK=$XDG_RUNTIME_DIR/gcr/ssh
```

enable gcr service

```bash
systemctl enable --now --user gcr-ssh-agent.socket
```



## 2.b KDE telepítése
```bash
$ pacman -S xorg plasma plasma-wayland-session kde-applications 

$ systemctl enable sddm.service
$ systemctl enable NetworkManager.service
```


## 3 POST install

### 3.1 AUR (yay) csomagkezelő

A Yay csak az Arch User Repository-ban érhető el. Vegyük figyelembe, hogy manuálisan is telepíthetünk csomagokat az AUR-ból anélkül, hogy AUR segédprogramot használnánk (hasonlóan ahhoz, ahogy a Yay-t fogod telepíteni), de ahogy a neve is sugallja, egy "AUR segédprogram" segít a telepítési folyamatban, megkönnyítve a csomagok telepítését minimális felhasználói beavatkozással.

```bash
sudo pacman -S --needed base-devel git
```

```bash
git clone https://aur.archlinux.org/yay.git
```

```bash
cd yay
```

```bash
makepkg -si
```


## 4 Pacman
Az arch alapértelmezett csomagkezelője a Pacman

### 4.1 Néhány hasznos pacman parancs

Azoknak a csomagoknak a kilistázása amelyek nem a main repóból jöttek tehát AUR vagy pkgbuild csomagok.

```sh
sudo pacman -Qm
```

#### 4.1.1 Zst csomagok telepítése

```bash
sudo pacman -U --noconfirm your-package.pkg.tar.zst
```

#### 4.1.2 Csomagok eltávolítása

``` bash
sudo pacman -Rcns <package>
```


Egyéb törléshez tartozó kapcsolók:

| -R              | csomag eltávolítása (törlés) flag                                                                      |
| --------------- | ------------------------------------------------------------------------------------------------------ |
| -c, --cascade   | csomagok és az azokat igénylő csomagok eltávolítása                                                    |
| -n, --nosave    | törölje a konfigurációs fájlokat                                                                       |
| -s, --recursive | a feleslegessé váló függőségeket is távolítsa el (-ss a kézzel telepített függőségeket is eltávolítja) |
| -u, --unneeded  | más csomag által nem igényelt csomagok eltávolítása                                                    |




## 5 ISSUES

### 5.1 Python issues

#### Error: To install Python packages system-wide, try 'pacman -S

You can use Python's `venv` like described [here](https://stackoverflow.com/a/75696359/10626495).

However if you really want to install packages that way, then there are a couple of solutions:

- use `pip`'s argument `--break-system-packages`,
- add following lines to `~/.config/pip/pip.conf`:

```ini
[global]
break-system-packages = true
```
### 5.2 Postman

Postman telepítése flatpak-et követően nem indul el. A hiba az hogy hiányolja (vélhetően) nem települ cert amit a Postman használni szeretne.
Megoldás generáljunk egy cert-et neki

```bash
mkdir -p ~/.var/app/com.getpostman.Postman/config/Postman/proxy
cd ~/.var/app/com.getpostman.Postman/config/Postman/proxy
openssl req -subj '/C=US/CN=Postman Proxy' -new -newkey rsa:2048 -sha256 -days 365 -nodes -x509 -keyout postman-proxy-ca.key -out postman-proxy-ca.crt
```

### 5.3 Visual studio

```bash
yay visual-studio-code-bin
```

Gnome extensions places open vscode issue
```bash
xdg-mime default org.gnome.Nautilus.desktop inode/directory
```

### 5.4 Flameshot

Flameshot telepítése flatpak segítségével
```bash
flatpak install org.flameshot.Flameshot
flatpak permission-set screenshot screenshot org.flameshot.Flameshot yes
```

### 5.5 Rendszer monitor

```bash
flatpak install io.missioncenter.MissionCenter 
```


## 6 Egyéb csomagok telepítése

```bash
sudo pacman -Sy nerd-fonts
```



 




