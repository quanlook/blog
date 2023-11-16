---
title: Scrapy 运行多个爬虫
date: 2020-10-25 19:51:19
tags:
  - scrapy
  - 爬虫
categories: 
  - 爬虫
cover: 
---



# Scrapy 运行多个爬虫

本文所使用的 Scrapy 版本：Scrapy==2.4.0

一个 Scrapy 项目下可能会有多个爬虫，本文陈述两种情况：

```powershell
d:\Code\demo\search_project
> tree /f /a

D:.
|   README.md
|   run.py
|   scrapy.cfg
\---search_project
    |   items.py
    |   middlewares.py
    |   pipelines.py
    |   settings.py
    |   __init__.py
    \---spiders
            demo1.py   # 爬虫1
            demo11.py  # 爬虫2
            test.py    #
            Baidu.py #
            Bing.py  #
            __init__.py
```

1. 多个爬虫
2. 所有爬虫

显然，这两种情况并不一定是等同的。假设当前项目下有 3 个爬虫，分别名为：test、hello、experience，并在项目目录下创建一个 `main.py` 文件，下面的示例代码都写在这个文件中，项目执行时，在命令行下执行 `python main.py` 或者在 PyCharm 中把这个脚本文件设置为执行脚本即可。

## 运行多个爬虫

核心点：使用 `CrawlerProcess`

代码如下：

```python
from scrapy.crawler import CrawlerProcess
from scrapy.utils.project import get_project_settings

# 根据项目配置获取 CrawlerProcess 实例
process = CrawlerProcess(get_project_settings())

# 添加需要执行的爬虫
process.crawl('test')
process.crawl('hello')
process.crawl('experience')

# 执行
process.start()
```

## 运行所有爬虫

核心点：使用 `SpiderLoader`

代码如下：

```python
from scrapy.crawler import CrawlerProcess
from scrapy.utils.project import get_project_settings
from scrapy.spiderloader import SpiderLoader

# 根据项目配置获取 CrawlerProcess 实例
process = CrawlerProcess(get_project_settings())

# 获取 spiderloader 对象，以进一步获取项目下所有爬虫名称
spider_loader = SpiderLoader(get_project_settings())

# 添加需要执行的爬虫
for spidername in spider_loader.list():
    process.crawl(spidername)

# 执行
process.start()
```

## 关于 ScrapyCommand

网上有部分文章说到使用 `ScrapyCommand` 这个类下面的 `crawler_process.list()` 方法可以得到项目下的所有爬虫名称，但我在最新的官方文档中已经搜索不到 `ScrapyCommand` 这个类，估计是已经弃用了

## 其它

如果需要向某个爬虫传递参数，可以在 `process.crawl` 方法中添加进去，例如：

```python
process.crawl('test', dt='20201120')
```

则在 `dining` 这个爬虫（类）中，可以在 `__init__` 方法中接收这个 `dt` 参数。例如：

```python
class Dining(scrapy.Spider):
    name = 'test'
    
    def __init__(self, dt)
        self.dt = dt
```
