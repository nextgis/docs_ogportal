.. sectionauthor:: Дмитрий Барышников <dmitry.baryshnikov@nextgis.ru>
.. sectionauthor:: Денис Рыков <denis.rykov@nextgis.ru>

.. _ngogportal_install:

Установка CKAN в CentOS 7
=========================

За основу взята инструкция по установке из
`официальной документации <http://docs.ckan.org/en/latest/maintaining/installing/install-from-source.html>`_.


Установка системных пакетов
---------------------------

Устанавливаем необходимые пакеты:

.. code-block:: bash

    sudo yum install epel-release
    sudo yum install python-pip python-virtualenv python-devel
    sudo yum install git gcc-c++ java-1.8.0-openjdk unzip lsof libxml2-devel libxslt-devel mailcap
    sudo yum install postgresql postgresql-server postgresql-libs postgresql-contrib postgresql-devel

.. warning::
   Если на сервере, куда устанавливается CKAN, уже установлен NextGIS Web,
   то пакеты для PostgreSQL ставить не нужно. В этом случае PostgreSQL
   уже должен быть установлен и настроен.


Установка CKAN
--------------

Создаём пользователя CKAN:

.. code-block:: bash

    sudo useradd -m -s /sbin/nologin -d /usr/lib/ckan -c "CKAN User" ckan

Домашняя папка пользователя CKAN остальным пользователям системы
доступна только на чтение:

.. code-block:: bash

    sudo chmod 755 /usr/lib/ckan

Переключаемся на только что созданного пользователя:

.. code-block:: bash

    sudo su -s /bin/bash - ckan

Устанавливаем виртуальное окружение ``default``:

.. code-block:: bash

    virtualenv --no-site-packages default

И активируем его:

.. code-block:: bash

    . default/bin/activate

Скачиваем и устанавливаем последнюю версию CKAN. На момент написания
данной инструкции это версия CKAN 2.5.2. Если вы устанавливаете
другую версию, то измените URL в следующей команде:

.. code-block:: bash

    pip install -e git+https://github.com/okfn/ckan.git@ckan-2.5.2#egg=ckan

Устанавливаем пакеты Python, от которых зависит CKAN:

.. code-block:: bash

    pip install -r default/src/ckan/requirements.txt


Настройка PostgreSQL
--------------------

Инициализируем базу данных и включаем автоматический запуск
PostgreSQL при старте системы:

.. code-block:: bash

    sudo postgresql-setup initdb
    systemctl start postgresql
    systemctl enable postgresql

.. warning::
   В ходе работы с CKAN размер базы будет увеличиваться, поэтому
   в случае необходимости кластер базы данных должен быть вынесен
   в папку с достаточным объёмом дискового пространства.

Создаём пользователя базы данных ``ckan_default`` и
саму базу также названную ``ckan_default``:

.. code-block:: bash

    sudo -u postgres createuser -S -D -R -P ckan_default
    sudo -u postgres createdb -O ckan_default ckan_default -E utf-8

Отредактируем параметры аутентификации в соответствующем файле:

.. code-block:: bash

    sudo nano /var/lib/pgsql/data/pg_hba.conf

Отредактируем его таким образом, чтобы в нём присутствовали следующие
строки (исправим метод аутентификации на ``md5``, если указан иной):

.. code-block:: bash

    # IPv4 local connections:
    host    all             all             127.0.0.1/32            md5
    # IPv6 local connections:
    host    all             all             ::1/128                 md5

.. warning::
   Предполагается, что CKAN и PostgreSQL установлены на одном хосте.
   Если это не так, то потребуется дополнительная настройка PostgreSQL.

Не забудьте перезапустить PostgreSQL:

.. code-block:: bash

    systemctl restart postgresql

.. warning::
   Если вы устанавливали PostgreSQL из стороннего репозитория,
   например `отсюда <http://yum.postgresql.org/>`_, то может
   потребоваться дополнительная настройка переменной ``PATH``,
   либо придётся писать полный путь до команд, например,
   ``/usr/pgsql-9.5/bin/psql``.


Создание конфигурационного файла CKAN
-------------------------------------

.. code-block:: bash

    sudo mkdir -p /etc/ckan/default
    sudo chown -R ckan /etc/ckan/

Переключаемся на пользователя CKAN и создаём конфигурационный файл:

.. code-block:: bash

    sudo su -s /bin/bash - ckan
    . default/bin/activate
    cd /usr/lib/ckan/default/src/ckan
    paster make-config ckan /etc/ckan/default/development.ini

Отредактируйте файл ``development.ini``, указав пароль пользователя
``ckan_default`` и URL сайта, по которому будет доступен CKAN:

.. code-block:: bash

    sqlalchemy.url = postgresql://ckan_default:pass@localhost/ckan_default
    ckan.site_url = http://82.162.194.216
    ckan.root_path = /ckan/{{LANG}}


Установка Solr5
---------------

Скачиваем Solr. На текущий момент последняя версия - 5.5.0:

.. code-block:: bash

    wget http://apache-mirror.rbc.ru/pub/apache/lucene/solr/5.5.0/solr-5.5.0.zip
    unzip solr-5.5.0.zip
    cd solr-5.5.0/bin

Устанавливаем Solr в ``/opt/solr5``:

.. code-block:: bash

    unzip solr-5.5.0.zip
    cd solr-5.5.0/bin
    sudo mkdir /opt/solr5
    sudo ./install_solr_service.sh ../../solr-5.5.0.zip -i /opt/solr5

После установки сервиса он будет запускаться автоматически при старте
системы. Скрипт запуска находится в файле ``/etc/init.d/solr``.

Создадим отдельное ядро Solr для CKAN:

.. code-block:: bash

    sudo su solr
    cd /opt/solr5/solr/bin
    ./solr create -c ckan

Настроим схему:

.. code-block:: bash

    cd /var/solr/data/ckan/conf
    ln -s /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema.xml .

Удалим файл ``managed-schema``:

.. code-block:: bash

    rm managed-schema

Откройте файл ``solrconfig.xml``. Найдите элемент
``<schemaFactory class="ManagedIndexSchemaFactory">``
и закомментируйте его:

.. code-block:: xml

    <!--
    <schemaFactory class="ManagedIndexSchemaFactory">
      <bool name="mutable">true</bool>
      <str name="managedSchemaResourceName">managed-schema</str>
    </schemaFactory>
    -->

Добавьте элемент:

.. code-block:: xml

    <schemaFactory class="ClassicIndexSchemaFactory"/>

Найдите элемент ``<initParams>``, ссылающийся на
``add-unknown-fields-to-the-schema`` и закомментируйте его:

.. code-block:: xml

    <!--
    <initParams path="/update/**">
      <lst name="defaults">
        <str name="update.chain">add-unknown-fields-to-the-schema</str>
      </lst>
    </initParams>
    -->

Также закомментируйте этот элемент:

.. code-block:: xml

    <!--
    <processor class="solr.AddSchemaFieldsUpdateProcessorFactory">
      <str name="defaultFieldType">strings</str>
      <lst name="typeMapping">
        <str name="valueClass">java.lang.Boolean</str>
        <str name="fieldType">booleans</str>
      </lst>
      <lst name="typeMapping">
        <str name="valueClass">java.util.Date</str>
        <str name="fieldType">tdates</str>
      </lst>
      <lst name="typeMapping">
        <str name="valueClass">java.lang.Long</str>
        <str name="valueClass">java.lang.Integer</str>
        <str name="fieldType">tlongs</str>
      </lst>
      <lst name="typeMapping">
        <str name="valueClass">java.lang.Number</str>
        <str name="fieldType">tdoubles</str>
      </lst>
    </processor>
    -->

Перезапускаем Solr:

.. code-block:: bash

    sudo service solr restart

Отредактируем файл ``/etc/ckan/default/development.ini``, раскомментировав
соответствующую строку и указав URL Solr:

.. code-block:: bash

    solr_url = http://127.0.0.1:8983/solr/ckan


Инициализация базы данных
-------------------------

.. code-block:: bash

    sudo su -s /bin/bash - ckan
    . default/bin/activate
    cd default/src/ckan
    paster db init -c /etc/ckan/default/development.ini


Установка DataStore
-------------------

Откроём файл ``development.ini`` и в список плагинов добавим
``datastore``:

.. code-block:: bash

    ckan.plugins = stats text_view image_view recline_view datastore

В базе данных создадим пользователя ``datastore_default``, который
не будет имет прав на запись, только на чтение:

.. code-block:: bash

    sudo -u postgres createuser -S -D -R -P -l datastore_default

Создадим базу данных ``datastore_default``, владельцем которой будет
пользователь ``ckan_default``:

.. code-block:: bash

    sudo -u postgres createdb -O ckan_default datastore_default -E utf-8

Откроем файл ``development.ini``, расскомментируем и отредактируем
следующие строки, указав соответствующие пароли:

.. code-block:: bash

    ckan.datastore.write_url = postgresql://ckan_default:pass@localhost/datastore_default
    ckan.datastore.read_url = postgresql://datastore_default:pass@localhost/datastore_default

Выставим права в базе данных:

.. code-block:: bash

    cd /usr/lib/ckan
    . default/bin/activate
    cd default/src/ckan
    paster --plugin=ckan datastore set-permissions -c /etc/ckan/default/development.ini | sudo -u postgres psql --set ON_ERROR_STOP=1


Установка DataPusher
--------------------

За основу взята инструкция по установке из
`официальной документации <http://docs.ckan.org/projects/datapusher/en/latest/>`_.

.. code-block:: bash

    sudo su -s /bin/bash - ckan
    virtualenv --no-site-packages datapusher
    . datapusher/bin/activate
    mkdir datapusher/src
    cd datapusher/src
    git clone -b stable https://github.com/ckan/datapusher.git
    cd datapusher
    pip install -r requirements.txt


В файле ``development.ini`` добавим соответствующий плагин:

.. code-block:: bash

    ckan.plugins = <прочие плагины> datapusher

Также раскомментируйте следующие строки:

.. code-block:: bash

    ckan.datapusher.formats = csv xls xlsx tsv application/csv application/vnd.ms-excel application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
    ckan.datapusher.url = http://127.0.0.1:8800/


Ссылка на who.ini
-----------------

Файл ``who.ini`` (конфигурационный файл Repoze.who) должен располагаться
в той же папке, что и конфигурационный файл CKAN:

.. code-block:: bash

    ln -s /usr/lib/ckan/default/src/ckan/who.ini /etc/ckan/default/who.ini


Установка темы
--------------

В файл ``development.ini`` добавьте плагин с темой:

.. code-block:: bash

    ckan.plugins = <прочие плагины> fareast_theme

И установите саму тему:

.. code-block:: bash

    sudo su -s /bin/bash - ckan
    . default/bin/activate
    cd default/src
    git clone https://github.com/nextgis/ckanext-fareast_theme.git
    pip install -e ./ckanext-fareast_theme


Дополнительные настройки
------------------------

В файлe ``development.ini``:

.. code-block:: bash

    ckan.auth.create_user_via_web = false
    ckan.locale_default = ru
    ckan.locales_offered = en ru
    ckan.locales_filtered_out = ru_RU

    # Убрать '/ckan', если приложение монтируется в '/'
    ckan.favicon = /ckan/base/images/ckan.ico

    # На данный каталог у пользователя, под которым будет
    # запускаться CKAN, должны быть права на запись
    ckan.storage_path = /mnt/portal/ckan/default


Развёртывание CKAN
------------------

За основу взята инструкция по развёртыванию из
`официальной документации <http://docs.ckan.org/en/latest/maintaining/installing/deployment.html>`_.


Переходим в папку ``/etc/ckan/default``:

.. code-block:: bash

    sudo su -s /bin/bash - ckan
    cd /etc/ckan/default

И создаём файл ``uwsgiapp.py`` со следующим содержимым:

.. code-block:: python

    import os
    activate_this = os.path.join('/usr/lib/ckan/default/bin/activate_this.py')
    execfile(activate_this, dict(__file__=activate_this))

    from paste.deploy import loadapp
    config_filepath = os.path.join(os.path.dirname(os.path.abspath(__file__)), 'development.ini')
    from paste.script.util.logging_config import fileConfig
    fileConfig(config_filepath)
    application = loadapp('config:%s' % config_filepath)

Устанавливаем Nginx и uWSGI:

.. code-block:: bash

    sudo yum install uwsgi uwsgi-plugin-python

Переходим в папку ``/etc/uwsgi.d`` и создаём файл ``ckan.ini``
следующего содержания:

.. code-block:: bash

    [uwsgi]

    plugins = python
    lazy-apps = true

    master = true
    workers = 4
    no-orphans = true

    pidfile = /run/uwsgi/%n.pid
    socket = /run/uwsgi/%n.sock
    chmod-socket = 666

    logto = /var/log/uwsgi/%n.log
    log-date = true

    harakiri = 6000

    mount = /ckan=/etc/ckan/default/uwsgiapp.py
    manage-script-name = true

Поскольку uWSGI запущен в режиме ``Tyrant``, то необходимо изменить
владельца конфигурационного файла ``ckan.ini``:

.. code-block:: bash

    sudo chown uwsgi:uwsgi ckan.ini

Создадим папку, куда будут писаться логи:

.. code-block:: bash

    sudo mkdir /var/log/uwsgi
    sudo touch /var/log/uwsgi/ckan.log
    sudo chown uwsgi:uwsgi /var/log/uwsgi/ckan.log

Для ротации логов в папке ``/etc/logrotate.d`` создадим файл
``uwsgi`` следующего содержания:

.. code-block:: bash

    /var/log/uwsgi/*.log {
        copytruncate
        daily
        rotate 5
        compress
        delaycompress
        missingok
        notifempty
    }

Запускаем uWSGI:

.. code-block:: bash

    sudo systemctl start uwsgi
    sudo systemctl enable uwsgi

В папке ``/etc/nginx/conf.d`` создадим файл ``ckan.conf``
следующего содержания:

.. code-block:: bash

    uwsgi_cache_path /var/lib/nginx/cache levels=1:2 keys_zone=ckan:30m max_size=250m;

    server {
          listen               80;
          server_name          82.162.194.216;
          client_max_body_size 100M;

          location /ckan {
            uwsgi_read_timeout 600s;
            uwsgi_send_timeout 600s;

            include            uwsgi_params;
            uwsgi_pass         unix:/run/uwsgi/ckan.sock;

            # Cache stuff
            uwsgi_cache         ckan;
            uwsgi_cache_methods GET HEAD;
            uwsgi_cache_bypass  $cookie_auth_tkt;
            uwsgi_no_cache      $cookie_auth_tkt;
            uwsgi_cache_valid   30m;
            uwsgi_cache_key     $host$scheme$proxy_host$request_uri;
        }
    }

Создадим папку под кэш:

.. code-block:: bash

    sudo mkdir /var/lib/nginx/cache
    sudo chown nginx:nginx /var/lib/nginx/cache

Запускаем Nginx:

.. code-block:: bash

    sudo systemctl start nginx
    sudo systemctl enable nginx

.. warning::
   Если приложение при попытке его открыть возвращает ``502 Bad Gateway``,
   а в логах Nginx
   ``connect() to unix:/run/uwsgi/ckan.sock failed (13: Permission denied)``,
   то причина в SELinux - либо настройте его, либо отключите.


Развёртывание DataPusher
------------------------

.. code-block:: bash

    sudo cp /usr/lib/ckan/datapusher/src/datapusher/deployment/datapusher.wsgi /etc/ckan/
    sudo cp /usr/lib/ckan/datapusher/src/datapusher/deployment/datapusher_settings.py /etc/ckan/
    sudo chown ckan:ckan /etc/ckan/datapusher.wsgi
    sudo chown ckan:ckan /etc/ckan/datapusher_settings.py

В файле ``datapusher_settings.py`` допишите строку:

.. code-block:: bash

    MAX_CONTENT_LENGTH = 1073741824

Подготавливаем uWSGI:

.. code-block:: bash

    sudo touch /etc/uwsgi.d/datapusher.ini
    sudo chown uwsgi:uwsgi /etc/uwsgi.d/datapusher.ini
    sudo touch /var/log/uwsgi/datapusher.log
    sudo chown uwsgi:uwsgi /var/log/uwsgi/datapusher.log

В файл ``datapusher.ini`` помещаем следующее содержимое:

.. code-block:: bash

    [uwsgi]

    plugins = python

    master = true
    workers = 1
    threads = 5
    no-orphans = true

    pidfile = /run/uwsgi/%n.pid
    http-socket = 127.0.0.1:8800

    logto = /var/log/uwsgi/%n.log
    log-date = true

    harakiri = 6000

    wsgi-file = /etc/ckan/datapusher.wsgi


Установка расширения ckanext-geoview
------------------------------------

Данное расширение предоставляет набор плагинов, позволяющих осуществлять
предпросмотр опубликованных ресурсов прямо в браузере.

Оригинальная версия данного расширения не содержит некоторых нужных
нам возможностей, поэтому была выполнена его доработка. В связи с этим
устанавливать его мы будем из собственного репозитория:

.. code-block:: bash

    sudo su -s /bin/bash - ckan
    . default/bin/activate
    cd default/src
    git clone https://github.com/nextgis/ckanext-geoview.git
    pip install -e ./ckanext-geoview

Изменим стандартную подложку, для этого в файл ``development.ini`` добавим
следующие строки:

.. code-block:: bash

    # Settings for ckanext-geoview extension
    ckanext.spatial.common_map.type = custom
    ckanext.spatial.common_map.custom.url = http://tiles.maps.sputnik.ru/{z}/{x}/{y}.png
    ckanext.spatial.common_map.custom.name = Карта Спутник
    ckanext.spatial.common_map.attribution = © <a href="http://sputnik.ru">Спутник</a> | © <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>


Исправление текущего кода CKAN
------------------------------

В последующих релизах эти ошибки будут устранены, но пока они не выпущены,
то приходится модифицировать код самостоятельно. Это необходимо сделать
в соответствии со следующими коммитами: `#9cf0369 <https://github.com/ckan/ckan/commit/9cf0369>`_,
`#a63de1b <https://github.com/ckan/ckan/commit/a63de1b>`_,
`#a76da0c <https://github.com/ckan/ckan/commit/a76da0c>`_.


Исправления текущего кода DataStore и DataPusher
------------------------------------------------

Откроем файл ``commas.py``:

.. code-block:: bash

    cd /usr/lib/ckan/datapusher/lib/python2.7/site-packages/messytables
    nano commas.py

И отредактируем следующую строку. Вместо:

.. code-block:: bash

    # Fix the maximum field size to something a little larger
    csv.field_size_limit(256000)

должно быть:

.. code-block:: bash

    # Fix the maximum field size to something a little larger
    csv.field_size_limit(100 * 1024 * 1024)

На данный момент в CKAN DataStore нет возможности исключить поле из
индекса на основе его имени,
см. `#2837 <https://github.com/ckan/ckan/issues/2837>`_, поэтому
(как временное решение) отключим текстовые поля для индексации.
В противном случае DataStore пытается построить индекс для поля
с WKT геометрией (если такое поле есть в CSV) и падает.
Откроем файл ``helpers.py``:

.. code-block:: bash

    cd /usr/lib/ckan/default/src/ckan/ckanext/datastore
    nano helpers.py

И отредактируем следующий фрагмент. Вместо:

.. code-block:: bash

    def should_fts_index_field_type(field_type):
        return field_type.lower() in ['tsvector', 'text', 'number']

должно быть:

.. code-block:: bash

    def should_fts_index_field_type(field_type):
        return field_type.lower() in ['tsvector', 'number']

Сделаем ещё одно исправление для отключения включения WKT в
полнотекстовый поиск. Откроем файл ``db.py``:

.. code-block:: bash

    cd /usr/lib/ckan/default/src/ckan/ckanext/datastore
    nano db.py

И отредактируем функцию ``_to_full_text``:

.. code-block:: python

    def _to_full_text(fields, record):
        full_text = []
        ft_types = ['int8', 'int4', 'int2', 'float4', 'float8', 'date', 'time',
                    'timetz', 'timestamp', 'numeric', 'text']
        for field in fields:
            value = record.get(field['id'])
            if not value:
                continue

            if 'POINT' in unicode(value) or \
               'LINESTRING' in unicode(value) or \
               'POLYGON' in unicode(value):
                    continue

            if field['type'].lower() in ft_types and unicode(value):
                full_text.append(unicode(value))
            else:
                full_text.extend(json_get_values(value))
        return ' '.join(set(full_text))

Увеличим таймаут в файле ``jobs.py``:

.. code-block:: bash

    cd /usr/lib/ckan/datapusher/src/datapusher/datapusher
    nano jobs.py

Вместо ``DOWNLOAD_TIMEOUT = 30`` выставьте ``DOWNLOAD_TIMEOUT = 5 * 60``.

.. warning::
   Проблемы, с которыми мы столкнулись при загрузке CSV в DataStore.
   В некоторых полях был текст ``NULL``, но по первой записи DataStore
   определял, что это ``numeric``, и когда доходил до ``NULL`` - падал.
   В некоторых полях в качестве разделителя разрядов была ``,``.
   Исправляется путём редактирования соответствующих ресурсов NextGIS Web.

При загрузке данных всегда сверяйте количество загруженных объектов
и фактическое количество объектов в слое. Количество объектов в слое
можно получить по API (подставьте свой идентификатор ресурса):

.. code-block:: bash

    http://82.162.194.216/ngw/api/resource/167/feature_count

Открытие портов
---------------

Откроем ``80`` порт:

.. code-block:: bash

    sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
    sudo firewall-cmd --reload

Если с локального хоста CKAN недоступен по своему публичному адресу,
то это может быть исправлено, например, так:

.. code-block:: bash

    sudo firewall-cmd --direct --add-rule ipv4 nat OUTPUT 0 -d 82.162.194.216 -p tcp --dport 80 -j DNAT --to-destination 127.0.0.1:80 --permanent
    sudo firewall-cmd --reload


Backup и Restore
----------------

Данные процедуры описаны в разделе
`db: Manage databases <http://docs.ckan.org/en/latest/maintaining/paster.html#db-manage-databases>`_
официальной документации.

Если CKAN и NextGIS Web были развёрнуты на одной машине, то при переносе
этой связки на другой адрес, необходимо изменить адреса ресурсов
NextGIS Web на новые. Проще всего это сделать, отредактировав файл
с бэкапом CKAN до момента его восставновления, например:

.. code-block:: bash

    sed -i 's:78.46.100.76/opendata_ngw:82.162.194.216/ngw:g' ckan.pg_dump

Если вы используете DataStore, то помимо переноса базы самого CKAN,
необходимо переносить и базу данных DataStore, ``datastore_default``
в нашем случае.

.. warning::
   Если после восстановления данных из архива отображается, что
   число наборов данных равно ``0``, то поможет
   `переиндексация <http://docs.ckan.org/en/latest/maintaining/paster.html#search-index-rebuild-search-index>`_.


Дополнительно
-------------

Конфигурационный файл uWSGI для NextGIS Web:


.. code-block:: bash

    [uwsgi]

    plugins = python
    lazy-apps = true

    master = true
    workers = 4
    no-orphans = true

    pidfile = /run/uwsgi/%n.pid
    socket = /run/uwsgi/%n.sock
    chmod-socket = 666

    logto = /var/log/uwsgi/%n.log
    log-date = true

    limit-post = 7516192768

    harakiri = 6000
    socket-timeout = 6000

    env = PASTE_CONFIG=/opt/ngw/development.ini
    env = LANG=ru_RU.UTF-8

    home = /opt/ngw/env
    mount = /ngw=/opt/ngw/nextgisweb/nextgisweb/uwsgiapp.py
    manage-script-name = true


Конфигурационный файл Nginx:

.. code-block:: bash

    uwsgi_cache_path /var/lib/nginx/cache levels=1:2 keys_zone=ckan:30m max_size=250m;
    uwsgi_cache_path /mnt/portal/ngw/cache levels=1:2 keys_zone=ngw:30m max_size=10g inactive=7d;

    server {
          listen                      80;
          server_name                 82.162.194.216;
          client_max_body_size        6G;
          large_client_header_buffers 8 32k;

          location /ckan {
            uwsgi_read_timeout 600s;
            uwsgi_send_timeout 600s;

            include            uwsgi_params;
            uwsgi_pass         unix:/run/uwsgi/ckan.sock;

            # Cache stuff
            uwsgi_cache         ckan;
            uwsgi_cache_methods GET HEAD;
            uwsgi_cache_bypass  $cookie_auth_tkt;
            uwsgi_no_cache      $cookie_auth_tkt;
            uwsgi_cache_valid   30m;
            uwsgi_cache_key     $host$scheme$proxy_host$request_uri;
        }

        location /ngw {
            uwsgi_read_timeout 600s;
            uwsgi_send_timeout 600s;

            include            uwsgi_params;
            uwsgi_pass         unix:/run/uwsgi/ngw.sock;

            # Cache stuff
            uwsgi_cache          ngw;
            uwsgi_cache_methods  GET HEAD;
            uwsgi_cache_bypass   $cookie_tkt;
            uwsgi_no_cache       $cookie_tkt;
            uwsgi_cache_valid    7d;
            uwsgi_ignore_headers Expires Cache-Control Set-Cookie;
            uwsgi_cache_key      $host$scheme$proxy_host$request_uri;
            add_header           X-uWSGI-Cache $upstream_cache_status;
        }

        location /opendata_map {
            index index.html index.htm;
            root /var/www;
        }
    }


Пример удаления записей из кэша Nginx по маске:

.. code-block:: bash

    grep -lr 'ngw/api/resource/604*' /mnt/portal/ngw/cache/* | xargs rm -rf
