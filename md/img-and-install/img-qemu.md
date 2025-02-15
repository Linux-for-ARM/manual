# Сборка образа для QEMU

Для того, чтобы запустить собранную систему в эмуляторе QEMU, нам потребуется два файла: файл с кодом загрузчика U-Boot и файл с образом системы. Это отличается от запуска системы на реальном оборудовании, поскольку для реального компьютера обычно загрузчик и собранная ОС содержатся в одном файле.

## Создание образа

{{#include create_img.md}}

## Копирование файлов

Теперь вам необходимо смонтировать созданный выше образ и скопировать в него файлы собранной системы (саму систему, ядро, Devicetree и сценарий загрузки):

```bash
mkdir -pv /tmp/lfa_rootfs
sudo mount -v rootfs.img /tmp/lfa_rootfs

sudo cp -rfv $LFA_SYS/* /tmp/lfa_rootfs
sync
```

> Все действия здесь выполняются от имени пользователя `lfa`. Поскольку здесь используется программа `sudo`, вам нужно добавить пользователя `lfa` в группу `wheel`.

После копирования файлов размонтируйте образ:

```bash
sudo umount /tmp/lfa_rootfs
```
