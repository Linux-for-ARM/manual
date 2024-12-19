# Сборка образа для Allwinner

Данный раздел содержит инструкции по сборке загрузочного образа для плат, оснащённых Allwinner SoC.

## Создание базовых файлов

Здесь вам нужно создать ряд образов (образ с загрузчиком и образ с собранной системой), которые потом будут «объединены» в один большой образ, пригодный для записи на загрузочный носитель и дальнейшей эксплуатации.

### Создание образа с загрузчиком U-Boot

```admonish warning title="Внимание"
Обратите внимание на то, что в данном разделе предполагается, что вы запишете собранный образ на SD-карту. Тем не менее, некоторые платы с Allwinner SoC оснащены встроенной eMMC- или SPI NOR памятью, куда также можно установить загрузчик. Тем не менее, эти два варианта в руководстве не рассматриваются. Если вам нужно установить загрузчик на eMMC или SPI NOR Flash, воспользуйтесь сведениями из [**документации U-Boot**](https://docs.u-boot.org/en/latest/board/allwinner/sunxi.html).
```

Сначала создайте заголовок размером 2 Мб. Этот заголовок будет содержать скомпилированный ранее загрузчик.

```bash
dd if=/dev/zero bs=1M count=2 of=bootloader.img
```

Поскольку все системы на кристалле Allwinner пытаются найти загрузочный код в секторе №256 (128 Кб) SD-карты, подключенной к первому MMC-контроллеру, вам нужно записать собранный файл загрузчика по этому смещению в образ `bootloader.img`:

```bash
dd if=$LFA/bootloader.bin \
  conv=notrunc seek=128 bs=1k \
  of=bootloader.img
```

```admonish warning title="Внимание"
Вне зависимости от SoC (Allwinner, Broadcom или Rockchip), для которого вы собирали U-Boot, он был сохранён в файл `$LFA/bootloader.bin`. Так что имя файла в аргументе `dd if=...` правильное.
```

> Если вы собирали систему для *старых* Allwinner SoC (выпущенных до 2013 года), то эти SoC ищут загрузчик в секторе 16 (8 Кб). Для использования загрузчика U-Boot на старых SoC просто замените `seek=128` на `seek=8`.

### Создание образа с базовой ОС

Теперь нужно создать образ, в котором будут содержаться файлы собранной системы:

```bash
dd if=/dev/zero bs=1M count=512 of=rootfs.img
```

Здесь размер образа составляет 512 МБ. В зависимости от требований и объёма собранной системы установите вместо 512 своё значение.

Создайте файловую систему на этом разделе:

```bash
mkfs.ext4 -L BOOT \
  -O ^metadata_csum -F \
  -b 4096 \
  -E stride=2,stripe-width=1024 \
  -L rootfs rootfs.img
```

{{#include rootfs.md}}

---

> **Смотрите также:**
>
> - [**How to prepare a SD card?**](https://docs.armbian.com/User-Guide_Getting-Started/#how-to-prepare-a-sd-card) (<https://docs.armbian.com/>).
> - [**Allwinner SoC based boards**](https://docs.u-boot.org/en/latest/board/allwinner/sunxi.html) (сборка загрузчика для плат на базе Allwinner SoC).