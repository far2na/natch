<div style="page-break-before:always;">
</div>

# <a name="app_1"></a>Приложение 1. Настройка окружения для использования лицензированного Natch

Для использования лицензированной версии инструмента *Natch* необходимо иметь HASP ключ или сетевую лицензию.

При наличии ключа следует выполнить только пункт 1 из этого приложения. В случае с сетевой лицензией необходимо выполнить
все нижеописанные действия.

Все необходимые файлы настройки будут предоставлены пользователю, а именно, пакет aksusbd_*current_version*\_amd64.deb (Ubuntu, Debian)/haspd-_*current_version*\_x86_64.rpm (Alt), файл с конфигурацией vpn ``*.ovpn`` и логин/пароль для подключения.

1. Установить окружение для ключей Sentinel

**Для Ubuntu, Debian**

Установить deb пакет `aksusbd` с помощью команды:
```bash
    sudo dpkg -i aksusbd_*current_version*_amd64.deb
```

**Для Alt**

Установить rpm пакет `haspd` и перезапустить службу:
```bash
    sudo rpm -i haspd-_*current_version*_x86_64.rpm
    sudo systemctl restart haspd
```

2. Создать VPN соединение (в качестве примера хостовой системы использована Ubuntu 20.04).

Для этого создания VPN соединения необходимо открыть настройки операционной системы и перейти в раздел *Network*. В секции *VPN* нажать на плюсик.

<img src="https://raw.githubusercontent.com/ispras/natch/main/images/app_vpn/settings.png"><figcaption>_Настройки сети_</figcaption>

Появится окно для добавления новой конфигурации VPN. Нужный вариант *Import from file..*, куда следует передать входящий в поставку
файл *\*.ovpn*. В предлагаемой ОС OpenVPN установлен по умолчанию.

<img src="https://raw.githubusercontent.com/ispras/natch/main/images/app_vpn/import.png"><figcaption>_Импорт настроек VPN_</figcaption>

После импорта файла появится окно настройки соединения, в которое необходимо вписать логин и пароль, так же входящие в поставку. В этом же окне следует перейти на вкладки IPv4 и IPv6 и установить флажок ``Use this connection only for resources on its network``. Далее осталось нажать кнопку *Add* в верхнем правом углу и конфигурация будет создана.


<img src="https://raw.githubusercontent.com/ispras/natch/main/images/app_vpn/login.png"><figcaption>_Добавление конфигурации VPN_</figcaption>

Осталось только включить переключатель напротив VPN и соединение будет установлено.

<img src="https://raw.githubusercontent.com/ispras/natch/main/images/app_vpn/ok.png"><figcaption>_VPN готов_</figcaption>

Кроме того, управлять подключением можно в трее, как показано на рисунке ниже.

<img src="https://raw.githubusercontent.com/ispras/natch/main/images/app_vpn/profit.png"><figcaption>_Подключение VPN_</figcaption>

3. Открыть браузер и ввести строку ``localhost:1947``

4. Откроется главная страница ``Sentinel Admin Control Center``, на которой нужно перейти в раздел ``Configuration``, в нем найти раздел ``Access to Remote License Managers``.

5. Убедиться, что в полях ``Allow Access to Remote Licenses`` и ``Broadcast Search to Remote Licenses`` стоят галочки.

6. В поле ``Remote License Search Parameters`` ввести ``license.intra.ispras.ru`` и нажать кнопку ``Submit``.

После всех проделанных действий инструмент готов к использованию на вашем компьютере.


# <a name="app_2"></a>Приложение 2. Командная строка эмулятора Qemu

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


# <a name="module_config"></a>Приложение 3. Формат списка исполняемых модулей (module.cfg)

Пример конфигурационного файла:

```ini
    # module.cfg

    [Image1]
    path=vmlinux
    map=System.map
    textstart=0xffffffff81000000

    [Image2]
    path=exe/dpkg
    debuginfo=exe/dpkg.dbg

    [Image3]
    path=exe/apt-get

    [Image4]
    path=exe/cat
    debuginfo=exe/cat-8.32-31.fc35.x86_64.debug
    tieddebug=exe/dwp
    vmidb=vmidb/cat
```

Конфигурационный файл содержит набор секций с префиксом *Image*.

В каждой такой секции описывается отдельный бинарный файл.
В этой секции могут быть объявлены следующие поля: *path*, *map*, *textstart*, *debuginfo*, *tieddebug*, *vmidb*.
Обязательным является только *path* (путь к бинарному файлу в хостовой системе).
В поле *map* можно указать путь к файлу с символьной информацией, сгенерированной
компилятором или дизассемблером IDA (при использовании IDA для генерации map файлов нужно выставлять галочку *Segmentation information*).
А в *debuginfo* - путь к ELF-файлу с отладочными символами.


Дополнительно поддерживается разделённая на части отладочная информация. Путь к вспомогательному
файлу указывается в поле *tieddebug*.

Отладочная и символьная информация из исполняемых файлов может быть заранее разобрана и сохранена
во время работы конфигурационных скриптов. Путь к полученному хранилищу записывается в поле *vmidb*.

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

# <a name="natch_mon_commands"></a>Приложение 4. Команды монитора Qemu для работы с Natch

Некоторыми плагинами, входящими в состав *Natch*, можно управлять с помощью дополнительных команд. В этом разделе
перечислены доступные команды.

-   **enable_tainting** - включить анализ помеченных данных.

-   **info replay** - посмотреть текущий шаг записи или воспроизведения. Шаги соответствуют числу выполненных процессорных команд.

-   **show_tasks <filename>** - посмотреть полное дерево задач Linux. Дерево будет отображено на экране и продублировано в указанный файл.

-   **show_modules_list <filename>** - посмотреть полный список загруженных модулей. Список будет отображен на экране и продублирован в указанный файл.

-   **taint_file <name>** - пометить файл (если секция *TaintFile* в конфигурации не активна, необходимо загрузить плагин *taint_file* командой ``load_plugin taint_file``).


# <a name="app_5"></a>Приложение 5. Формат файла с покрытием кода


Выходной файл начинается со списка загруженных модулей, из которого собирается информация о покрытии. Каждый элемент этого списка содержит следующую информацию о модуле:

- Идентификатор
- Адрес его начала и конца
- Адрес начала секции .text (если данная информация была указана в конфигурационном файле)
- Идентификатор и имя процесса, в котором модуль был выполнен
- Путь, по которому модуль расположен на диске

После списка загруженных модулей, находится таблица, содержащая ББ, которые были выполнены при сборе информации о покрытии.

Каждый ББ представляет собой двоичную структуру, размером 8 байт, со следующими полями:

- *4 байта* : Смещение от начала модуля, к которому принадлежит ББ
- *2 байта* : Размер ББ
- *2 байта* : Идентификатор модуля, в котором находится ББ

Каждый ББ встречается в логе ровно один раз, независимо от того, сколько раз он был выполнен.

# <a name="app_requirements"></a>Приложение 6. Требования и ограничения Natch

## Требования к хостовой системе

* Объем ОЗУ на хосте должен быть минимум втрое больше, чем ОЗУ гостевой системы (виртуальной машины).
* Для записи сценария предпочтительным является процессор, обеспечивающий наибольшую производительность на одно ядро. Это связано с тем, что сценарий записывается в режиме полносистемной эмуляции вычислительных ресурсов гостевой системы, при этом задействуется одно вычислительное ядро хостовой системы. Анализировать записанный сценарий можно на многоядерном сервере.
* Хостовая ОС должна быть семейства Linux, официальные сборки делаются для Ubuntu 20, Ubuntu 22, Debian 11, Alt 10.

## Требования к гостевой системе (виртуальной машине)

* Процессор x86_64. 32-битный x86 процессор не поддерживается.
* Эмулируемая аппаратная платформа должна поддерживаться в qemu.
* Поддерживается эмуляция только сетевой карты e1000. Разделение нескольких сетевых карт в виртуальной машине не поддерживается
* Гостевая ОС должна быть семейства Linux, в частности поддерживается работа в Ubuntu 20-22, Fedora 23-36, Astra Linux 1.6, Debian.
* ELF-файлы для анализа могут быть 32-битными или 64-битными. Ядро должно быть 64-битным.
* Гостевая система должна нормально работать с включенными виртуальными часами (параметр qemu -icount 0 или -icount auto).

## Подготовка подлежащей анализу гостевой системы

* Бинарные файлы анализируемых программ необходимо извлечь из гостевой системы для того, чтобы они могли быть найдены в памяти гостевой системы на этапе анализа. То же касается нестандартных сборок Python-интерпретатора.
* Собирать бинарные файлы анализируемых программ нужно с отладочной информацией (ключ -g) любым современным компилятором. Если при этом отключать оптимизации (ключ -O0), то отображаемый в интерфейсе анализа стек вызовов будет более точным/понятным.
* Не рекомендуется использовать в гостевой системе ОС с графическим интерфейсом, так как его загрузка занимает много времени.
* Этап подготовки гостевой системы (установка ОС, сборка бинарных файлов анализируемых программ и т.п.), предшествующий этапу записи сценария, рекомендуется выполнять в режиме аппаратной виртуализации (ключ --qemu-kvm) с целью ускорения процесса подготовки.
* Анализ кластеров из нескольких хостовых систем возможен по отдельности -- для каждой хостовой системы запускается отдельный экземпляр qemu. Кластер можно связать через виртуальный коммутатор. Генерация совокупной интегральной поверхности атаки и ее единого графического представления не поддерживается.

## Дополнительные возможности Natch (по запросу)

* Поддержка 32-битного ядра гостевой системы возможна, но не тестировалась.
* Пометка данных, поступающих через другие виды сетевых карт, отличных от e1000.

# <a name="app_releases"></a>Приложение 7. История релизов Natch

[**Natch v.2.4**](https://nextcloud.ispras.ru/index.php/s/zxMra4pdpBRkCgd)

* SNatch
    * Интерфейс SNatch теперь и на русском языке
    * Добавлен поиск в графах (стек вызовов, флейм граф)
    * Добавлена информация об операциях с файлами
    * Добавлена навигация по истории перемещений в боковой панели
    * Некоторые изменения интерфейса
    * Исправлен ряд багов
* Natch
    * Поддержка отладочной информации для языка Go
    * Добавлена возможность запуска Natch в облегченном режиме
    * Добавлена сборка Natch для Astra Linux 1.7.3
    * Добавлена поддержка автотюнинга для ОС семейства Windows
    * Изменения скриптов:
        * добавлен скрипт для обновления конфигурационного файла для модулей
        * выходные файлы анализа теперь в собственном каталоге, архив не перезаписывается
    * Исправлен ряд багов

[**Natch v.2.3.1**](https://nextcloud.ispras.ru/index.php/s/NALSzi9xGSaftsN)

* SNatch
    * Улучшен PDF отчет
* Natch
    * Расширен перехват системных вызовов
    * Улучшена поддержка DWARF

[**Natch v.2.3**](https://nextcloud.ispras.ru/index.php/s/natch_v.2.3)

* SNatch
    * Добавлен граф вызовов Python функций
        * реализован переход из основного графа вызовов в Python и обратно
    * Добавлена возможность получения отчетов об анализе:
        * полный отчет в формате PDF
        * сохранение графов в виде цветного или ч/б изображения
    * Добавлен раздел Файлы, отображающий участвующие в сценарии файлы с возможностью фильтрации
    * Добавлена боковая панель свойств для большинства аналитик
        * для процессов-интерпретаторов предусмотрено отображение списка выполнявшихся скриптов
    * Сохранение положения узлов графов во время и после сессии
    * Добавлена опция фильтрации во вкладках Ресурсы и Дерево процессов
    * Во вкладке Трафик добавлена фильтрация: только помеченные пакеты; пакеты, участвующие в сценарии; все пакеты
    * Во вкладку О проекте добавлена статистика помеченных данных
    * Изменения интерфейса:
        * пересмотрено главное меню
        * новая цветовая схема
        * скрывающаяся легенда для графов
        * возможность выбора цветовой схемы для флейм графа
    * Исправлен ряд багов
* Natch
    * Добавлена возможность пометки локальных соединений
    * Добавлена поддержка Python:
        * перехват вызовов Python функций
        * автоматическое скачивание интерпретатора CPython/libpython из образа
    * Добавлена поддержка маленьких исполняемых файлов в качестве объекта оценки
    * Добавлена возможность сбора корпуса данных для фаззинга выбранных функций (для аргументов простых типов)
    * Добавлена возможность фильтрации трафика по порту для протокола UDP
    * Сборки Natch теперь в виде пакетов (ubuntu20-22, debian11, alt10)
    * Улучшена работа с извлечением символов из DWARF
    * Ликвидированы файлы с текстовой поверхностью атаки (output_text)
    * Изменения скриптов:
        * добавлена удобная возможность создавать подкаталоги для записываемых сценариев
        * скачивание системных модулей доступно, даже если пользовательские не загружены
        * доработан скрипт change_settings.py
    * Исправлен ряд багов

[**Natch v.2.2**](https://nextcloud.ispras.ru/index.php/s/natch_v.2.2)

* SNatch
    * Добавлен раздел Ресурсы, включающий информацию о модулях, файлах и сокетах 
    * Добавлен раздел Трафик:
        * списки интерфейсов и сессий
        * возможность просмотра трафика в Wireshark 
    * Улучшение юзабилити. Новые возможности:
        * параллельное создание нескольких проектов
        * кнопка отмены для построения графов
        * переименовывание проектов
        * закрытие всех вкладок одной кнопкой
        * отсутствие дублирования вкладок
        * логирование работы SNatch в файл
    * Добавлена метаинформация о сценарии
    * Улучшено дерево процессов
    * Добавлен прототип интеграции со Svace
    * Добавлена генерация аннотаций для Futag
    * Исправлен ряд багов
* Natch
    * Разбор отладочной информации вынесен на этап конфигурирования
    * Добавлено скачивание отладочных символов для модуля ядра
    * Улучшено распознавание процессов и модулей
    * Визуальное отображение хода тюнинга
    * Пометка директорий, а так же файлов по заданному префиксу
    * Архиватор артефактов изменен на более быстрый
    * Добавлена статистика помеченных сущностей в результате выполнения сценария
    * Изменен скрипт natch_run.sh:
        * поддержан текстовый режим работы эмулятора
        * добавлена возможность скачивать отладочные символы
        * поддержка относительных путей для образов
    * Исправлен ряд багов


[**Natch v.2.1.1**](https://nextcloud.ispras.ru/index.php/s/natch_v.2.1.1)

* Исправлен баг с построением графа процессов
* Улучшения в распространении помеченных данных
* Доработан раздел Modules в SNatch


[**Natch v.2.1**](https://nextcloud.ispras.ru/index.php/s/natch_v.2.1)

[(reserve) Natch v.2.1_ova](https://nextcloud.ispras.ru/index.php/s/natch_v.2.1_vbox)

* Обновлен графический интерфейс SNatch
    * Добавлены новые аналитики:
        * временной граф процессов
        * флейм граф процессов
        * граф помеченных модулей
        * дерево процессов
    * Улучшен граф вызовов
    * Добавлены горячие клавиши для управления визуальными элементами
    * Изменена схема БД, больше не требуется копирование модулей
* Доступно автоматическое получение отладочной информации для системных библиотек для гостевых ОС Ubuntu, Debian и Fedora
    * Добавлен скрипт для извлечения файлов из гостевой ОС
* Добавлена частичная поддержка FreeBSD
* Доработан механизм определения смещений ядерных структур
* Добавлено определение строк запуска процессов
* Исправлен ряд багов
* Скрипты для конфигурирования Natch:
    * абсолютные пути заменены на относительные (кроме образа)
    * добавлен скрипт для внесения изменений в скрипты запуска Natch
* Документация теперь доступна на github, а так же в виде PDF


[**Natch v.2.0**](https://nextcloud.ispras.ru/index.php/s/natch_v.2.0)

[(reserve) Natch v.2.0_ova](https://nextcloud.ispras.ru/index.php/s/natch_v.2.0_vbox)

* Представлен графический интерфейс SNatch v.1.0. Основные возможности:
    * построение графа взаимодействия процессов
        * интерактивный просмотр с помощью привязки к timeline
        * доступно четыре режима отображения сущностей
    * построение стека вызовов
* Улучшено распознавание модулей
* Добавлена поддержка сжатых секций с отладочной информацией
* Добавлена возможность фильтрации сетевых пакетов по протоколу
* Доработан скрипт для конфигурирования Natch
    * рабочая директория для проектов
    * добавлена возможность проброса портов в гостевую систему
* Запущен внешний баг-трекер


[**Natch v.1.3.2**](https://nextcloud.ispras.ru/index.php/s/natch_v.1.3.2)

* Улучшение и рефакторинг механизма распознавания модулей
* Поддержка набора инструкций SSE4.2 при отслеживании помеченных данных
* Настройка Natch теперь осуществляется с помощью одного скрипта
* Выходные файлы инструмента собираются в одну директорию
* Добавлен журнал событий Natch
* Небольшие изменения в настройке и конфигурационном файле Natch


[**Natch v.1.3.1**](https://nextcloud.ispras.ru/index.php/s/natch_v.1.3.1)

* Исправлена ошибка сбора покрытия для Ida 7.0
* Исправлена ошибка сохранения лога для помеченных параметров функций
* Исправлена опечатка в генерируемом конфигурационном файле
* Название снапшота вынесено в начало скрипта запуска Natch в режиме воспроизведения


[**Natch v.1.3**](https://nextcloud.ispras.ru/index.php/s/natch_v.1.3)

* Добавлена поддержка отладочной информации
* Добавлена поддержка map файлов, сгенерированных компилятором gcc
* Расширен набор опций конфигурационного файла Natch:
    * добавлена возможность указывать список файлов для пометки
    * добавлена возможность загрузки дополнительных плагинов
* Добавлен скрипт для генерации конфигурационного файла для модулей
* Обновлен скрипт для генерации командных строк запуска Natch
* Обновлено ядро эмулятора Qemu до версии 6.2


[**Natch v.1.2.1**](https://nextcloud.ispras.ru/index.php/s/natch_v.1.2.1)

* Исправлена ошибка работы утилиты qemu-img в VirtualBox под Windows 10
* Исправлена ошибка с генерацией имени оверлея в скрипте для генерации командных строк
* Добавлена возможность задавать поля скрипта перед его запуском


[**Natch v.1.2**](https://nextcloud.ispras.ru/index.php/s/natch_v.1.2)

* Скрипт для генерации командных строк
* Выгрузка данных о покрытии кода в IDA Pro
* Построение графа модулей, передающих друг другу помеченные данные
* Ранжирование функций поверхности атаки по числу обращений к помеченным данным
* Исправлены дефекты в механизме распространения пометок
* Мелкие изменения в конфигурационном файле инструмента


[**Natch v.1.1**](https://nextcloud.ispras.ru/index.php/s/natch_v.1.1)

* Возможность настраивать Natch с помощью конфигурационного файла
* Логирование входящих сетевых пакетов
* Отображение операций записи помеченных данных
* Построение графа, описывающего взаимодействие процессов
* Поддержка ELF32
* Исправление списка процессов: уничтожение завершившихся


[**Natch v.1.0**](https://nextcloud.ispras.ru/index.php/s/natch_v.1.0)

* Пометка сетевого трафика (сетевая карта e1000)
* Пометка файлов
* Возможность задания порогового значения для пометок
* Определение модулей с исполняемым кодом в памяти виртуальной машины
* Возможность подгружать map файлы из IDA
* Получение списка процессов, модулей и функций, участвующих в обработке помеченных данных
* Получение подробной трассы по каждому обращению к помеченным данным, включающей стек вызовов функций, адрес обращения к помеченным данным и количество помеченных байтов

