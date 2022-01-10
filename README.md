#                                                               Загрузка системы BIOS/UEFI

##                                          LINUX НА UEFI

  [Введение в UEFI и GPT](https://help.ubuntu.ru/wiki/%D1%80%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE_%D0%BF%D0%BE_ubuntu_desktop_14_04/%D0%BE%D1%81%D0%BE%D0%B1%D0%B5%D0%BD%D0%BD%D0%BE%D1%81%D1%82%D0%B8_%D1%83%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B8_%D0%BD%D0%B0_%D0%BF%D0%BB%D0%B0%D1%82%D1%8B_%D1%81_uefi)

Перед тем, как начинает загружаться операционная система, специализированное программное обеспечение компьютера инициализирует все компоненты, проверяет их готовность к работе и только потом передает управление загрузчику ОС.
Раньше для этих целей использовался набор программ BIOS, но этот стандарт сейчас считается устаревшим, а ему на замену пришла технология UEFI, которая поддерживает защищенную загрузку и много других преимуществ.
Большинство образов Linux уже содержат загрузчик EFI и прошивка компьютера его найдет при следующей перезагрузке.
У UEFI в этом плане есть несколько отличий от BIOS. Во первых, это использование таблицы разделов диска GPT. Это новая технология, которая имеет массу преимуществ над MBR, среди которых очень большое количество первичных разделов (в MBR только четыре), восстановление при повреждениях, и многое другое.  Второе отличие в том, что файлы загрузчика операционной системы больше не хранятся в первых 512 байтах жесткого диска. Все они хранятся на отдельном разделе, который называется ESP.

#                        ЧЕМ ОТЛИЧАЕТСЯ MBR ОТ GPT

По сути жесткий диск представляет собой огромное адресное пространство в которое можно   записывать данные. Чтобы знать сколько разделов существует, какого они размера, на какой ячейке начинаются и на какой заканчиваются нужно где-то хранить эти данные. Вот тут уже нужна таблица разделов MBR или GPT. Или как они расшифровываются Master Boot Record и GUID Partition Table. Не смотря на то, что они отличаются архитектурой, они выполняют одну и ту же работу.

####                     MBR

-  MBR - это старый стандарт таблицы разделов

        MBR находится в самом начале диска, использует 32-битные идентификаторы для разделов (точнее будет сказать, что указатели на начало и конец раздела имеют размерность 32 бита), которые размещаются в очень маленьком кусочке пространства (64 байта) в самом начале диска (в конце первого сектора диска).

        Из-за такого маленького объёма поддерживаются только четыре первичных раздела. Поскольку используется 32-битная адресация, то каждый раздел может быть не более 2,2 ТБ.

        В MBR находится исполняемый код, который может сканировать разделы в поисках операционной системы, а также инициировать загрузку операционной системы в Linux там находится код инициализации Grub. Поскольку места там очень мало, обычно этот код используется только для инициализации основного загрузчика расположенного где-нибудь на диске.

        Загрузочная запись не имеет никакой запасной MBR, так что если приложение перезапишет основную загрузочную запись, то вся информация о разделах будет потеряна.

####                       GPT

-  GPT это современный стандарт управления разделами на жестком диске. Это часть стандарта EFI (Extensible Firmware Interface),

        Использует уже 64-битные идентификаторы для разделов, поэтому кусочек   пространства, в котором сохраняется информация о разделах, уже больше чем 512 байт, кроме того, не существует ограничения на количество разделов.

        Самое первое отличие - это использование совсем другой адресации диска. В MBR использовалась адресация зависимая от геометрии диска. Адрес состоял с трех значений головка, цилиндр и сектор (например 0,0,0). В GPT используется адресация LBA. Это блочная адресация, каждый блок имеет свой номер, например LBA1, LBA2, LBA3, и так далее, при чем адреса MBR автоматически транслируются в LBA, например LBA1 будет иметь адрес 0,0,1 и так далее.

        GPT не содержит кода загрузчика, она рассчитывает что этим будет заниматься EFI, здесь размещена только таблица разделов. В блоке LBA0 находится MBR, это сделано для защиты от затирания GPT старыми утилитами работы с дисками, а уже с блока (LBA1) начинается сама GPT. Под таблицу разделов резервируется 16 384 байт памяти, по 512 на блок, а это 32 блока, таким образом первые разделы начнутся с блока LBA34 (32+1MBR+1GPT).

        GPT поддерживает юникод поэтому вы можете задавать имена и атрибуты разделам. Имена могут быть заданы на любом поддерживаемом языке и вы сможете обращаться к дискам по этим именам. Для дисков используются глобальные уникальные идентификаторы GUID (Globally Unique IDentifier), это одна из вариаций UUID с большей вероятностью уникальных значений, может также использоваться для идентификации дисков вместо имен.

Для загрузчика UEFI — на диске должен быть создан небольшой раздел (100–250 МБ), который называется "Extensible Firmware Interface System Partition" (системный раздел расширяемого интерфейса прошивки, ESP). Кроме указанного размера, раздел должен быть отформатирован в файловой системе FAT32 (требование стандарта — UEFI может загружаться только с носителя, отформатированного в файловой системе FAT32) и быть загрузочным. На нем находятся драйверы аппаратных компонентов, к которым может получать доступ запущенная операционная система. И в этом случае загрузка происходит прямо с этого раздела, что намного быстрее.
___

## Чтобы по полной задействовать функционал UEFI, диск должен быть с GPT, и на нём должен быть специальный раздел ESP
___

 #### Как узнать gpt или mbr используется.
 В Linux мы можем использовать для этого утилиту fdisk или parted. Просто выполните:

        sudo fdisk -l (parted -l)
        ...............
        ...............
        Disklabel type: dos
        Disk identifier: 0x1c50df99

-  Disklabel type:

        dos - значит, что  используется mbr,
        gpt - значит, что используется gpt.

___

## Без плясок с бубном невозможно 32-битную систему заставить работать с UEFI
___

#                        1.  Домашнее задание

1. Попасть в систему без пароля несколькими способами
2. Установить систему с LVM, после чего переименовать VG
3. Добавить модуль в initrd
4. * LVM без отдельного раздела boot


##                 1.1  Попасть в систему без пароля несколькими способами

> **Начальный образ загрузки системы _Initial RAM disk (initrd)_. Содержит модули ядра необходимые для работы устройств при дальнейшей инициализации системы и скрипты их загрузки.**


##### 1.1.1  Первый вариант изменения пароля пользователя:

Для получения доступа необходимо открыть GUI VirtualBox, запустить виртуальную машину и при выборе ядра для загрузки нажать "e" - в данном контексте edit. Попадаем в окно где можем изменить параметры загрузки:   

-  ищем строку:

            linux16 /boot/vmlinuz-5.10.11.old root=UUID=1c419d6c-5064-4a2b-953c-05b2c67edb15 ro no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rhgb quiet  

-  удаляем в ней, если есть (rhgb quiet) , и  (console=tty0 console=ttyS0,115200n8), и добавляем значение rd.break:

            linux16 /boot/vmlinuz-5.10.11.old root=UUID=1c419d6c-5064-4a2b-953c-05b2c67edb15 ro no_timer_check  net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto   rd.break

    для продолжения загрузки, нажимаем ctrl-x, и попадаем в аварийный режим ("emergency mode"). В этом режиме корневая файловая система смонтирована (в режиме Read-Only)  в /sysroot. Для выполнения задания, нужно перемонтировать корневую систему в режим rw (чтения-записи), сделать её основной, относительно загруженной точки (мы находимся не в корне нашей ФС), и уже там выполнить команду по смене пароля root или любого другого пользователя.

            mount -o remount,rw /sysroot  <------ перемонтировали в rw режим
            chroot /sysroot                          <------- сделали каталог /sysroot корневым
            passwd root                                <--------- смена пароля пользователя root, просмотреть по быстрому всех пользователей можно комбинацией (~ +tab tab  -  тильда и два раза tab)
            touch /.autorelabel                       < -------- обязательно сделать, чтоб подсистема защиты SELINUX перемаркировала файлы, иначе зайти не получится.

-  Далее exit, выход из chroot, и exit, выход из аварийного режима режима, сисетма сама перегрузится попутно перемаркировав файлы системы (selinux)

##### 1.1.2 Второй вариант изменения пароля пользователя:            

Для получения доступа необходимо открыть GUI VirtualBox, запустить виртуальную машину и при выборе ядра для загрузки нажать "e" - в данном контексте edit. Попадаем в окно где можем изменить параметры загрузки:

-  ищем строку:

            linux16 /boot/vmlinuz-5.10.11.old root=UUID=1c419d6c-5064-4a2b-953c-05b2c67edb15 ro no_timer_check **console=tty0 console=ttyS0,115200n8** net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto **rhgb quiet**

удаляем в ней, если есть,  эти значения:

  - **rhgb quiet** <--   чтоб видеть процесс загрузки  и  отключить графический экран  
  - **console=tty0 console=ttyS0,115200n8**

добавляем значение
  - **init=/bin/sh**

результирующая строка:

  - **_linux16 /boot/vmlinuz-5.10.11.old root=UUID=1c419d6c-5064-4a2b-953c-05b2c67edb15 ro no_timer_check net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto init=/bin/sh_**

для продолжения загрузки, нажимаем ctrl-x.  При таком передаваемом значении  ядру -  **init=/bin/sh**, корневая файловая система смонтирована (в режиме Read-Only) в "/".

Для выполнения задания, нужно перемонтировать корневую систему в режим rw (чтения-записи)

  - mount -o remount,rw /
  - passwd root
  - touch /.autorelabel <------ **обязательно сделать, чтоб подсистема защиты SELINUX перемаркировала файлы, иначе зайти не получится.**
  - mount -o remount,ro /  

Далее exit, выход из chroot, и exit, выход из аварийного режима режима,

  > На выходе получили "Kernel panic", но при перезагрузке, доступ с новым паролем получен.


#                                           1.2  * LVM без отдельного раздела boot


Это задание бдет выполнятся на базе VM (виртуальной машины) созданной на первом занятии, с созданием своего ядра из исходников. Используем VM "ashum1976/centos7_k5_raid_home" из вагрант облака.
        параметры машины:
        диск  40 Gb, на xfs, один " / " раздел.
        добавляем диск на 10Gb
        Разметка нового диска будет GPT, загрузка из BIOS (пока без UEFI)

Перенесём это всё на другой диск GPT, создадим  LVM с загрузкой непосредственно с тома LVM без отдельного boot раздела.


Метки при использовании parted обозначают тип таблицы разделов, которую вы хотите использовать. Если вы выбрали «gpt», убедитесь, что вы загрузились с машины UEFI. **Ваша система не загрузится, если вы ошиблись!** Чтобы проверить что у вас, введите:

        ls sys/firmware


Если он содержит строку efi, все готово:

        acpi dmi **efi** memmap qemu_fw_cfg



Первоначально сделаем диск с разметкой GPT:

            [root@raid-create ~]# parted -s /dev/sdb mklabel gpt

            [root@raid-create ~]# fdisk -l
            .......
            .......
            WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

            Disk /dev/sdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
            Units = sectors of 1 * 512 = 512 bytes
            Sector size (logical/physical): 512 bytes / 512 bytes
            I/O size (minimum/optimal): 512 bytes / 512 bytes
            Disk label type: gpt
            Disk identifier: 006AF6E9-90E6-4CF0-9E4C-506AC4647128

Для того чтобы установить любой дистрибутив на GPT с BIOS, необходимо создать на диске таблицу разделов GPT, а также небольшой раздел (2 мегабайта) без файловой системы и с типом bios_grub (тип для parted, для gdisk код типа EF02), в котором будет находится загрузочный код:

            [root@raid-create ~]# parted -s /dev/sdb mkpart  bootgrub 0 2Mib    <-------- просто создаём раздел, без файловой системы, размером 2Mb


            [root@raid-create ~]# parted -s /dev/sdb set 1 bios_grub on  <------- помечаем его как bios_grub

Создадим раздел, для нашей системы под LVM:

            [root@raid-create ~]# parted -s /dev/sdb mkpart  lvmroot 2Mib 100%

            [root@raid-create ~]# fdisk -l
            .......
            .......
               Device Boot      Start         End      Blocks   Id  System

                        WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

            Disk /dev/sdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
            Units = sectors of 1 * 512 = 512 bytes
            Sector size (logical/physical): 512 bytes / 512 bytes
            I/O size (minimum/optimal): 512 bytes / 512 bytes
            Disk label type: gpt
            Disk identifier: AFA6DBA0-32A8-497A-9827-0934F5F779AA


            #         Start          End    Size  Type            Name
            1           34         4095      2M  BIOS boot       lvmgpt
            2         2048     20969471     10G  Linux LVM       lvm


Далее создаём диск под LVM, группу, том, файловую систему :

            [root@raid-create ~]# pvcreate  /dev/sdb2

            Physical volume "/dev/sdb1" successfully created

            [root@raid-create ~]# vgcreate hwlvm /dev/sdb2

            [root@raid-create ~]# lvcreate -n root hwlvm -l+100%FREE

            [root@raid-create ~]# mkfs.xfs /dev/mapper/hwlvm-root

            [root@raid-create ~]# mkfs.xfs /dev/mapper/hwlvm-root

 Монтируем созданную ФС в каталог /mnt/, биндим псевдофайловые системы (proc, dev,sys,run) делаем его корневым  :          

            [root@raid-create ~]# mount /dev/mapper/hwlvm-root /mnt

            [root@raid-create ~]# for i in /proc/ /sys/ /dev/ /run/ ; do mount --bind $i /mnt/$i; done

            [root@raid-create ~]# chroot /mnt

Поставим grub2 c предложенного репозитория:

            [root@raid-create /]# yum-config-manager --add-repo=https://yum.rumyantsev.com/centos/7/x86_64/

            [root@raid-create /]# yum install grub2 -y --nogpgcheck

            [root@raid-create ~]# blkid

            /dev/sdb1: PARTLABEL="lvmgpt" PARTUUID="0043207c-dbde-44f8-8901-e893db3e5c85"
            /dev/sdb2: UUID="HNkVZs-4NwR-tgeC-Yn0Y-jIWH-nbiI-qjJGY7" TYPE="LVM2_member" PARTLABEL="lvm" PARTUUID="49f6a518-e61b-4ae0-9c0d-8447537506a1"
            /dev/mapper/hwlvm-root: UUID="1bd467eb-2731-48bf-aa7f-158f769e84f6" TYPE="xfs

            [root@raid-create /]# vi /etc/fstab   <----- записываем UUID нашего коневого раздела (1bd467eb-2731-48bf-aa7f-158f769e84f6) , вместо существующего

Добавляем строчки (rd.lvm.lv=hwlvm/root), для подключения нашего корневого раздела на LVM:


            [root@raid-create /]# vi /etc/default/grub  

Конфигурируем загрузчик:

            [root@raid-create /]# grub2-mkconfig -o /boot/grub2/grub.cfg

Пересобираем initramfs, для включения LVM модулей

            [root@raid-create /]# dracut -f -v /boot/initramfs-5.10.11.img

Устанавливем загрузчик:

            [root@raid-create /]# grub2-install /dev/sdb

            Installing for i386-pc platform.
            Installation finished. No error reported.
Выключаем машину, отсоединяем диск ( в VirtualBox удаляем диск), грузимся с новой системы.

## В результате проделанной работы, перенесли рабочую систему с MBR на GPT диск, и превели с обычного раздела на LVM раздел. Вторй раздел присутствует, но только как необходимость для возможности загрузки GPT диска с BIOS системы.


#                                           1.3  Установить систему с LVM, после чего переименовать VG (Volume Group)

Смотрим что у нас есть по LVM:


            [root@gpt-lvm ~]# vgs               <------- смотрим имя нашей VG (Volume Group) группы в LVM

            VG    #PV #LV #SN Attr   VSize   VFree
            hwlvm   1   1   0 wz--n- <10.00g    0           <----- текущее имя группы "hwlvm"

Переименовываем:

            [root@gpt-lvm ~]# vgrename hwlvm vgnew

            Volume group "hwlvm" successfully renamed to "vgnew"


Далее правим /etc/fstab (если запись о корневом разделе делали не по UUID), /etc/default/grub, /boot/grub2/grub.cfg. Везде заменяем старое название на новое.  Пересоздаем initrd image, чтобы он знал новое название Volume Group.
На стенде, fstab прописан по UUID, править не нужно, а вот /etc/default/grub нужно подправить:

            root@gpt-lvm ~]# cat /etc/default/grub
            GRUB_TIMEOUT=1
            GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
            GRUB_DEFAULT=saved
            GRUB_DISABLE_SUBMENU=true
            GRUB_TERMINAL_OUTPUT="console"
            GRUB_CMDLINE_LINUX="no_timer_check console=tty0 console=ttyS0,115200n8 rd.lvm.lv=hwlvm/root net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto"
            GRUB_DISABLE_RECOVERY="true"

*          меняем  rd.lvm.lv=hwlvm/root на rd.lvm.lv=vgnew/root

-          тоже делаем и с файлом /boot/grub2/grub.cfg

генерируем новые initramfs файлы:

            [root@gpt-lvm boot]#  for i in `ls initramfs-*img`; do dracut -v -f $i `echo $i | sed "s/initramfs-//g;s/.img//g"`; done   <------ делаем новые initramfs для всех ядер, которые у нас есть


Перегружаемся и проверяем:

            [root@gpt-lvm ~]# vgs

            VG    #PV #LV #SN Attr   VSize   VFree
            vgnew   1   1   0 wz--n- <10.00g    0

#                                           1.4 Добавить модуль в initrd

Скрипты модулей хранятся в каталоге /usr/lib/dracut/modules.d/. Для того чтобы добавить свой модуль создаем там папку с именем 05test.
В нее поместим два скрипта:

1.  module-setup.sh - который устанавливает модуль и вызывает скрипт test.sh
2.  test.sh - собственно сам вызываемый скрипт, в нём у нас рисуется пингвинчик

Сгенерируем по новой initramfs файл для ядра:

            [root@gpt-lvm modules.d]# dracut -f -v /boot/initramfs-$(uname -r).img
            Executing: /sbin/dracut -f -v
            dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
            dracut module 'plymouth' will not be installed, because command 'plymouthd' could not be found!
            dracut module 'plymouth' will not be installed, because command 'plymouth' could not be found!
            dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
            dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
            dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
            dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
            dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
            dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
            dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
            dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
            dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
            dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
            dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
            *** Including module: bash ***
            *** Including module: nss-softokn ***
            *** Including module: test **                                            <---- наш модуль

или команда:

            [root@gpt-lvm modules.d]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

            Смотрим, что модуль появился в initramfs:

            [root@gpt-lvm ~]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
            test



После перезагрузкм в логах увидим пингвина, или при загрузке смотреть как загружается система, и увидеть пингвина :)



- initial RAM disk (initrd)

#                       Ускорение загрузки Linux

[Ссылка на статью по настройке запуска](https://losst.ru/uskorenie-zagruzki-linux)

 **Начнём с настройки загрузчика grub**

Значения внесённые в конфиг файл загрузчика grub (/etc/default/grub)

rootfstype=ext4    <- тип файловой системы корневого раздела
raid=noautodetect  <- рэйда нет, детектить нет необходимости (lvm есть)
lpj=3500099        <- параметр выцепить из dmesg файла, и прописать в конфиге загрузчика. Позволяет задать константу loops_per_jiffy, что позволит ядру не вычислять ее каждый раз и сэкономит
plymouth.enable=0  <- заставка plymouth
quiet              <- вывод, это долго, поэтому говорим ядру что на экран нужно выводить минимум информации

> отключил монтирования swap раздела, он нужен для запуска сервисов hibernate-resume, но пока система при загрузке спотыкается на отсуттствие swap раздела и висит 90сек. пока не  отваливает по тайм-ауту, из-за отстутствия swap раздела.

после всех изменений нужно сгенерировать новый конфиг файл grub2:

      grub2-mkconfig -o /boot/grub2/grub.cfg


**Отключение ненужных служб и сервисов**

На домашнем компе отключил систему гибернации и сна, нормально не работает, система через раз выходит из данного состояния, причём в syspend режиме ещё быстро, в режиме гибернации - быстрее два раза загрузиться просто без гибернации и ещё раз перезагрузиться:
    - systemctl mask suspend.target
    - systemctl mask sleep.target
    - systemctl mask hibernate.target
    - systemctl mask hybrid-sleep.target
    - systemctl mask systemd-hibernate-resume@.service
    - systemctl mask systemd-suspend.service
    - systemctl mask systemd-hybrid-sleep.service
    - systemd-suspend-then-hibernate.service



plymouth-read-write.service - ????

systemctl disable --now sssd.service
systemctl disable vmware-USBArbitrator.service
systemctl disable vmware.service
systemctl disable rasdaemon.service --now      <- Проверка на наличие аппаратных ошибок
systemctl disable --now gpm.service

systemctl disable vmtoolsd.service  --now
