---
layout: docs
title: Установка тайл-сервера из готового пакета
permalink: /serving-tiles/building-a-tile-server-from-packages/
---

Если вы хотите собрать свой собственный тайл-сервер, но у вас более старая версия Ubuntu, то лучше всего использовать готовый пакет, так как это сэкономит много времени.

Эти пакеты работают с версиями Ubuntu Linux между 12.04 LTS (Precise Pangolin) и 14.04 LTS (Trusty Tahr). После установки у вас сразу должен быть свой собственный работающий тайл-сервер со стандартной таблицей стилей OSM Mapnik, в которую можно импортировать данные из OSM для генерации (рендеринга) тайлов.

Он основан на том же самом программном обеспечении, которое используется для создания собственного тайл-сервера OSM:

* mod_tile - модуль для Apache
* renderd - главный инструмент рендера
* mapnik - библиотека для рендера

Главная цель использования готовых пакетов - упрощение установки путем максимальной автоматизации.

# Установка

Для запуска установки необходимо ввести в терминал следующие команды:

Если у вас не установлен add-apt-repository, добавьте его с:

для Ubuntu 12.04

    sudo apt-get install python-software-properties

для Ubuntu 12.10 и более поздних версий

    sudo apt-get install software-properties-common

Добавьте репозиторий, содержащий пакеты:

    sudo add-apt-repository ppa:kakrueger/openstreetmap

Обновите список локальных пакетов, чтобы получить новый репозиторий:

    sudo apt-get update

Install the package libapache2-mod-tile and its dependencies. During configuration, it will ask you few questions. To make sure the automatic setup scripts work, you should keep the defaults. However, in the question about permissions for users of the database, you want to add your own username after the user www-data separated by a space to be able import data under your user.

    sudo apt-get install libapache2-mod-tile

# Иммпорт геоданных

Скачайте ту территорию из OpenStreetMap, тайлы которой вы бы хотели сгенерировать (отрендерить). Всю планету можно скачать с [Planet OSM](https://planet.openstreetmap.org/), а на [Geofabrik](http://download.geofabrik.de/) есть выгрузки по странам. Например, мы скачаем Северную Ирландию:

    wget http://download.geofabrik.de/europe/ireland-and-northern-ireland-latest.osm.pbf

Import the data into your postgresql database with osm2pgsql. There are a number of different parameters one can use with osm2pgsql that depend on your available hardware and the size of the data extract you want to import. The most likely ones you will need to set are ``-C``, ``--slim`` and ``-number-processes``. ``-C`` specifies the number of Mb osm2pgsql will use to cache data. So this depends on the amount of RAM you have available. ``--slim`` keeps the complete osm data on disk, necessary for updates and low memory environments. ``--number-processes`` specifies the number of parallel processes that are used for some parts of the import process. The optimal value mostly depends on the speed of your disk system and the available processor cores.

    osm2pgsql --slim -C 1500 --number-processes 4 ireland-and-northern-ireland.osm.pbf

Depending on the size of the extract you are importing and the performance of the computer, this can take anywhere from a few minutes for small extracts up to several days for the full planet on slower hardware! If you are importing the full planet, it is strongly recommended to set -C 18000 (18 GB of ram cache and may increase as the OSM database grows). This may cause your server to swap during the import if you don’t have a sufficient amount of memory, but in many cases it will still be faster than using smaller values for the node cache. However, you need to ensure you have sufficient swap memory configured.

If you are importing a full planet file, you might also want to use the –flat-nodes option. It uses a custom format file for some of the data rather than the postgis database which makes it more efficient, but less efficient for regional extracts. It may also make sense to temporarily change your PostgreSQL configurations during the import. (For example, increase the number of checkpoint segments, reduce shared buffer size.)

A large proportion of the import time as well as size of the database are used for creating indexes to keep track of updates. If you are not planning to keep the database continously up-to-date with “diff files”, you may want to import the data with the ``--drop`` option which will drop the “slim-tables” after import whih are not needed for rendering and does not create indexes which are only needed to support diff imports. Once you update less often than perhaps once every 1 – 2 weeks, it may be more efficient to do full reimports each time rather than use updates.

mod_tile is designed to always serve up-to-date tiles (see updating below). As it is generally not feasible to re-render all changed tiles at the time of change, mod_tile initiates re-rendering of outdated tiles at the time of serving. As such mod_tile needs to know when the planet was imported. This is done by changing the timestamp on the file planet-import-complete

    touch /var/lib/mod_tile/planet-import-complete

Почти все готово, нам осталось лишь перезапустить программу, выполняющую генерацию (рендер) тайлов. 

    sudo /etc/init.d/renderd restart

Если все прошло хорошо, то у вас уже должен быть запущен тайл-сервер. Результаты его работы можно посмотреть перейти по следующему адресу: http://localhost/osm/slippymap.html

If you are not opening slippymap.html on localhost and you are seeing only pink tiles, you will need to edit the html page and replace localhost with the correct server name in ‘new OpenLayers.Layer.OSM(“Local Tiles”…’ or change it to a relative URL.

# Обновление

С помощью следующих команд вы сможете обновлять свой тайл-сервер свежими данными из OSM.

После импорта исходной базы данных с помощью osm2pgsql, как это описано выше (вам нужно будет использовать опцию ``--slim`` во время проведения первого импорта, чтобы разрешить обновление), необходимо сделать следующее

Установить osmosis

    sudo apt-get install osmosis

Предоставьте права на обновление таблиц пользователю www-data

    sudo /usr/bin/install-postgis-osm-user.sh gis www-data

Initialise the osmosis replication stack to the data of your data import. Choose the date of the planet data, as this is the date from which the diffs will start.

    sudo -u www-data /usr/bin/openstreetmap-tiles-update-expire 2012-04-21

As the packaged script currently uses an outdated service to determine the correct replication start-point, you will need to manually choose and download the correct state.txt from the base_url (see below) which corresponds to slightly before the age of the extract to make sure all modifications are included in your db. This needs to be copied to /var/lib/mod_tile/.osmosis/state.txt

Далее вам необходимо обновить настройки osmosis по умолчанию. В файле configuration.txt (/var/lib/mod_tile/.osmosis/) измените base_url на следующий 
`https://planet.openstreetmap.org/replication/minute/`

Update your tileserver by up to an hour and expire the corresponding rendered tiles

    sudo -u www-data /usr/bin/openstreetmap-tiles-update-expire

If your tile server is behind more than an hour you will need to call the openstreetmap-tiles-expire script multiple times. If you want to continuously keep your server up to data, you need to add the openstreetmap-tiles-expire script to your crontab.

Keeping the data up-to-date can be resource intense, in particular because after the import you may already be multiple days behind. Consider changing maxInterval in /var/lib/mod_tile/.osmosis/configuration.txt to 21600 (six hours) till you have caught up. Further, add “–number-processes 2” to the osm2pgsql command in /usr/bin/openstreetmap-tiles-update-expire or a higher number if this is appropriate for your hardware.

The initial install installed pre-processed coastlines, from time to time it may make sense to replace the files with new versions:

    wget http://tile.openstreetmap.org/processed_p.tar.bz2

and extract it to

    /etc/mapnik-osm-data/world_boundaries/

# Поиск и устранение неисправностей

Есть много вещей, которые могут пойти не так. Вот несколько общих проблем и их решений:

## Сбой подключения к базе данных в osm2pgsql

If you are getting the following error message when importing data with osm2pgsql, you have probably forgotten to add your username in the allowed users section during the configuration. Connection to database Failed: FATAL: Ident authentication failed for user “xyz”

Вы можете исправить это одним из двух способов:

Either you reconfigure the package, this time including both www-data and your own user name in a space separated list,

    sudo dpkg-reconfigure openstreetmap-postgis-db-setup

Or, you manually grant permissions to your user account with the following command:

    sudo /usr/bin/install-postgis-osm-user.sh gis xyz

# Оборудование

Если вы хотите работать с достаточно большими по площади территориями, то необходимо очень мощное оборудование. Однако, если планируете генерировать (рендерить) тайлы для пространств, которые умещаются в 100-300 Мб, хватит и обычного настольного компьютера (4 ГБ ОЗУ, стандартный жесткий диск, 2-4 ядерный процессор), который обработает эти данные где-то за час.

Если же нужны тайлы всей планеты, то тут потребуется настоящий сервер с минимум 24 Гб оперативной памяти. Также настоятельно рекомендуем хранить базу данных на SSD или, по крайней мере, на быстром RAID-массиве. В данный момент выгрузка всей планеты "весит" порядка 256 Гб, так что для хранения всех БД на SSD, вам, скорее всего, понадобится 512 ГБ SSD. Если же вы не планируете после обновлять свою базу, скорее всего, вы сможете все уместить на 256 Гб SSD. Если нет возможности хранить всю БД на SSD, то туда можно перенести только наиболее частозапрашиваемые части БД, а остальное хранить на более медленных дисках. Osm2pgsql поддерживает использование для этой цели отдельных табличных пространств для разных частей базы данных.
# FAQ

## Где хранятся различные файлы (база данных, тайлы и пр.)?

* Таблицы стилей и береговый линии находятся в папке /etc/mapnik-osm-data
* Тайлы - /var/lib/mod_tile
* Файл конфигурации Renderd - /etc/renderd.conf
* Файл конфигурации mod_tile - /etc/apache2/sites-available/tileserver_site
* Скрипты - /usr/bin
* Конфигурация базы данных - /etc/postgresql/X.X/main (где X.X это версия PostgreSQL, например, 9.1)
* Файл конфигурации osm2pgsql и state.txt - /var/lib/mod_tile/.osmosis

## Как выполнить предварительную генерацию (рендер) тайлов?

Вы можете использовать render_list для предварительной генерации (рендера) тайлов:

    Usage: render_list [OPTION] ...
     -a, --all render all tiles in given zoom level range instead of reading from STDIN
     -f, --force render tiles even if they seem current
     -m, --map=MAP render tiles in this map (defaults to 'default')
     -l, --max-load=LOAD sleep if load is this high (defaults to 5)
     -s, --socket=SOCKET unix domain socket name for contacting renderd
     -n, --num-threads=N the number of parallel request threads (default 1)
     -t, --tile-dir tile cache directory (defaults to '/var/lib/mod_tile')
     -z, --min-zoom=ZOOM filter input to only render tiles greater or equal to this zoom level (default is 0)
     -Z, --max-zoom=ZOOM filter input to only render tiles less than or equal to this zoom level (default is 18)

If you are using –all, you can restrict the tile range by adding these options:

     -x, --min-x=X minimum X tile coordinate
     -X, --max-x=X maximum X tile coordinate
     -y, --min-y=Y minimum Y tile coordinate
     -Y, --max-y=Y maximum Y tile coordinate


Without –all, send a list of tiles to be rendered from STDIN in the format X Y Z, e.g.

    0 0 1
    0 1 1
    1 0 1
    1 1 1

The above would cause all 4 tiles at zoom 1 to be rendered

Note that you have to set ``--socket=/var/run/renderd/renderd.sock``

# Благодарность

Спасибо Каю Крюгеру за разработку пакетов и подготовку документации.
