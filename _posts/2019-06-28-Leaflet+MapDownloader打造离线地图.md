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

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/leaflet.png)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/leaflet2.png)



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

    $.get('/media/all_address_info.txt', function (csv) {

            var data = csv.split('\n');
            $.each(data, function (n, value) {
                // 索引 值
                console.log(value);
                if (value) {
                    //没有金额，跳过：
                    var la = parseFloat(value.split(',')[0]);
                    var ln = parseFloat(value.split(',')[1]);
                    var address = value.split(',')[2];
                    var address_id = value.split(',')[3];
                    console.log(la, ln, address);
                    var icon = L.icon({
                        iconUrl: "/media/image/wt_sxt.png",
                        iconSize: [40, 40],
                        iconAnchor: [30, 30]
                    });
                    var marker = L.marker([ln, la], {icon}).addTo(map);
                    marker.bindPopup('<a href="/devices/deviceModify/'+address_id+'/'+'">'+address+'</a>').openPopup();

                }

            });

        }
    );


</script>
</body>
```

- [Marker](https://leafletjs.com/reference-1.0.3.html#marker) 用于在地图上添加可点击和移动的图标
- [Popup](https://leafletjs.com/reference-1.0.3.html#popup) 用于在地图的某个点打开弹出窗口
- [Tooltip](https://leafletjs.com/reference-1.0.3.html#tooltip) 用于在地图的某个点显示少量文字