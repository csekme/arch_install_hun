A GPT (GUID Partition Table) egy modern partíciós tábla formátum, amelyet az MBR (Master Boot Record) korlátainak kiküszöbölésére fejlesztettek ki. A GPT az UEFI (Unified Extensible Firmware Interface) specifikáció része, és számos előnyt kínál az MBR-rel szemben.
A GPT Partíciós Tábla Előnyei és Jellemzői:

Több Partíció:
    A GPT akár 128 partíciót is támogat (alapértelmezés szerint), míg az MBR csak 4 elsődleges partíciót vagy 3 elsődleges és 1 kiterjesztett partíciót támogat.

Nagyobb Lemezméret Támogatás:
    A GPT több mint 2 terabájtos lemezeket is támogat, míg az MBR legfeljebb 2 terabájtos lemezekkel kompatibilis.

Redundancia:
    A GPT a lemez elején és végén is tárolja a partíciós táblát, ami növeli a hibával szembeni ellenállóképességet. Ez lehetővé teszi a partíciós tábla helyreállítását, ha az egyik példány sérül.

CRC32 Ellenőrzés:
    A GPT partíciós tábla és a partíciós adatok CRC32 ellenőrzőösszeggel vannak védve, ami biztosítja a tárolt adatok épségét és segít az esetleges hibák felismerésében.

Partíció Azonosító (GUID):
    Minden GPT partíció egyedi azonosítóval (GUID - Globally Unique Identifier) rendelkezik, ami megkönnyíti az egyes partíciók azonosítását és kezelését.

A GPT Partíciós Tábla Szerkezete:

Protective MBR:
    A GPT lemez első szektora egy "védő MBR"-t tartalmaz, amely az MBR-kompatibilis rendszerek számára jelzi, hogy a lemez egyetlen, nem partícionált területként van megjelölve, így megakadályozva, hogy ezek a rendszerek véletlenül felülírják a GPT lemezt.

GPT Header:
    A lemez második szektora tartalmazza a GPT fejlécet, amely információkat tartalmaz a GPT partíciós tábláról, például a GPT verzióját, a lemez GUID-ját, a partíciós táblák helyét és méretét, valamint az ellenőrzőösszegeket.

Partition Entries:
    A GPT fejléc után következnek a partíciós bejegyzések, amelyek mindegyike egy partíció adatait tartalmazza, beleértve a partíció GUID-ját, a partíció típusát, az induló és záró LBA (Logical Block Address) értékeket, valamint az esetleges partíció-specifikus attribútumokat.

GPT Használata Linux Rendszeren:

A GPT partíciós táblát különféle eszközökkel hozhatod létre és kezelheted Linux rendszereken, például gdisk, parted, cfdisk vagy fdisk (az újabb verziók támogatják a GPT-t).