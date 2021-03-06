.. sectionauthor:: Дмитрий Барышников <dmitry.baryshnikov@nextgis.ru>

.. _ngogportal_user:

Инструкция по использованию портала
==============================================

Данный портал обеспечивает процессы получения, подготовки, проверки и публикации 
открытых геопространственных данных, систематизации открытых данных, модификации 
свойств открытых наборов, а также отмены публикации данных. Вы можете просмотреть 
их в браузере или скачивать в виде файлов. Данные доступны в машиночитаемом формате, 
поддерживаемые современным программным обеспечением: NextGIS QGIS, ArcGIS, библиотеками GDAL.

* Если вы хотите ознакомиться с данными - просмотрите их в браузере по ссылке.
* Если вы хотите сохранить данные себе - скачайте файлы.
* Если вы хотите работать с данными в Open Office Calc / Microsoft Excel - скачайте 
  файлы в формате :term:`CSV`.
* Если вы хотите заняться пространственным анализом данных - скачайте файлы в :term:`GeoJSON`или :term:`ESRI Shapefile` .
* Если вы хотите разработать сервис, который будет показывать данные с портала на 
  вашей карте - передавайте информацию с помощью протоколов :term:`WMS`, :term:`WFS` или, 
  если вам нужна высокая скорость - скачайте файлы.


Просмотр геоданных на портале
--------------------------------------

Вам потребуется:

1. Компьютер с доступом к cети Интернет.
2. Адрес портала - http://opendata25.primorsky.ru/ckan .
3. Манипулятор "Мышь" или аналог.

1. Откройте в веб-браузере ссылку http://opendata25.primorsky.ru/ckan. Вы увидите 
главную страницу портала (см. :numref:`ogportalCKANUserIndex`). Нажмите на ссылку 
"Пакеты данных" или "Наборы данных".

.. figure:: _static/ogportalCKANUserIndex.png
   :name: ogportalCKANUserIndex
   :align: center
   :width: 15cm

   Главная страница портала.

2. Вы увидите список массивов данных. Выберите название нужного вам массива данных (см. :numref:`ogportalCKANUserPacketsPage`): 

.. figure:: _static/ogportalCKANUserPacketsPage.png
   :name: ogportalCKANUserPacketsPage
   :align: center
   :width: 15cm

   Список пакетов данных.

3. В массиве данных находятся ресурсы. Вы увидите их список. Каждый набор данных 
представлен в нескольких форматах - эти форматы обозначаются значком слева: :term:`JSON`, 
GeoJSON, Data, CSV.  Выберите нужный вам набор данных, (см. :numref:`ogportalCKANUserResourcesPage`) 
и нажмите на GeoJSON (см. :numref:`ogportalCKANGeoJSONIcon`). Этот формат наиболее 
удобно отображается в браузере. 


.. figure:: _static/ogportalCKANUserResourcesPage.png
   :name: ogportalCKANUserResourcesPage
   :align: center
   :width: 15cm

   Список ресурсов в массиве данных.

.. figure:: _static/ogportalCKANGeoJSONIcon.png
   :name: ogportalCKANGeoJSONIcon
   :align: center

   Иконка GeoJSON (перенести в текст).

6. На экране появится карта (см. :numref:`ogportalCKANUserGeojsonWebmap`). Синим 
цветом на ней обозначены данные набора. 

.. figure:: _static/ogportalCKANUserGeojsonWebmap.png
   :name: ogportalCKANUserGeojsonWebmap
   :align: center
   :width: 15cm

   Пример карты с наложенными данными.

7. Если необходимости просмотреть атрибуты объекта, нажмите мышкой на объект, и 
на экране появится окно с таблицей атрибутов объекта(см. :numref:`ogportalCKANUserGeojsonWebmapIdentify`). 
Этот процесс называется идентификацией.

.. figure:: _static/ogportalCKANUserGeojsonWebmapIdentify.png
   :name: ogportalCKANUserGeojsonWebmapIdentify
   :align: center
   :width: 15cm

   Идентификация.

Просмотр данных в табличном виде
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 

1. Откройте данные в формате CSV:

.. figure:: _static/ogportalCKANCSVIcon.png
   :name: ogportalCKANCSVIcon
   :align: center

   Иконка CSV (перенести в текст).

2. На экране появится таблица данных (см. :numref:`ogportalCKANUserDataTable`):

.. figure:: _static/ogportalCKANUserDataTable.png
   :name: ogportalCKANUserDataTable
   :align: center
   :width: 15cm

   Просмотр данных в таблице.

Получение данных в машиночитаемом формате
-----------------------------------------------------------------

Выберите нужный вам набор данных и нажмите на значок нужного формата. Если у вас 
нет специальных требований, выбирайте формат GeoJSON, он открывается современными 
программами и не вносит ограничения на данные. 
На странице будет ссылка на скачивание файла.

.. figure:: _static/ogportalCKANDownloadGeoJSONLink.png
   :name: ogportalCKANDownloadGeoJSONLink
   :align: center
   :width: 15cm

   Ссылка на скачивание карты.

Как открыть данные в машиночитаемом формате на компьютере?
--------------------------------------------------------------------

Рассмотрим на примере программы NextGIS QGIS. Это свободное программное обеспечение, 
распространяемое бесплатно. Точно таким же образом можно работать в программе QGIS 
на других операционных системах.

1. Сохраните файл в формате GeoJSON.
2. Откройте программу NextGIS QGIS.
3. Нажмите :menuselection:`Слой --> Добавить слой --> Добавить векторный слой`. (см. :numref:`ogportalQGISOpenGeoJSON1`), (см. :numref:`ogportalQGISOpenGeoJSON2`). 
В диалоге выберите скачанный вами файл в формате GeoJSON (см. :numref:`ogportalQGISOpenGeoJSON3`).

.. figure:: _static/LREGQGISOpenShape1.png
   :name: ogportalQGISOpenGeoJSON1
   :align: center
   :width: 15cm

   Добавление векторного слоя. 

.. figure:: _static/LREGQGISOpenShape2.png
   :name: ogportalQGISOpenGeoJSON2
   :align: center
   :width: 15cm

   Добавление векторного слоя.
 
.. figure:: _static/ogportalQGISOpenGeoJSON3.png
   :name: ogportalQGISOpenGeoJSON3
   :align: center
   :width: 15cm

   Интерфейс QGIS. 

5. Выделите слой в списке слоёв и откройте таблицу атрибутов, выбрав в меню :menuselection:`Слой` ---> `Таблица атрибутов` (см. :numref:`ogportalQGISOpenGeoJSON4`).

.. figure:: _static/ogportalQGISOpenGeoJSON4.png
   :name: ogportalQGISOpenGeoJSON4
   :align: center
   :width: 15cm

   Слой данных и таблица атрибутов.

Ссылка на QMS?
Таким образом геоданные можно открывать в программе для работы на компьютере.

Как открыть данные, если ПО не поддерживает GeoJSON?
---------------------------------------------------------------------

Скачайте данные в формате ESRI Shapefile (значок DATA). В этом формате данные распространяются 
в zip-архиве, который нужно распаковать и открыть в вашей программе файл .shp. 

.. figure:: _static/ogportalCKANSHPIcon.png
   :name: ogportalCKANSHPIcon
   :align: center

   Нажмите на эту ссылку.

.. figure:: _static/ogportalSHPZip.png
   :name: ogportalSHPZip
   :align: center
   :width: 15cm

   Содержимое архива.

Как открыть данные в Calc или Excel на компьютере?
---------------------------------------------------------------------

1. Скачайте данные в формате CSV:

.. figure:: _static/ogportalCKANCSVIcon.png
   :name: ogportalCKANCSVIcon1
   :align: center

   Иконка CSV (перенести в текст).

2. Откройте файл в редакторе электронных таблиц. Укажите разделитель - запятая, 
и кодировку - Юникод (UTF-8). 

.. figure:: _static/ogportalCalcOpenCSV.png
   :name: ogportalCalcOpenCSV
   :align: center
   :width: 15cm

   Открытие CSV в Open Office Calc.

.. figure:: _static/ogportalCalc.png
   :name: ogportalCalc
   :align: center
   :width: 15cm

   Пример таблицы в Open Office Calc.






