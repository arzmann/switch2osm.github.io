---
layout: docs
title: Начинаем работать с Leaflet
permalink: /using-tiles/getting-started-with-leaflet/
---

# Введение

[Leaflet](http://leafletjs.com/) - это легкая библиотека на JavaScript, позволяющая встраивать карты в web. Она распространяется под лицензией BSD для программного обеспечения с открытым исходным кодом, что позволяет ее использовать абсолютно на любом сайте без каких-либо дополнительных юридических ограничений. Ее исходный текст доступен на [GitHub](http://github.com/Leaflet/Leaflet).

На этом сайте мы ограничимся лишь небольшим примером, демонстрирующим возможности библиотеки, а также приведем ссылки на официальные [руководства](http://leafletjs.com/examples.html) и [документацию](http://leafletjs.com/reference.html) по использованию этой замечательной библиотеки.

# Начало работы
Создайте файл `leaflet.html`, скопируйте в него нижеприведенный код, сохраните и откройте в браузере:

{% highlight html %}
{% include leaflet.html %}
{% endhighlight %}

# Дополнительные ссылки
Если вы хотите:

* использовать другой фон → Leaflet изначально поддерживает [TMS](https://en.wikipedia.org/wiki/Tile_Map_Service) и [WMS](https://ru.wikipedia.org/wiki/Web_Map_Service). Посмотрите [здесь](http://leafletjs.com/reference.html#tilelayer), какие еще опции есть у Leaflet.
* обозначить все места, где располагается ваша компания? → Сохраните их координаты в файле [GeoJSON](http://geojson.org/) и они [появятся](http://leafletjs.com/examples/geojson.html) на карте.
* использовать другую картографическую проекцию? → Используйте плагин [Proj4Leaflet](https://github.com/kartena/Proj4Leaflet).
