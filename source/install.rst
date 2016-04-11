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

.. code:: bash

    sudo yum install epel-release
    sudo yum install python-pip python-virtualenv python-devel
    sudo yum install git gcc-c++ java-1.8.0-openjdk unzip lsof
    sudo yum install postgresql postgresql-server postgresql-libs postgresql-contrib postgresql-devel

.. warning::
   Если на сервере, куда устанавливается CKAN уже установлен NextGIS Web,
   то пакеты для PostgreSQL ставить не нужно. В этом случае PostgreSQL
   уже должен быть установлен и настроен.


Установка CKAN
--------------

Создаём пользователя CKAN:

.. code:: bash

    sudo useradd -m -s /sbin/nologin -d /usr/lib/ckan -c "CKAN User" ckan

Домашняя директория пользователя CKAN остальным пользователям системы
доступна только на чтение:

.. code:: bash

    sudo chmod 755 /usr/lib/ckan

Переключаемся на только что созданного пользователя:

.. code:: bash

    sudo su -s /bin/bash - ckan

Устанавливаем виртуальное окружение ``default``:

.. code:: bash

    virtualenv --no-site-packages default

И активируем его:

.. code:: bash

    . default/bin/activate

Скачиваем и устанавливаем последнюю версию CKAN. На момент написани
данной инструкции это версия CKAN 2.5.2. Если вы устанавливаете
другую версию, то измените URL в следующей команде:

.. code:: bash

    pip install -e git+https://github.com/okfn/ckan.git@ckan-2.5.2#egg=ckan

Устанавливаем пакеты Python от которых зависит CKAN:

.. code:: bash

    pip install -r default/src/ckan/requirements.txt


Настройка PostgreSQL
--------------------

Инициализируем базу данных и включаем автоматический запуск
PostgreSQL при старте системы:

.. code:: bash

    sudo postgresql-setup initdb
    systemctl start postgresql
    systemctl enable postgresql

.. warning::
   В ходе работы с CKAN размер базы будет увеличиваться, поэтому
   в случае необходимости кластер базы данных должен быть вынесен
   в директорию с достаточным объёмом дискового пространства.

Создаём пользователя базы данных ``ckan_default`` и
саму базу также названную ``ckan_default``:

.. code:: bash

    sudo -u postgres createuser -S -D -R -P ckan_default
    sudo -u postgres createdb -O ckan_default ckan_default -E utf-8

Отредактируем параметры аутентификации в соответствующем файле:

.. code:: bash

    sudo nano /var/lib/pgsql/data/pg_hba.conf

Отредактируем его таким образом, чтобы в нём присутствовали следующие
строки (исправим метод аутентификации на ``md5``, если указан иной):

.. code:: bash

    # IPv4 local connections:
    host    all             all             127.0.0.1/32            md5
    # IPv6 local connections:
    host    all             all             ::1/128                 md5

.. warning::
   Предполагается, что CKAN и PostgreSQL установлены на одном хосте.
   Если это не так, то потребуется дополнительная настройка PostgreSQL.


Создание конфигурационного файла CKAN
-------------------------------------

.. code:: bash

    sudo mkdir -p /etc/ckan/default
    sudo chown -R ckan /etc/ckan/

Переключаемся на пользователя CKAN и создаём конфигурационный файл:

.. code:: bash

    su -s /bin/bash - ckan
    . default/bin/activate
    cd /usr/lib/ckan/default/src/ckan
    paster make-config ckan /etc/ckan/default/development.ini

Отредактируйте файл ``development.ini``, указав пароль пользователя
``ckan_default`` и URL сайта, по которому будет доступен CKAN:

.. code:: bash

    sqlalchemy.url = postgresql://ckan_default:pass@localhost/ckan_default
    ckan.site_url = http://domain_name


Установка Solr5
---------------

Скачиваем Solr. На текущий момент последняя версия - 5.5.0:

.. code:: bash

    wget http://apache-mirror.rbc.ru/pub/apache/lucene/solr/5.5.0/solr-5.5.0.zip
    unzip solr-5.5.0.zip
    cd solr-5.5.0/bin

Устанавливаем Solr в ``/opt/solr5``:

.. code:: bash

    unzip solr-5.5.0.zip
    cd solr-5.5.0/bin
    sudo mkdir /opt/solr5
    sudo ./install_solr_service.sh ../../solr-5.5.0.zip -i /opt/solr5

После установки сервиса он будет запускаться автоматически при старте
системы. Скрипт запуска находится в файле ``/etc/init.d/solr``.

Создадим отдельное ядро Solr для CKAN:

.. code:: bash

    sudo su solr
    cd /opt/solr5/solr/bin
    ./solr create -c ckan

.. TODO: Настройка схемы CKAN.

Отредактируем файл ``/etc/ckan/default/development.ini``, раскомментировав
соответствующую строку и указав URL Solr:

.. code:: bash

    solr_url = http://127.0.0.1:8983/solr/ckan


Инициализация базы данных
-------------------------

.. code:: bash

    sudo su -s /bin/bash - ckan
    . default/bin/activate
    cd default/src/ckan
    paster db init -c /etc/ckan/default/development.ini


Установка DataStore
-------------------


Ссылка на ``who.ini``
---------------------


Установка темы
--------------


Открытие портов
---------------


Развёртывание
-------------

