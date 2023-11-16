---
title: SpiderTools
date: 2020-10-24  12:57:17
cover: 
tags: 
  - spider
  - 爬虫
---





# 爬虫 常用的第三方库

请求库

- requests
- urllib
- aiohttp
- Selenium
- ChomeDrive

解析库

- xpath

- jsonpath

- demjson

```python
什么是demjson?
此模块提供用于编码或解码数据的类和函数。这个实现试图尽可能符合JSON规范（RFC 4627），同时仍然提供许多可选的扩展，以允许限制较少的JavaScript语法。它包括完整的Unicode支持，包括UTF-32、BOM和代理项对处理。它还可以支持JavaScript的NaN和无限数值类型，以及它的“undefined”类型。它还包括一个类似lint的JSON语法验证器，用于测试JSON文本是否严格符合标准。

官方文档：https://pypi.org/project/demjson/1.6/

为什么json库不能用？
python中json模块能解析的对象格式是有限的，查看下面这三种格式，唯独第三种可以用json.loads方法。

# javascript中的对象，json.loads(js_json)会报错
js_json1 = "{x:1, y:2, z:3}"
 
# python打印出来的字典，json.loads(js_json1)会报错
py_json1 = "{'x':1, 'y':2, 'z':3}"
 
 
# 唯独这种格式，json.loads(py_json2)不会报错，得到{'x': 1, 'y': 2, 'z': 3}
py_json2 = '{"x":1, "y":2, "z":3}'
 

demjson的安装：
pip3 install demjson -i https://pypi.douban.com/simple

上面json模块无法解析的情况，demjson都能搞定：
import demjson
 
js_json = "{x:1, y:2, z:3}"
 
py_json1 = "{'x':1, 'y':2, 'z':3}"
 
py_json2 = '{"x":1, "y":2, "z":3}'
 
data = demjson.decode(js_json)
print(data)
# {'y': 2, 'x': 1, 'z': 3}
 
data = demjson.decode(py_json1)
print(data)
# {'y': 2, 'x': 1, 'z': 3}
 
data = demjson.decode(py_json2)
print(data)
# {'y': 2, 'x': 1, 'z': 3}
demjson的decode 和encode方法：
encode 将 Python 对象编码成 JSON 字符串
decode 
将已编码的 JSON 字符串解码为 Python 对象

代码示例：
import demjson
 
data = [{'a': 1, 'b': 2, 'c': 3, 'd': 4, 'e': 5}]
print(demjson.encode(data)) #[{"a":1,"b":2,"c":3,"d":4,"e":5}]
 
json = '{"a":1,"b":2,"c":3,"d":4,"e":5}';
print(demjson.decode(json)) # {'a': 1, 'b': 2, 'c': 3, 'd': 4, 'e': 5}
```

- lxml

- pyquery

- beautifulsoup

- tesserocr

伪装代理

- fake_useragent : 生成随机UserAgent
  - Github： <https://github.com/hellysmile/fake-useragent>

```python
# 安装
pip installfake-useragent
# 查看 useragent
http://fake-useragent.herokuapp.com/browsers/0.1.5

# 使用
ua = UserAgent()
print(ua.ie)   #随机打印ie浏览器任意版本
print(ua.firefox) #随机打印firefox浏览器任意版本
print(ua.chrome)  #随机打印chrome浏览器任意版本
print(ua.random)  #随机打印任意厂家的浏览器

https://www.cnblogs.com/qingchengzi/p/9633616.html
```

- IPProxy

```python

```

APP爬取相关库

- mitmproxy
- Android

数据库存储

- pymysql
- pymongo
- redis
- redisdump
- pymssql

爬虫框架

- scrapy
- Crawley
- portia

web库

- flask：轻量级web服务器 简单易用灵活 主要做API服务
- django：web服务器框架，提供完整的后台管理，引擎，接口等可以使用它做一个完整的网站
