---
layout: docs
title: Начинаем работать с OpenLayers
permalink: /using-tiles/getting-started-with-openlayers/
---

## Введение

[OpenLayers](http://openlayers.org/) - это многофункциональная библиотека на JavaScript для встраивания карт. Она распространяется под лицензией BSD для программного обеспечения с открытым исходным кодом, что позволяет ее использовать абсолютно на любом сайте без каких-либо дополнительных юридических ограничений. Ее исходный текст доступен на [GitHub](https://github.com/openlayers/ol3/).

На этом сайте мы ограничимся лишь небольшим примером, демонстрирующим возможности библиотеки, а также приведем ссылки на официальные [руководства](http://openlayers.org/en/latest/examples/) и [API](http://openlayers.org/en/latest/apidoc/) для использования этой замечательной библиотеки.

## Начало работы

Создайте файл `openlayers.html`, скопируйте в него нижеприведенный код, сохраните и откройте в браузере:

{% highlight html %}
{% include openlayers.html %}
{% endhighlight %}

# Дополнительные ссылки
Если вы хотите:

* использовать другой фон → OpenLayers изначально поддерживает [TMS](https://en.wikipedia.org/wiki/Tile_Map_Service) и [WMS](https://ru.wikipedia.org/wiki/Web_Map_Service). Посмотрите [здесь](http://openlayers.org/en/latest/examples/) и [здесь](http://openlayers.org/en/latest/apidoc/), чтобы узнать какие есть еще опции.
* обозначить все места, где располагается ваша компания? → Сохраните их координаты в файле [GeoJSON](http://geojson.org/) и они [появятся](http://openlayers.org/en/latest/examples/select-features.html) на карте.
* использовать другую картографическую проекцию? → OpenLayers поддерживает все проекции [proj4js](http://proj4js.org/), если вы включили библиотеку proj4js JavaScript. Кроме того, поддерживается растровая [репроекция](http://openlayers.org/en/latest/examples/reprojection-by-code.html) на стороне клиента, так что можно использовать тайлы OpenStreetMap в локальной проекции.
