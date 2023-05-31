Для корректного отображения лучше открыть в VSCode и нажать Ctr + Shift + V

# Содержание

1. [Получение справки](#help)
1. [Запуск контейнера](#Запуск)
1. [Просмотр (перечисление) контейнеров](#Просмотр)
1. [Очистка (удаление) контейнеров](#Удаление_rm)
1. [Остановка, запуск и перезапуск контейнеров](#Остановка_запуск)
1. [Политики перезапуска контейнеров](#Политики_перезапуска)
1. [Уничтожение контейнеров](#Уничтожение)
1. [Выполнение команд внутри контейнера](#Выполнение_команд_внутри)
1. [Присоединение (attach) к контейнеру](#Присоединение)
1. [Копирование файлов](#Копирование_файлов)
1. [Инспектирование контейнера](#Инспектирование)
1. [Docker system](#Docker_system)
1. [Образы (images)](#Образы)
1. [Создание нового образа из изменений контейнера](#Создание_нового)
1. [Dockerfile](#Dockerfile)
1. [Volumes (Тома)](#Volumes)
1. [Сеть](#Сеть)
1. [Docker compose](#compose)

# Введение

В Docker используется архитектура клиент-сервер, клиент Docker общается с демоном Docker который выполняет процессы создания, запуска, распределения контейнеров. Клиент и демон Docker взаимодействуют с помощью REST API, через сокеты UNIX или сетевой интерфейс. 

## Архитектура Docker

<br>
<details>
<summary>Архитектура Docker</summary>

![image](source/_static/architecture.png)

</details>
<br>

<o>*Репозиторий (реджестри)*</o> - это набор образов Docker c именем репозитория, но с ипользованием разных тегов.

<o>*Слой*</o> - это файлы которые хранятся в кэше Docker Engine, они имеют метаданные, необходимые для хранения иерархической информации о предке слоя и информации о выполнении и времени сборки. Слои могут совместно использоваться разными образами и разными контейнерами. При совместном использовании слоев их нельзя редактировать, в Docker это обеспечивается тем, что слои образов предоставляются только для чтения. Этот механизм позволяет уменьшить размер образов Docker и ускорить их сборку.

<o>*Dockerfile*</o> - это текстовый документ, в котором содержатся все команды, которые пользователь может вызвать в командной строке для сборки образа. Dockerfile состоит из инструкций, комментариев, директив синтаксического анализа и пустых строк. Любая инструкция выполняет конкретное действие в <o>новом слое</o> поверх текущего образа и создает новый слой, который доступен для последующих шагов в Dockerfile.

<o>*Контекст сборки*</o> - это набор файлов, расположенных в указанной папке PATH или URL.

<o>*Образ (image)*</o> - это шаблон только для чтения с инструкциями по созданию контейнера Docker, другими словами - это логическая коллекция слоев образа. Для создания собственного образа необходимо создать Dockerfile, описать необходимые шаги (команды), каждая команда в Dockerfile создает слой в образе. При изменении Dockerfile и пересборке образа пересобираются только те слои которые изменились, начиная  самого "нижнего".

<o>*Контейнер*</o> - это исполняемый экземпляр образа. Его можно запускать, останавливать, перемещать или удалять с использованием Docker API или командной строки. Также контейнер можно подключать к одной или нескольким сетям, присоединить к нему хранилище или создать новый образ на основе его текущего состояния. При этом можно контролировать степень изоляции сети, хранилища или других подсистем контейнера. При удалении контейнера все изменения в его состоянии, которые не хранились в постоянном хранилище, исчезают. Все это обеспечивается использованием пространства имен ядра и механизма cgroups. 
Контейнеры работают, только когда выполняется приложение внутри контейнера, когда процесс выполнения приложения завершается, контейнер переходит в состояние выхода. После выхода он ни куда не исчезает, не использует ЦП или память, но занимает дисковое пространство. Их можно повторно запустить, проверить логи, скопировать файлы в файловую систему контейнера или из нее. Docker не удаляет неработающий контейнер если прямо не указать на это (флаг --rm).

<br>
<details>
<summary>Жизненный цикл контейнера:</summary>

![image](source/_static/life_cicle.png)

</details>
<br>

<br>
<details>
<summary>Полный жизненный цикл контейнера:</summary>

![image](source/_static/full_life_cicle.png)

</details>
<br>

## Команды Docker и их использование

1. <g>Получение справки:</g> <a name="help"></a>
    - общая справка
        ```bash
        docker help
        ```
    - справка по команде
        ```bash
        docker login --help
        ```
1. <g>Запуск контейнера:</g> <a name="Запуск"></a>
    - команда <o>run</o> осуществляет создание контейнера и его запуск, в отичии от команды <o>create</o> которая только создает контейнер
        ```bash
        docker container run dxctraining/docker-demo-about
        ```
    - в интерактивном режиме (<o>-i</o>) с подкюлчением к терминальному сеансу внутри контейнера (<o>-t</o>)
        ```bash
        docker container run -it dxctraining/docker-demo-about /bin/bash
        ```
    - с удалением контейнера после остановки (<o>--rm</o>)
        ```bash
        docker container run --rm dxctraining/docker-demo-about
        ```
    - с присвоением имени (<o>--name</o>)
        ```bash
        docker container run --name demo dxctraining/docker-demo-about
        ```
    - в фоновом режиме, в режиме демона (<o>-d</o>) с перенаправлением порта контейнера 80 на порт 8080 (<o>-p</o>) ip-адрес указываем из сети хостовой машины или вообще не указыаем 
        ```bash
        docker container run --rm -d -p 192.168.2.150:8080:80 ngnix
        ```
    - c автогенерацией порта (<o>::</o>)
        ```bash
        docker container run --rm -d -p 192.168.2.150::80 nginx
        ```
        <br>
        <details>
        <summary>Схема открытия портов:</summary>

        ![image](source/_static/open_port.png)

        </details>
        <br>

    - перечисление открытых портов или конкретного порта
        ```bash
        docker container port nginx

        docker container port nginx 80
        ```
1. <g>Просмотр (перечисление) контейнеров:</g> <a name="Просмотр"></a>
    - просмотр (перечисление) запущеных контейнеров
        ```bash
        docker container ls или ps
        ```
    - просмотр (перечисление) остановленных контейнеров
        ```bash
        docker ps -a
        ```
    - или с использованием фильтра (<o>-f</o>) для вывода контейнеров с определенным статусом (регистр должен быть нижним)
        ```bash
        docker container ls -f 'status=exited'
        ```
    - переименование (rename)
        ```bash
        docker rename demo demo1
        ```
    - просмотр логов контейнера
        ```bash
        docker container logs nginx
        ```
1. <g>Очистка (удаление) контейнеров:</g> <a name="Удаление_rm"></a>
    - удаление запущеонго контейнера
        ```bash
        docker container rm id (или его уникальная часть) или имя контейнера
        ```
    - удаление запущеонго контейнера без подтверждения (<o>-f</o>)
        ```bash
        docker container rm -f dxctraining/docker-demo-about
        ```
    - удаление сразу всех остановленных контейнеров (<o>-q</o> вывод только id контейнера)
        ```bash
        docker container rm $(docker container ls -f 'status=exited' -q)
        ```
    - удаление без запроса на подтверждение (<o>-f</o>) остановленных более 5 мин (<o>--filter</o>)
        ```bash
        docker container prune -f --filter "until=5m"
        ```
    - удаление всех не используемых данных из системы Docker, таких как остановленные контейнеры, образов и сетей которым не назначено имя
        ```bash
        docker system prune -f
        ```
    - удаление всех не используемых данных из системы Docker, таких как остановленные контейнеры, образов и сетей которым не назначено имя и неиспользуемые образы (<o>-a</o>)
        ```bash
        docker system prune -f -a
        ```
1. <g>Остановка, запуск и перезапуск контейнеров:</g> <a name="Остановка_запуск"></a>
    - предполложим у нас есть остаонвленный контейнер stress (передаваемые параметры: <o>stress --cpu 1 --io 1 --vm 1 --vm-bytes 1G</o> создают нагрузку)
        ```bash
        docker container run --name stress -d polinux/stress stress --cpu 1 --io 1 --vm 1 --vm-bytes 1G
        ```
    - запуск контейнера
        ```bash
        docker container start stress
        ```
    - отановка контейнера (или остановка через 20 секунд, с параметром <o>-t 20</o>)
        ```bash
        docker container stop stress
        ```
    - рестарт (тоже самое что stop + start)
        ```bash
        docker container restart stress
        ```
    - приостановить работу
        ```bash
        docker container pause stress
        ```
    - возобноивть работу
        ```bash
        docker container unpause stress
        ```
    - запуск остановленного контейнера с прилагаемой оболочкой к нему
        ```bash
        docker container start -ai nginx
        ```
1. <g>Политики перезапуска контейнеров:</g> <a name="Политики_перезапуска"></a>

    При запуске контейнера используя флаг <o>--restart</o> можно установить политику перезапуска контейнера, т.е. следует или неследует перезапускать контейнер при выходе. 
    <br>
    <details>
    <summary>Политики перезапуска контейнеров:</summary>

    ![image](source/_static/policy.png)

    </details>
    <br>

    Для примера используем контейнер с образом busybox, который по сути запускает оболучку sh и завершает свою работу, если нет команды на входе. В нашем случае, перед выходом, выведем три раз текущую дату (echo Start; for i in \$(seq 1 3); do date; sleep 1; done). 
    ```bash
    docker container run --name busybox -d busybox sh -c "echo Start; for i in \$(seq 1 3); do date; sleep 1; done"
    ```
    проверив лог контейнера <o>docker container logs busybox</o> увидим одну последовательность вывода даты, т.е. контейнер после выхода не перезапустился
    ```bash
    Start
    Tue May 23 09:50:52 UTC 2023
    Tue May 23 09:50:53 UTC 2023
    Tue May 23 09:50:54 UTC 2023
    ```
    теперь изменим политику на <o>always</o>
    ```bash
    container run --restart always --name busybox -d busybox sh -c "echo Start; for i in \$(seq 1 3); do date; sleep 1; done"
    ```
    проверив лог контейнера <o>docker container logs busybox</o> увидим несколько последовательностей вывода даты, т.е. контейнер после выхода начал перезапускаться
    ```bash
    Start
    Tue May 23 10:13:34 UTC 2023
    Tue May 23 10:13:36 UTC 2023
    Tue May 23 10:13:37 UTC 2023
    Start
    Tue May 23 10:13:38 UTC 2023
    Tue May 23 10:13:39 UTC 2023
    Tue May 23 10:13:40 UTC 2023
    Start
    Tue May 23 10:13:42 UTC 2023
    Tue May 23 10:13:43 UTC 2023
    ```
1. <g>Уничтожение контейнеров:</g> <a name="Уничтожение"></a>
    - запускаем тестовый контейнер
        ```bash
        docker container run --name signals -it dxctraining/docker-demo-signals
        ```
    - проверяем разные варианты уничтожения контейнера (<o>kill --signal=</o>) (из другой консоли)
        ```bash
        docker container kill --signal=USR1 signals
        docker container kill --signal=10 signals
        docker container kill --signal=SIGKILL signals
        ```
1. <g>Выполнение команд внутри контейнера:</g> <a name="Выполнение_команд_внутри"></a>
    - вывод содержимого каталога (<o>ls -al</o>) в контейнере (параметр <o>exec</o>)
        ```bash
        docker container exec nginx ls -al /usr/share/nginx/html
        ```
1. <g>Присоединение (<o>attach</o>) локальных стандартных входных, выходных данных и потока сообщений об ошибках к выполняемому контейнеру:</g> <a name="Присоединение"></a>
    <br>
    <details>
    <summary>Опиции attach, detach:</summary>

    ![image](source/_static/attach.png)

    </details>
    <br>

    Открепление зависит от опций выполнения и может быть сделано тремя способами:

    Запустим контейнер nginx, без парамера (<o>-it</o>) в режиме демона
    ```bash
    docker container run -d --name nginx -p 8080:80 nginx
    ```
    1. подключаемся используя опцию (<o>--sig-proxy=false</o>)
        ```bash
        docker attach --sig-proxy=false nginx
        ```
        для открепления выхода нужно нажать Ctrl+C (что соответствует таблице выше) контейнер при этом продолжит работать
    2. подключаемся (по умолчанию опция --sig-proxy=true)
        ```bash
        docker attach nginx
        ```
        для открепления выхода нужно нажать Ctrl+C но при этом контейнер по SIGINT будет остановлен.

    Теперь запустим контейнер nginx, с парамера (<o>-it</o>) в режиме демона (см. таблицу выше, первая колонка)
    ```bash
    docker container run -it -d --name nginx -p 8080:80 nginx
    ```
    3. подключаемся используя опцию (<o>--no-stdin</o>)
        ```bash
        docker attach --no-stdin nginx
        ```
        для открепления выхода нужно нажать Ctrl+C контейнер при этом продолжит работать.
1. <g>Копирование файлов и папок между контейнером и хостовой системой:</g> <a name="Копирование_файлов"></a>
    - из контейнера
        ```bash
        docker container cp nginx:/usr/share/nginx/html/index.html .
        ```
    - в контейнер
        ```bash
        docker container cp ./index.html nginx:/usr/share/nginx/html
        ```
1. <g>Инспектирование контейнера, вывод подробной информации о контейнере:</g> <a name="Инспектирование"></a>
    - просмотреть подробные сведения
        ```bash
        docker container inspect nginx
        ```
    - просмотреть env (найти в выводе (<o>env</o>) и вывести 5 строк ниже (<o>-A 5</o>))
        ```bash
        docker container inspect nginx | grep -i env -A 5
        ```
1. <g>Docker system</g> <a name="Docker_system"></a>
    - просмотр сколько освободится места после очистки (RECLAIMABLE)
        ```bash
        docker system df
        ```
    - получение событий с сервера в реальном времени
        ```bash
        docker system events
        ```
    - вывод системной информации о сервере(системе)
        ```bash
        docker system info
        ```
        ```bash    
        ...
        Server:
          Containers: 7
            Running: 1
            Paused: 0
            Stopped: 6
          Images: 22
        ...
        ```
1. <g>Образы (images):</g> <a name="Образы"></a>
    - подключение к репозиторию
        ```bash
        docker login -u LOGIN -p PASSWORD registry.docker.io
        ```
    - извлечение образа из репозитория или реестра
        ```bash
        docker image pull nginx
        ```
    - отображение слоев(истории) образа (для полного вывода можно добавить опцию <o>--no-trunc</o>)
        ```bash
        docker image history nginx
        ```    
    - перечисление образов
        ```bash
        docker image ls
        ``` 
    - сборка образа (параметр <o>builder</o> - дает расширенные возможности сборки) "<o>.</o>" указывает на каталог с контекстом сборки
        ```bash
        docker builder build .
        ```
    - добавление тега (в качестве old_tag_name можно указать хэш)
        ```bash
        docker image tag old_tag_name new_tag_name
        ```
    - добавление тега (<o>-t</o>) при сборке
        ```bash
        docker builder build -t tag_name .
        ```
    - удаление образов
        ```bash
        docker image rm nginx
        ```
    - удаление неиспользуемых образов, если указать параметр <o>-a</o> будут удаены и все образы на которые не ссылаются ни какие контейнеры
        ```bash
        docker image prune
        ```
1. <g>Создание нового образа из изменений контейнера:</g> <a name="Создание_нового"></a>
    - создание нового образа без тега
        ```bash
        docker commit 448a08f1d2f9
        ```  
    - создание нового образа с тегом
        ```bash
        docker commit container_name new_image_name:tag_name
        ```
    - запустим контейнер и поставим nano
        ```bash
        docker container run --name a_temp -it alpine
        apk add nano
        ```
        получили новый контейнер <o>a_temp</o> базирующийся на образе <o>alpine</o>
        ```bash
        CONTAINER ID   IMAGE     COMMAND     CREATED          STATUS                      PORTS     NAMES
        2eebc7f86c14   alpine    "/bin/sh"   58 seconds ago   Exited (0) 20 seconds ago             a_temp
        ```
        создадим собственный образ alpine_dev из контейнера a_temp (используя опцию <o>-m</o> можно оставить коментарий, используя опцию <o>-a</o> можно указать информацию об авторе, а используя опцию <o>-с</o> можно добавить свою переменную окружения)
        ```bash
        docker commit -m "Added nano" -a "My Name xxx@xxx.ru" -c "ENV NEW_VAR=VALUE" a_temp alpine_dev
        ```
        в результате посмотрев имеющиеся образы увидим 
        ```bash
        docker image ls
        REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
        alpine_dev   latest    ce44ab37b90e   8 seconds ago   9.96MB
        alpine       latest    5e2b554c1c45   2 weeks ago     7.33MB
        ```
        ```bash
        docker image history alpine_dev
        IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
        ce44ab37b90e   7 minutes ago   /bin/sh                                         2.64MB    Added nano
        5e2b554c1c45   2 weeks ago     /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
        <missing>      2 weeks ago     /bin/sh -c #(nop) ADD file:7625ddfd589fb824e…   7.33MB
        ```
        ```bash
        docker image inspect alpine_dev | grep -i author -A 1 -B 1
        "DockerVersion": "23.0.5",
        "Author": "My Name xxx@xxx.ru",
        "Config": {
        ```
        создадим контейнер из полученного образа и проверим наличие <o>nano</o> и новой переменной окружения <o>NEW_VAR</o>
        ```bash
        docker container run --name alpine_dev -it alpine_dev
        ```
        попав в консоль видим что <o>nano</o> установлен и присутствует переменная окружения <o>NEW_VAR</o>
        ```bash
        / # nan
        nanddump   nandwrite  nano
        / # env | grep -i new_var
        NEW_VAR=VALUE
        ```
        посмотрев имеющиеся контейнеры увидим созданый контейнер <o>alpine_dev</o> базирующийся на образе <o>alpine_dev</o>
        ```bash
        docker container ls -a
        CONTAINER ID   IMAGE        COMMAND     CREATED          STATUS                      PORTS     NAMES
        8074ba9b055c   alpine_dev   "/bin/sh"   14 seconds ago   Exited (0) 5 seconds ago              alpine_dev
        2eebc7f86c14   alpine       "/bin/sh"   21 minutes ago   Exited (0) 20 minutes ago             a_temp
        ```
1. <g>Dockerfile:</g> <a name="Dockerfile"></a>
    <br>
    <details>
    <summary>Dockerfile</summary>

    ![image](source/_static/dockerfile.png)

    </details>
    <br>

    В dockerfile используются такие инструкции как:

    - `FROM <image>[:<tag>][AS <name>]` - указывает Docker Engine, какой базовый образ необходимо использовать для последующих инструкций. Каждый Dockerfile дожен начинаться с инструкции FROM.
    - `WORKDIR <path>` - устанавливает текущую рабочую директорию для инструкций: COPY, ADD, RUN, CMD, ENTRYPOINT. Ее можно указывать много раз. При этом если указывать относительный путь (без /), он будет относительным к пути предыдущей инструкции WORKDIR (т.е. можно формировать вложеные каталоги относительно предыдущей инструкции.)
    - `COPY [--chown=<user>:<group>] <src> ... <src> <dest>` - копирует новые файлы или директории из <src> и добавляет их в файловую систему контейнера по пути <dest>. Если <src> является директорией, копируется все содержимое директории, включая метаданые файловой системы. Если после <dest> не стоит завершающий слеш, но будет рассматриваться как файл, и содержимое <src> будет записано в него. Если указано несколько ресурсов <src> то <dest> должен быть директорией.
    - `ADD - [--chown=<user>:<group>] <src> ... <src> <dest>` - аналогична команде COPY, но при этом позвояет: загружать файлы по URL и извлекать файлы из архива (локальная распаковка), копируя их в определенное место.
    - `RUN` - выполняет любые команды. Имеет две формы: <o>shell</o> для выполнения команд в оболочке - `RUN <command>` и <o>exec</o> для выполнения команд без shell (нет расширений shell, вроде * или $HOME) - `RUN ["executble", "parameter 1", "parameter 2"]` Команды можно объединять используя && и \
    - `ARG <name>[=<default value>]` - позволяет задать переменную, значение которой можно передать из командной строки в образ во время его сборки (`docker build --build-arg <name>=<value>`). Значение по умолчанию можно представить в Dockerfile.
    - `ENV <key>=<value>` - устанавливает переменную среды <o>key</o> со значением <o>value</o>. Обратные слеши (`\`) можно использовать для включения пробелов в значения (ENV NAME=My\ Name аналогично ENV NAME="My Name"). Если пременная среды необходима только во время сборки, а не в окончательном образе, можно установить значение для одной команды: `RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y ...` Значение <o>ENV</o> может быть изменено при запуске контейнера опцией `--env` например `docker container run --env <key>=<value>`
    - ENTRYPOINT и CMD - определяют какие инструкции будут выолняться при запуске контейнера. Обе инструкции имеют формы <o>shell</o> и <o>exec</o>. Docker конкатенирует аргументы ENTRYPOINT и CMD и выполняет их как финальную команду. Следующая таблица показывает варианты их использования:
        <br>
        <details>
        <summary>ENTRYPOINT и CMD:</summary>

        ![image](source/_static/cmd.png)

        </details>
        <br>
        В случае формы <o>shell</o> у процесса не будет PID=1, а в случае формы <o>exec</o> будет.
    - в dockerfile используются и другие инструкции: LABEL, EXPOSE, USER и т.п.

    .dokerignore - исключение файлов в контекстной директории для передачи в процессе сборки 
1. <g>Volumes (Тома):</g> <a name="Volumes"></a>

    В Docker существуют различные стратегии хранения данных:
    1. <o>Bind mounts</o> - для подключения директории или файла хостовой машины внутри контейнера
        <br>
        <details>
        <summary>Bind mounts:</summary>

        ![image](source/_static/bind_mounts.png)

        </details>
        <br>
        Флаг <o>--volumes</o> или <o>-v</o> подключает указанную директорию к контейеру:

        `-v HOST_PATH:CONTAINER_MOUNT_PATH[:READ_WRITE_MODE]`

            - HOST_PATH - аболютный путь на хостовой машине
            - CONTAINER_MOUNT_PATH - аболютный путь в файловой системе контейнера
            - READ_WRITE_MODE - может быть либо режимом read-only/ro, либо режимом read-write/rw
        Флаг <o>--mount</o> подключает указанную директорию к контейеру:

        `--mount type=bind,sourse=HOST_PATH,target=CONTAINER_MOUNT_PATH[,READ_ERITE_MODE]`

            - READ_WRITE_MODE - может быть либо режимом read-only/ro, либо ничем
        
        создадим файл mes.txt с каким нибудь содержимым и подключим к контейнеру
        ```bash
        docker container run --rm -it --name test-mount \
        -v "$(pwd)/mes.txt":/data/message.txt:ro \
        bash sh -c "while true; do cat /data/message.txt; sleep 3; done"
        ```
        по умолчанию внутри контейнера у вас есть полномочия root при необходимости изменить права на файл необходимо при запуске контейнера добавить параметр <o>-u \$(id -u):\$(id -g)</o> 
    1. <o>Volumes</o> - для хранения данных внутри выделенного хранилища
        <br>
        <details>
        <summary>Volume:</summary>

        ![image](source/_static/volume.png)

        </details>
        <br>

        Тома делятся на:
        - именованные - пользователь присваивает тому имя при выполнении команды `docker volume create`. Затем это имя используется при запуске контейнера
            ```bash
            docker volume create my_vol
            docker container run -v my_vol:/CONTAINER_MOUNT_PATH my_image
            ```
        - анонимные - том не получает явного имени при подключении к контейнеру, поэтому Docker присваивает такому тому произвольное уникальное имя. 
            ```bash
            docker container run -v :/CONTAINER_MOUNT_PATH my_image
            docker container run my_image
            ```
        - также тома можно создавать на ходу (при использовани опции <o>--rm</o> также происходит удаление тома, а если том был создан ранее, то флаг <o>--rm</o> на него не повлияет)
            ```bash
            docker container run --rm -v my_vol:/data alpine ls -al /data
            total 8
            drwxr-xr-x    2 root     root          4096 May 29 10:12 . 
            drwxr-xr-x    1 root     root          4096 May 29 10:12 ..

            docker volume ls
            DRIVER    VOLUME NAME
            local     adbb33eec06d7dc7f1ba9b64a5195da677d73906a52680d30f29e45a1c2d089d
            local     my_vol
            ```
        Просмотр тома можно осуществить двумя способами:
        1. используя флаг <o>--volume</o> или <o>--mount</o>
        1. используя флаг <o>--volumes-from</o> - был создан для простого совместного использования тома контейнерами. Он подключается к тому же пути с теми же опциями, с какими он был подключен к исходному контейнеру.
            
            Создадим имидж из Dockerfile
            ```bash
            FROM alpine
            RUN mkdir /my_vol && echo One > /my_vol/mes.txt
            VOLUME /my_vol
            CMD while true; do \
                    cat /my_vol/mes.txt; \
                    sleep 4; \
                done
            ```
            ```bash
            docker build -t test-vol .
            ```
            Запустим контейнер <o>test</o> c подключенным томом <o>my_vol</o>
            ```bash
            docker container run -it --name test test-vol
            ```
            Запустим второй контейнер и убедимся что том подмонтировался и в него
            ```bash
            docker container run --rm --volumes-from test -it busybox
            / # mount | grep my_vol
            /dev/sde on /my_vol type ext4 (rw,relatime,discard,errors=remount-ro,data=ordered)
            / # ls -l /my_vol/
            total 4
            -rw-r--r--    1 root     root             4 May 29 10:49 mes.txt
            / #
            ```
        - создание тома именованного
            ```bash
            docker volume create my_vol
            ```
        - создание тома анонимного
            ```bash
            docker volume create
            ```
        - создание сетевого тома с использованием драйвера
            ```bash
            docker volume create --driver local \
            --opt type=nfs \
            --opt o=addr=192.168.1.1,rw \
            --opt device=:/path/to/dir \
            nfs-drive
            ```
            ```bash
            docker volume create --driver local \
            --opt type=tmpfs \
            --opt device=tmpfs \
            --opt o=size=100m,uid=1000 \
            mem-drive
            ```
        - вывод информации о томах `docker volume inspect [OPTIONS] VOLUME [VOLUME ...]`
            ```bash
            docker volume inspect my_vol
            ```
            ```bash
            docker volume ls
            ```
        - вывод списка томов
            ```bash
            docker volume ls
            ```
        - удаление томов `docker volume rm [OPTIONS] VOLUME [VOLUME ...]`
            ```bash
            docker volume rm my_vol
            ```
        - удаление всех неиспользуемых томов (те, на которые не ссылаются контейнеры)
            ```bash
            docker volume prune
            ```
    1. <o>Tmpfs mounts</o> - хранятся только в памяти хостовой машины и никогда не записываются в ее файловую систему
1. <g>Сеть</g> <a name="Сеть"></a>

    Сетевую подсистему Docker можно подключать с помощью драйверов. Имеется несколько драйверов по умолчанию, которые обеспечивают основные сетевые возможности:
    - none - отключение всех сетевых ресуров.
        <br>
        <details>
        <summary>Network none</summary>

        ![image](source/_static/none.png)

        </details>
        <br>
    - bridge - сетевой драйвер по умолчанию (все контейнеры запущеные без флага --network подключаются к мостовой сети по умолчанию). bridge (мостовые) сети обычно используются, когда приложения выполняются а автономных контейнерах, которые должны взаимодействовать друг с другом. При этом обеспечивается изоляция от контейнеров которые не подкюлчены к данной мостовой сети. Сетевые параметры мостовой сети по умолчанию изменить нельзя. Можно создать определяемый пользователем мост, который обеспечивает автоматическое определение DNS, более хорошую изоляцию, является настраиваемым мостом.
        <br>
        <details>
        <summary>Network bridge</summary>

        ![image](source/_static/bridge.png)

        </details>
        <br>
        <br>
        <details>
        <summary>Network user bridge</summary>

        ![image](source/_static/bridge2.png)

        </details>
        <br>
    - host - для автономных контейнеров устраняется сетевая изолированность между контейнером и хостовой машиной и напрямую используются сетевые ресурсы хостовой машины. Сетевой стек контейнера не изолирован от хостовой машины и не получает собственного выделенного IP-адреса. Не требуется преобразование сетевых адресов (NAT). Распределение портов, с данным типом драйвера, не работает.
        <br>
        <details>
        <summary>Network host</summary>

        ![image](source/_static/host.png)

        </details>
        <br>
    - overlay - наложенные сети соединяют несколько демонов Docker.
    - macvlan - данные сети позволяют присваивать контейнеру MAC-адрес, благодоря чему он выглядит как физическое устройство в сети.

    также возможно использование сторонних сетевых плагинов.
    Если необходимо полностью отключить сетевой стек в контейнере, то при запуске контейнера можно использовать флаг <o>--network none</o>. В этом случае в контейнере создается только устройство закольцовывания (loopback адрес (127.0.0.1)).
    - просмор списка сетей
        ```bash
        docker network ls
        ```
    - вывод подробной информации об одной или нескольких сетях
        ```bash
        docker network inspect
        ```
    - создание сети (<o>-d</o> - драйвер)
        ```bash
        docker network create -d bridge user_bridge
        ```
    - создание контейнера в определяемом пользователем мосте
        ```bash
        docker container run --rm -d -it --network user_bridge --name user_bridge1 alpine
        docker container run --rm -d -it --network user_bridge --name user_bridge2 alpine
        ```
        просмотрим перечень контейнеров в сети <o>user_bridge</o>
        ```bash
        docker network inspect user_bridge
        "Containers": {
            "1ede4322133c4644acff9f6ac172e8a0fbabc9fd25d8d29443e30680fe127f0f": {
                "Name": "user_bridge2",
                "IPv4Address": "172.19.0.3/16"
            },
            "fd5e5548e792e4caaed898a6e41698c5bed5d872f4b19d7e3cc2acae3105e4fc": {
                "Name": "user_bridge1",
                "IPv4Address": "172.19.0.2/16"
            }
        ```        
    - подключение контейнера к сети (контейнер def_bridge1 ранее был подключен к дефолтной сети bridge)
        ```bash
        docker network connect user_bridge def_bridge1
        ```
    - отключение контейнера от сети
        ```bash
        docker network disconnect user_bridge def_bridge1
        ```
    - удаление сети (возможно только после остановки контейнеров в этой сети)
        ```bash
        docker network rm user_bridge
        ```
    - удаление всех неиспользуемых сетей
        ```bash
        docker network prune
        ```
1. <g>Docker compose</g> <a name="compose"></a>

    Compose - это инструмент для определения и запуска мультиконтейерных приложений Docker. Для настройки сервисов приложений можно использовать файл YAML. Затем с помощью одной команды можно создать и запустить все сервисы в данной конфигурации.
    Пример docker-compose.yaml
    ```yaml
    version: "3.5"
    services:
        cluster-1:
            container_name: cluster-1
            image: nats
            ports:
                - "8111:8222"
                - "5001:4222"
            volumes:
                - /home/nats2:/etc/nats
            command: "-c /etc/nats/n1a.conf"
            networks:
                - nats
                - shar
        nats-1:
            container_name: Node-1-1_C1
            image: nats
            ports:
                - "8112:8222"
                - "5002:4222"
            volumes:
                - /home/nats2:/etc/nats
            command: "-c /etc/nats/n1b.conf"
            networks:
                - nats
                - shar
            depends_on: ["cluster-1"]
        cluster-2:
            container_name: cluster-2
            image: nats
            ports:
                - "8320:8222"
                - "5301:4222"
            volumes:
                - /home/nats2:/etc/nats
            command: "-c /etc/nats/n2a.conf"
            networks:
                - nats2
                - shar
            depends_on: ["nats-1"]
        nats-2:
            container_name: Node-2-1_C2
            image: nats
            ports:
                - "8221:8222"
                - "5202:4222"
            volumes:
                - /home/nats2:/etc/nats
            command: "-c /etc/nats/n2b.conf"
            networks:
                - nats2
                - shar
            depends_on: ["cluster-2"]
        nats-2-2:
            container_name: Node-2-2_C2
            image: nats
            ports:
                - "8222:8222"
                - "5203:4222"
            volumes:
                - /home/nats2:/etc/nats
            command: "-c /etc/nats/n2c.conf"
            networks:
                - nats2
                - shar
            depends_on: ["cluster-2"]

    networks:
    shar:
        name: shar
    nats:
        name: nats
    nats2:
        name: nats2
    ```
    - для запуска используется команда
    ```bash
    docker-compose -f docker-compose.yaml up
    ```
    - для остановки используется команда
    ```bash
    docker-compose -f docker-compose.yaml stop|down
    ```
    - просмотр запущеных контейнеров
    ```bash
    docker-compose ps
    ```
    - просмотр логов
    ```bash
    docker-compose logs
    docker-compose logs cluster-1
    ```

<style>
o { color: Orange }
g { color: lightGreen }
</style>