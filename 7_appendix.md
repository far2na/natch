# Приложение 1. Настройка окружения для использования лицензированного Natch

Для использования лицензированной версии инструмента *Natch* необходимо иметь HASP ключ или сетевую лицензию.
При наличии ключа следует только выполнить пункт 1 из этого приложения. В случае с сетевой лицензией необходимо выполнить
все нижеописанные действия.

В качестве хостовой системы предлагается использовать Ubuntu 20.04.

Все необходимые файлы для установки и настройки будут предоставлены пользователю, а именно
пакет aksusbd_8.31-1_amd64.deb, sailor-license.ovpn и логин/пароль для подключения.

1. Установить deb пакет aksusbd_8.31-1_amd64.deb с помощью команды:

```bash
    sudo dpkg -i aksusbd_8.31-1_amd64.deb
```

2. Открыть браузер и ввести строку ``localhost:1947``

3. Откроется главная страница ``Sentinel Admin Control Center``, на которой нужно перейти в раздел ``Configuration``, в нем найти раздел ``Access to Remote License Managers``.

4. Убедиться, что в полях ``Allow Access to Remote Licenses`` и ``Broadcast Search to Remote Licenses`` стоят галочки.

5. В поле ``Remote License Search Parameters`` ввести ``license.intra.ispras.ru`` и нажать кнопку ``Submit``.

6. Создать VPN соединение.

Для этого создания VPN соединения необходимо открыть настройки операционной системы и перейти в раздел *Network*. В секции *VPN* нажать на плюсик.

<img src="https://raw.githubusercontent.com/ispras/natch/main/images/app_vpn/settings.png" width=60% height=60% alt="Настройки сети">

Появится окно для добавления новой конфигурации VPN. Нужный вариант *Import from file..*, куда следует передать входящий в поставку
файл *sailor-license.ovpn*. В предлагаемой ОС OpenVPN установлен по умолчанию.

<img src="https://raw.githubusercontent.com/ispras/natch/main/images/app_vpn/import.png" width=40% height=40% alt="Импорт настроек VPN">

После импорта файла появится окно настройки соединения, в которое необходимо вписать логин и пароль, так же входящие в поставку.
Далее осталось нажать кнопку *Add* в верхнем правом углу и конфигурация будет создана.


<img src="https://raw.githubusercontent.com/ispras/natch/main/images/app_vpn/login.png" width=40% height=40% alt="Добавление конфигурации VPN">

Осталось только включить переключатель напротив VPN и соединение будет установлено.

<img src="https://raw.githubusercontent.com/ispras/natch/main/images/app_vpn/ok.png" width=40% height=40% alt="VPN готов">

Кроме того, управлять подключением можно в трее, как показано на рисунке ниже.

<img src="https://raw.githubusercontent.com/ispras/natch/main/images/app_vpn/profit.png" width=60% height=60% alt="Подключение VPN">

После всех проделанных действий инструмент готов к использованию на вашем компьютере.


# Приложение 2. Командная строка эмулятора Qemu

Пример командной строки для запуска Qemu выглядит следующим образом:

``./qemu-system-x86_64 -hda debian.qcow2 -m 6G -monitor stdio -netdev user,id=net0 -device e1000,netdev=net0``

- ``qemu-system-x86_64``: исполняемый файл эмулятора
- ``-hda debian.qcow2``: подключение образа гостевой операционной системы
- ``-m 6G``: выделение оперативной памяти гостевой системе
- ``-monitor stdio``: подключение управляющей консоли эмулятора к терминалу
- ``-netdev user,id=net0 -device e1000,netdev=net0``: настройка сети и подключение сетевой карты модели е1000

Командная строка выше просто запускает эмулятор с заданным образом диска. Для работы Natch потребуются дополнительные опции командной строки, а именно:

```
-os-version Linux
-plugin <plugin_name>
```

Опция ``os-version`` настраивает *Natch* для работы с операционной системой Linux, а ``plugin`` непосредственно загружает плагин.


# Приложение 3. Формат списка исполняемых модулей (module_config.cfg)

Пример конфигурационного файла:

```ini
    # module_config.cfg

    [Image1]
    path=vmlinux
    map=System.map
    textstart=0xffffffff81000000

    [Image2]
    path=exe/dpkg
    debuginfo=exe/dpkg.dbg

    [Image3]
    path=exe/apt-get
```

Конфигурационный файл содержит набор секций с префиксом *Image*.

В каждой такой секции описывается отдельный бинарный файл.
В этой секции могут быть объявлены четыре поля: *path*, *map*, *textstart* и *debuginfo*.
Обязательным является только *path* (путь к бинарному файлу в хостовой системе).
В поле *map* можно указать путь к файлу с символьной информацией, сгенерированной
компилятором или дизассемблером IDA. А в *debuginfo* - путь к ELF-файлу с отладочными символами.

При использовании IDA для генерации map файлов нужно выставлять галочку *Segmentation information*.

**Поле textstart**

В разделах типа *Image* вместе с *map* может быть задано поле *textstart*.
Как правило, нет необходимости определять *textstart* вручную, потому что это может
делать скрипт *module_config.py*. Если же утилита вывела сообщение об ошибке,
необходимо проанализировать бинарный файл самостоятельно.

*textstart* используется, если адреса в символьном файле абсолютные, а в исполняемом файле нет.
Так как модуль может загружаться в разные места памяти, необходимо вычислить смещение каждой функции от начала секции .text.
В поле *textstart* как раз и указывается адрес начала секции *.text*.
Это поле нужно в редких случаях, например, для ядерных модулей (см. ниже). В остальных случаях можно ничего не указывать.

Чтобы получить информацию о секциях в исполняемом файле, можно использовать утилиту *readelf*:

```bash
    readelf -S <config_name>
```

Пример, когда *textstart* необходимо указывать:

```text
    map-файл:

    ffffffffa0000bed t cleanup_module
    ffffffffa00008c2 t logring_syslog_write_raw
    ffffffffa0000a4b t init_module
    ffffffffa000098d t logring_syslog_write

    вывод утилиты readelf:

    Section Headers:
    [Nr] Name              Type             Address           Offset
         Size              EntSize          Flags  Link  Info  Align
    [ 0]                   NULL             0000000000000000  00000000
         0000000000000000  0000000000000000           0     0     0
    [ 1] .text             PROGBITS         0000000000000000  00000040
         0000000000000c8c  0000000000000000  AX       0     0     4
```

Нас интересуют секции типа *PROGBITS*. Если у них указан нулевой адрес, то их не получится
сопоставить с map-файлом при его загрузке в *Natch*.
Поэтому необходимо вручную определить адрес секции *.text*.

Пример, когда *textstart* не нужен:

```text
    map-файл:

    00000006:000000000000C000       .init_proc
    00000007:000000000000C030       .wget_netrc_db_free
    00000007:000000000000C040       .wget_bar_update
    00000007:000000000000C050       .seteuid
    00000007:000000000000C060       .chdir
    00000007:000000000000C070       .fileno
    00000007:000000000000C080       .wget_list_free
    00000007:000000000000C090       .dup2
    00000007:000000000000C0A0       .printf

    вывод утилиты readelf:

    Section Headers:
    [Nr] Name              Type             Address            Offset
         Size              EntSize          Flags  Link  Info  Align
    [11] .init             PROGBITS         000000000000c000   0000c000
         0000000000000017  0000000000000000 AX     0     0     4
```

Адрес секции здесь указан и он соответствует адресам, записанным в map-файле,
поэтому параметр *textstart* можно не указывать.


# Приложение 4. Формат конфигурационного файла для секции Tasks (task_struct_offsets.ini)

Конфигурационный файл содержит в себе смещения полей структур ядра Linux, необходимых для работы инструмента.

Пример конфигурационного файла:

```ini
    [Version]
    Version=3

    [Task struct offsets]
    pid=1224
    name=1648
    parent=1240
    state=16
    task_struct=89152

    [Files struct offsets]
    ts_files=1720
    fs_file=48
    fs_fdt=32
    fdt_file=8
    f_dentry=24
    d_parent=24
    d_name=40
    d_iname=56

    [Memory mapping struct offsets]
    ts_mm=1048
    ts_mm_active=1056
    mm_mmap=0
    mm_map_count=104
    mm_exe_file=928
    vm_start=0
    vm_end=8
    vm_next=16
    vm_prev=24
    vm_mm=64
    vm_file=160
    vma_struct_size=208
```

**Секция Version**

- *Version*. Версия формата конфигурационного файла.

**Секция Task struct offsets**

- *task_struct*. Смещение переменной *current_task* в сегменте *GS*. Для старых ядер должно быть 0, *current_task* извлекается из стека.
- *pid*. Смещение поля *pid* внутри *task_struct*.
- *name*. Смещение поля *comm* внутри *task_struct*.
- *parent*. Смещение поля *real_parent* внутри *task_struct*.
- *state*. Смещение поля *__state* внутри *task_struct*.

**Секция Files struct offsets**

- *ts_files*. Смещение поля *files* внутри *task_struct*.
- *fs_file*. Смещение поля *fd* или *fdtab.fd* внутри *files_struct*.
- *fs_fdt*. Смещение поля *fdt* внутри *files_struct* или 0 для старых ядер.
- *fdt_file*. Смещение поля *fd* внутри *fdtable* или 0 для старых ядер.
- *f_dentry*. Смещение поля *f_path.dentry* внутри *file*.
- *d_parent*. Смещение *d_parent* внутри *dentry*.
- *d_name*. Смещение поля *name* или *d_name.name* внутри *dentry*.
- *d_iname*. Смещение *d_iname* внутри *dentry*.

**Секция Memory mapping struct offsets**

- *ts_mm*. Смещение поля *mm* внутри *task_struct*.
- *ts_mm_active*. Смещение поля *active_mm* внутри *task_struct*.
- *mm_mmap*. Смещение поля *mmap* внутри *mm_struct*.
- *mm_map_count*. Смещение поля *map_count* внутри *mm_struct*.
- *mm_exe_file*. Смещение поля *exe_file* внутри *mm_struct*.
- *vm_start*. Смещение поля *vm_start* внутри *vm_area_struct*.
- *vm_end*. Смещение поля *vm_end* внутри *vm_area_struct*.
- *vm_next*. Смещение поля *vm_next* внутри *vm_area_struct*.
- *vm_prev*. Смещение поля *vm_prev* внутри *vm_area_struct*.
- *vm_mm*. Смещение поля *vm_mm* внутри *vm_area_struct*.
- *vm_flags*. Смещение поля *vm_flags* внутри *vm_area_struct*.
- *vm_file*. Смещение поля *vm_file* внутри *vm_area_struct*.
- *vma_struct_size*. Размер структуры *vm_area_struct*.

## Принцип работы автоматической настройки

Смещения полей в структурах данных ядра определяются в ходе отдельного настроечного запуска *Natch*.
В первую очередь ищется смещение переменной *current_task* в сегменте *GS*.
Поиск опирается на эвристики, связанные с перехватом системного вызова *getpid*. Затем определяются смещения полей
*pid*, *name*, *parent* и всех переменных из раздела *files struct offsets*. Для их вычисления обрабатываются системные вызовы
*getpid* и *open*. Для поиска смещения поля *state* перехватывается
системный вызов *exit*. Смещения переменных из раздела *memory mapping struct offsets*
вычисляются на основе системного вызова *mmap*.


# Приложение 5. Команды монитора Qemu для работы с Natch

Некоторыми плагинами, входящими в состав *Natch*, можно управлять с помощью дополнительных команд. В этом разделе
перечислены доступные команды.

-   **enable_tainting** - включить анализ помеченных данных.

-   **info replay** - посмотреть текущий шаг записи или воспроизведения. Шаги соответствуют числу выполненных процессорных команд.

-   **natch_get_attack_surface <filename> [named]** - получить поверхность атаки, параметр *filename* обязательный, параметр *named* опциональный, логического типа, при включении отображает только те сущности, которые имеют имена.

-   **show_tasks <filename>** - посмотреть полное дерево задач Linux. Дерево будет отображено на экране и продублировано в указанный файл.

-   **show_modules_list <filename>** - посмотреть полный список загруженных модулей. Список будет отображен на экране и продублирован в указанный файл.

-   **taint_file <name>** - пометить файл (если секция *TaintFile* в конфигурации не активна, необходимо загрузить плагин *taint_file* командой ``load_plugin taint_file``).


# Приложение 6. История релизов Natch
    
**Natch v.2.0**

*   Представлен графический интерфейс SNatch v.1.0. Основные возможности:

        * Построение графа взаимодействия процессов

                * интерактивный просмотр с помощью привязки к timeline
                * доступно четыре режима отображения сущностей
        * Построение стека вызовов

*   Улучшено распознавание модулей
*   Добавлена поддержка сжатых секций с отладочной информацией
*   Добавлена возможность фильтрации сетевых пакетов по протоколу
*   Доработан скрипт для конфигурирования Natch

        * рабочая директория для проектов
        * добавлена возможность проброса портов в гостевую систему
*   Запущен внешний баг-трекер

**Natch v.1.3.2**

*   Улучшение и рефакторинг механизма распознавания модулей
*   Поддержка набора инструкций SSE4.2 при отслеживании помеченных данных
*   Настройка Natch теперь осуществляется с помощью одного скрипта
*   Выходные файлы инструмента собираются в одну директорию
*   Добавлен журнал событий Natch
*   Небольшие изменения в настройке и конфигурационном файле Natch

**Natch v.1.3.1**

*   Исправлена ошибка сбора покрытия для Ida 7.0
*   Исправлена ошибка сохранения лога для помеченных параметров функций
*   Исправлена опечатка в генерируемом конфигурационном файле
*   Название снапшота вынесено в начало скрипта запуска Natch в режиме воспроизведения

**Natch v.1.3**

*   Добавлена поддержка отладочной информации
*   Добавлена поддержка map файлов, сгенерированных компилятором gcc
*   Расширен набор опций конфигурационного файла Natch:

        * добавлена возможность указывать список файлов для пометки
        * добавлена возможность загрузки дополнительных плагинов
*   Добавлен скрипт для генерации конфигурационного файла для модулей
*   Обновлен скрипт для генерации командных строк запуска Natch
*   Обновлено ядро эмулятора Qemu до версии 6.2


**Natch v.1.2.1**

-   Исправлена ошибка работы утилиты qemu-img в VirtualBox под Windows 10
-   Исправлена ошибка с генерацией имени оверлея в скрипте для генерации командных строк
-   Добавлена возможность задавать поля скрипта перед его запуском

**Natch v.1.2**

-   Скрипт для генерации командных строк
-   Выгрузка данных о покрытии кода в IDA Pro
-   Построение графа модулей, передающих друг другу помеченные данные
-   Ранжирование функций поверхности атаки по числу обращений к помеченным данным
-   Исправлены дефекты в механизме распространения пометок
-   Мелкие изменения в конфигурационном файле инструмента

**Natch v.1.1**

-   Возможность настраивать Natch с помощью конфигурационного файла
-   Логирование входящих сетевых пакетов
-   Отображение операций записи помеченных данных
-   Построение графа, описывающего взаимодействие процессов
-   Поддержка ELF32
-   Исправление списка процессов: уничтожение завершившихся

**Natch v.1.0**

-   Пометка сетевого трафика (сетевая карта e1000)
-   Пометка файлов
-   Возможность задания порогового значения для пометок
-   Определение модулей с исполняемым кодом в памяти виртуальной машины
-   Возможность подгружать map файлы из IDA
-   Получение списка процессов, модулей и функций, участвующих в обработке помеченных данных
-   Получение подробной трассы по каждому обращению к помеченным данным, включающей стек вызовов функций, адрес обращения к помеченным данным и количество помеченных байтов
