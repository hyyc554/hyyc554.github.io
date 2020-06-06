---
title: Python一行代码处理地理围栏
date: 2019-05-05 18:42:08

tags:
  - Python
---

最近在工作中遇到了这个一个需求，用户设定地理围栏，后台获取到实时位置信息后通过与围栏比较，判断是否越界等。

这个过程需要用到数据协议为GEOjson，通过查阅资料后，发现python的shapely库可以非常简单的解决这个问题,接下来演示一下我处理这个问题的过程。

<!--more-->

## 测试数据：

通过http://geojson.io/来获得测试数据，如下图，在地图上绘制了一个多边形设为地理围栏，分别取了围栏内外两个点来进行测试。

![FUwEh4.png](https://s1.ax1x.com/2018/12/14/FUwEh4.png)

得到GEOjson数据如下：

``````json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {},
      "geometry": {
        "type": "Polygon",
        "coordinates": [
          [
            [
              114.3458104133606,
              30.476167529462785
            ],
            [
              114.34512376785278,
              30.475575748963195
            ],
            [
              114.34576749801636,
              30.474540124433936
            ],
            [
              114.3467652797699,
              30.475363076967565
            ],
            [
              114.34693694114685,
              30.476102803645833
            ],
            [
              114.3458104133606,
              30.476167529462785
            ]
          ]
        ]
      }
    },
    {
      "type": "Feature",
      "properties": {},
      "geometry": {
        "type": "Point",
        "coordinates": [
          114.34605717658997,
          30.475584995561178
        ]
      }
    },
    {
      "type": "Feature",
      "properties": {},
      "geometry": {
        "type": "Point",
        "coordinates": [
          114.346604347229,
          30.476518897432545
        ]
      }
    }
  ]
}
``````



## 安装shapely

本测试基于python——python3.6

``````
$ pip install shapely
``````

windows安装shapely会报错



## shapely解析地理围栏

话不多说直接上代码

``````python
from shapely.geometry import Point
from shapely.geometry.polygon import Polygon

point = Point(0.5, 0.5)
polygon = Polygon([(0, 0), (0, 1), (1, 1), (1, 0)])
print(polygon.contains(point))
``````

下面是实际的实例：

``````
from shapely.geometry import Point
from shapely.geometry.polygon import Polygon

polygon_data= [
            [
              114.3458104133606,
              30.476167529462785
            ],
            [
              114.34512376785278,
              30.475575748963195
            ],
            [
              114.34576749801636,
              30.474540124433936
            ],
            [
              114.3467652797699,
              30.475363076967565
            ],
            [
              114.34693694114685,
              30.476102803645833
            ],
            [
              114.3458104133606,
              30.476167529462785
            ]
          ]
          
point1 = Point([114.34605717658997,30.475584995561178])
point2 = Point([114.346604347229,30.476518897432545])
polygon = Polygon(polygon_data)
print(polygon.contains(point1))
print(polygon.contains(point2))
``````

输出结果：

```
True
False
```



这样一来我们就快速的实现了，目标点是否在地理围栏内的判断。

## 总结

Python还是挺好用的：）

## 参考资料：

> https://stackoverflow.com/questions/36399381/whats-the-fastest-way-of-checking-if-a-point-is-inside-a-polygon-in-python

