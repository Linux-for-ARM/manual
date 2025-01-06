# Сборка образа (для остальных)

```admonish warning title="Внимание"
Это заготовка страницы. Здесь приведены общие инструкции, тестирование которых не производилось. Сведения здесь приведены только для формирования у читателя общего представления о процессе сборки загрузочного образа LFA в том случае, если он собирал LFA для тех SoC, которые не были описаны в этом руководстве. Для получения информации о структуре загрузочного носителя (информация о разделах, их форматах и смещениях, а также иные данные), смотрите [**документацию U-Boot**](https://docs.u-boot.org/en/latest/board/index.html) для каждого поддерживаемого SoC.

Инструкции отсюда готовились для второй версии LFA. В более новых версиях руководства работоспособность и актуальность комманд отсюда не гарантируется.
```

## Создание базовых файлов

Здесь вам нужно создать ряд образов (образ с загрузчиком и образ с собранной системой), которые потом будут «объединены» в один большой образ, пригодный для записи на загрузочный носитель и дальнейшей эксплуатации.

### Создание образа с загрузчиком U-Boot

```admonish warning title="Важно"
Далее вам предлагается создать файл `bootloader.img`. Создавайте его только в том случае, если собирали систему для Allwinner, Broadcom или Rockchip SoC. Если вы собирали загрузчик для запуска в эмуляторе U-Boot, вам нужно пропустить действия из этого подпункта «Создание образа с загрузчиком U-Boot».
```

Сначала создадим заголовок размером 2 Мб:

```bash
dd if=/dev/zero bs=1M count=2 of=bootloader.img
```

И копируем сохранённый образ U-Boot по смещению 128:

```bash
dd if=$LFA/bootloader.bin \
  conv=notrunc seek=128 \
  of=bootloader.img
```

```admonish warning title="Внимание"
Вне зависимости от SoC (Allwinner, Broadcom или Rockchip), для которого вы собирали U-Boot, он был сохранён в файл `$LFA/bootloader.bin`. Так что имя файла в аргументе `dd if=...` правильное.
```

### Создание образа с базовой ОС

Теперь нужно создать образ, в котором будут содержаться файлы собранной системы:

```bash
dd if=/dev/zero bs=1M count=512 of=rootfs.img
```

Здесь размер образа составляет 512 Мб. В зависимости от требований и объёма собранной системы установите вместо `512` своё значение.

Создайте файловую систему на этом разделе:

```bash
mkfs.ext4 -L BOOT \
  -O ^metadata_csum -F \
  -b 4096 \
  -E stride=2,stripe-width=1024 \
  -L rootfs rootfs.img
```

## Копирование файлов

Смонтируйте полученный образ и скопируйте в неё файлы нашей системы (саму систему, ядро, Devicetree и сценарий загрузки):

```bash
mkdir -pv /tmp/lfa_rootfs
sudo mount -v rootfs.img /tmp/lfa_rootfs

sudo cp -rfv $LFA_SYS/* /tmp/lfa_rootfs
sync
```

```admonish warning title="Внимание"
Все действия здесь выполняются от имени пользователя `lfa`. Поскольку здесь используется программа `sudo`, вам нужно добавить пользователя `lfa` в группу `wheel`.
```

После копирования файлов размонтируйте образ:

```bash
sudo umount /tmp/lfa_rootfs
```

## Создание окончательного образа

Объедините два образа в один:

```bash
dd if=rootfs.img conv=notrunc oflag=append bs=1M seek=2 of=bootloader.img
```

Если вы не генерировали образ `bootloader.img` (т.е. собираете систему для эмуляции в QEMU), то этот пункт вам нужно пропустить. Вместо него переименуйте образ `rootfs.img` в `bootloader.img`:

```bash
mv -v rootfs.img bootloader.img
```

## Создание таблицы разделов

Созданный нами образ нерабочий, поскольку ещё не содержит таблицу разделов. Создайте её с помощью `fdisk`:

```fdisk
fdisk bootloader.img
o
n
p
1
4096
+512M

w
```

> **Значения новых команд:**
>
> После начала редактирования мы вводим команду `o`, чтобы создать пустую таблицу разделов MBR. Затем командой `n` создаём новый раздел. Выбираем тип, номер и первый сектор. Дело в том, что размер сектора равен 512 байт, т.е. 1 Кб равен двум секторам. Размер заголовка 2 Мб, т.е. 2048 Кб или 4096 секоторов. Размер раздела можно указать в Мб. Сохранение и выход командой `w`.

Переименуйте файл `bootloader.img`, дав ему более логичное и подходящее имя:

```bash
mv -v bootloader.img lfa-{{lfa_ver}}.img
```

## Сжатие образа

При необходимости вы можете сжать образ с помощью `xz` или любого другого архиватора, который вам больше нравится. Я часто видел `img`-образы, сжатые с помощью `xz`, поэтому использую его:

```bash
xz lfa-{{lfa_ver}}.img
```

## Запись образа на SD-карту

Теперь вы можете записать полученный образ на SD-карту. Для этого можете использовать `dd`:

```bash
sudo dd if=lfa-{{lfa_ver}}.img of=/dev/sdX
```

где `X` - буква (`a`, `b`, `c`, etc.) устройства, на которое будет производиться запись образа, например, `/dev/sdc`. Обратите внимание на то, что в некоторых случаях вместо `sdX` имя устройства может быть и `mmcblkX`.

Также для записи можете использовать программу Balena Etcher.

---

> **Смотрите также:**
>
> - [**How to prepare a SD card?**](https://docs.armbian.com/User-Guide_Getting-Started/#how-to-prepare-a-sd-card) (<https://docs.armbian.com/>).
