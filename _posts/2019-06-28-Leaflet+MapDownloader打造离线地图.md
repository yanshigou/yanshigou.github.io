---
title: "Leaflet+MapDownloader打造离线地图"
date: 2019-06-28 17:47
author: dzt
subtitle: 记录一下
tags:
  - 离线地图
  - Leaflet+MapDownloader
---



先简单记录一下 

有空再填坑



下载MapDownloader地图下载器



可以mysql或本地磁盘切片

配置MapDownloader.exe.config

```html
<body>
<div id="map"></div>
<script type="text/javascript">

    var amapNormalUrl = 'http://192.168.31.54:8844/788865972/{z}/{x}/{y}';
    console.log(amapNormalUrl);
    var amapNormalLayer = new L.TileLayer(amapNormalUrl, {
        minZoom: 1,
        maxZoom: 18,
        attribution: '高德普通地图ͼ'
    });
    var mapCenter = new L.LatLng(29.5116, 106.52386);

    console.log(amapNormalLayer);
    var map = new L.Map('map', {
        center: mapCenter,
        zoom: 13,
        minZoom: 1,
        maxZoom: 18,
        layers: [amapNormalLayer]
    });

    $.get('/media/a.txt', function (csv) {

            {#$.get('', function(csv) {#}
            var data = csv.split('\n');
            $.each(data, function (n, value) {
                // 索引 值
                {#alert(n + ' ' + value);#}
                console.log(value);

                var la = parseFloat(value.split(',')[0]);
                var ln = parseFloat(value.split(',')[1]);
                var address = parseFloat(value.split(',')[1]);
                console.log(la, ln, address);
                var marker = L.marker([ln, la]).addTo(map);

            });

        }
    );
</script>
</body>
```