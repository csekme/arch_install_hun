Az initramfs (initial RAM filesystem) egy ideiglenes fájlrendszer, amelyet a Linux kernel használ a rendszer indítási folyamatának korai szakaszában. Az initramfs biztosítja a szükséges környezetet ahhoz, hogy a kernel betölthesse a rendszer fő fájlrendszerét és elindíthassa az init folyamatot. Az initramfs tartalmazza azokat az eszközmeghajtókat, modulokat és segédprogramokat, amelyek szükségesek az alapvető rendszerindítási feladatok elvégzéséhez.
Az initramfs főbb jellemzői és funkciói:

Kis Méret:
    Az initramfs kis méretű és a memóriában fut, ami gyors indítást tesz lehetővé.

Dinamikus Tartalom:
    Az initramfs tartalma dinamikusan generálható a rendszer konfigurációja alapján. A tartalom olyan modulokat és eszközmeghajtókat tartalmaz, amelyek szükségesek a rendszerindításkor.

Modularitás:
    Az initramfs különféle modulokat tartalmaz, amelyek lehetővé teszik a kernel számára az eszközök felismerését és a fájlrendszer csatolását. Például tartalmazhat RAID, LVM, fájlrendszer és hálózati modulokat.

Rendszerindítási Folyamat Támogatása:
    Az initramfs biztosítja a szükséges eszközöket és modulokat a root fájlrendszer felcsatolásához és az init folyamat elindításához. Ez különösen fontos, ha a root fájlrendszer speciális beállításokat igényel, mint például titkosítás vagy LVM.

Az initramfs Működése:

Bootloader:
    Az indítási folyamat során a bootloader (pl. GRUB, systemd-boot) betölti a Linux kernelt és az initramfs képfájlt a memóriába.

Kernel Indítása:
    A kernel elindul és csatlakoztatja az initramfs-t mint egy ideiglenes root fájlrendszert.

initramfs Futtatása:
    Az initramfs elindít egy init szkriptet vagy folyamatot (általában init vagy init.sh), amely elvégzi a szükséges inicializációs lépéseket. Ezek közé tartozik az eszközmeghajtók betöltése, a fájlrendszerek ellenőrzése, és a root fájlrendszer felcsatolása.

Root Fájlrendszer Csatolása:
    Az initramfs felcsatolja a végleges root fájlrendszert, általában a pivot_root vagy switch_root parancsok segítségével.

Átadás a Fő Rendszernek:
    Miután a root fájlrendszer csatlakoztatva van, az initramfs átadja az irányítást a fő rendszer init folyamatának (pl. systemd, SysVinit).

initramfs Generálása:

Az initramfs képfájlt általában a disztribúcióhoz tartozó eszközök generálják, mint például mkinitcpio (Arch Linux), dracut (Fedora, RHEL), vagy update-initramfs (Debian, Ubuntu).
Példa az mkinitcpio használatára Arch Linuxon:

Az alapértelmezett konfiguráció ellenőrzése:

```sh
    cat /etc/mkinitcpio.conf
```

initramfs generálása:

```sh
    sudo mkinitcpio -p linux
```

Ez a parancs generál egy új initramfs képfájlt az alapértelmezett kernel (linux) számára. A konfigurációs fájl (/etc/mkinitcpio.conf) meghatározza, hogy milyen modulokat és beállításokat kell belefoglalni az initramfs-be.

