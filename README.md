# testLeafletMap
利用leaflet.js根据从josm下载下来的数据源绘制地图
步骤：
1、下载josm软件，地址：https://josm.openstreetmap.de/  我这里下载的是josm-tested.jar这个jar包，
需要电脑安装jre(java执行环境)，直接去官网下载jdk1.6以上，网址：http://www.oracle.com/technetwork/java/javase/downloads/index.html，选择合适的版本
2、打开josm，点击下载按钮，会弹出一个界面，一张地图，拖动鼠标左键可以选择要下载的区域，鼠标右键可以移动地图，滚轮可以进行放大缩小，
这里注意：选择的区域不要太大，会被服务器拒绝(ps:这里选择区域，我还没有找到范围精确限定，就是指定具体的最大经纬度和最小经纬度来选择区域
，就随意选择了一个区域)
3、点击下载后，就会出现你刚刚选择的那个区域的局部地图信息，保存为json或者geojson类型的文件，
geojson文件说明：
GeoJSON是一种对各种地理数据结构进行编码的格式，基于Javascript对象表示法的地理空间信息数据交换格式。GeoJSON对象可以表示几何、特征或者特征集合。
GeoJSON支持下面几何类型：点、线、面、多点、多线、多面和几何集合。GeoJSON里的特征包含一个几何对象和其他属性，特征集合表示一系列特征
    我的理解就是一种描述地理信息的文件，是json格式类型，所以我们采用js来读取这个文件
4、下载leaflet.js地址：http://leafletjs.com/，引入leaflet-src.js和leaflet.css到我们的html文件中，同时要一如jquery的js文件
    <link rel="stylesheet" href="leaflet/leaflet.css" />
    <script src="leaflet/leaflet-src.js"></script>
    <script src="js/jquery-1.9.1.min.js"></script>
5、需要一小段js代码来读取文件，调用我们的leaflet.js的方法进行绘制，下面读取文件的代码以及绘制地图的方法
$.getJSON('./data/home.geojson',function(data){

        mapData = data;
        // console.log(d);
        //创建GeoJSON 图层
        $.each(mapData.features,function(i,item) {

            if(!item.geometry.type) {
                return true;
            }
            var geo = L.geoJSON(item,{
                style:style,
                onEachFeature:onEachFeature
            }).addTo(map);
            console.log(i+"  "+geo);
            map.on("click",function(e) {
                popup
                    .setLatLng(e.latlng)
                    .setContent("lan_lng: " + e.latlng.toString())
                    .openOn(map);
            })
        });
    });
这里做简要说明：
home.geojson：这是我下载下来的数据文件，这是个描述地理信息的json格式的文件，大概结构如下所示：
{ "type": "FeatureCollection",
  "features": [
    { "type": "Feature",
      "geometry": {"type": "Point", "coordinates": [102.0, 0.5]},
      "properties": {"prop0": "value0"}
      },
    { "type": "Feature",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [102.0, 0.0], [103.0, 1.0], [104.0, 0.0], [105.0, 1.0]
          ]
        },
      "properties": {
        "prop0": "value0",
        "prop1": 0.0
        }
      },
    { "type": "Feature",
       "geometry": {
         "type": "Polygon",
         "coordinates": [
           [ [100.0, 0.0], [101.0, 0.0], [101.0, 1.0],
             [100.0, 1.0], [100.0, 0.0] ]
           ]
       },
       "properties": {
         "prop0": "value0",
         "prop1": {"this": "that"}
         }
       }
     ]
   }
重点关注features这个字段他表示地图上某一个类型的特征，是一个数据，因为地图上有很多元素
每一个feature描述某一个类型的特征，类型可以通过feomotry这个字段来查看，lineString、point、ploygon分别表示线，点，多边形等等
还有很多种类型，例如：多点，多线，多面等，
coordinates字段是描述坐标位置的，即经纬度
properties：属性的描述，例如这个地区的名字，类型等，不过下载下来的数据有些没有这些属性，或者属性很少，不知道为什么
6、绘制地图
    var map = L.map('map',{
        crs:L.CRS.Simple,
        center:[121.4646,31.2192],
        zoom:1,
        minZoom:0,
        maxZoom:18
    });    
L.map()，创建一个map对象，需要传递一个容器div的id，他会在指定id的div中创建map，后面的参数是指定，坐标，中心点，缩放等级，最小缩放等级，最大缩放等级，
他还有很多参数，可以参考官网
接下来是刚刚上面那段读取文件的代码
var geo = L.geoJSON(item,{
                style:style,
                onEachFeature:onEachFeature
            }).addTo(map);
    geoJSON方法用来创建geiJSON图层，item表示geojson格式的对象
    style指定样式，onEachFeature表示在初始化的时候执行一次这个方法，一般用来绑定监听事件
        在绘制的时候遇到一些小问题，geoJSON方法接受一个geojson格式的对象，结果在绘制的时候一直报错，说geojson格式错误，
        后来debug了一下leaflet.js的源码，发现是在创建地图元素的时候，他会根据geometry.type来创建对应的地图元素，结果我下载的部分数据没有type这个类型
        导致创建地图的时候报错了，所以采用循环的方式来进行绘制，再循环的时候判断一下是否包含这个属性，绘制成功
        
  浏览器打开testMap.html，用鼠标滚轮放大地图，可以看到和我们之前用josm下载的那个局部地图轮廓一致
    
