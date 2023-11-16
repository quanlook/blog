---
title: Scrapy集成Selenium下载中间件
date: 2020-10-25 20:30:19
tags:
  - scrapy
  - 爬虫
cover: 
---

## Scrapy框架集成 Selenium下载中间件

- spider

```python
import scrapy
from scrapy import signals
from selenium import webdriver
from selenium.webdriver import FirefoxOptions

class JdSpider(scrapy.Spider):
    name = 'jd'
    allowed_domains = ['jd.com']
    start_urls = [
        'https://www.jd.com/',
        'https://miaosha.jd.com/'
        ]
 
    @classmethod
    def from_crawler(cls, crawler, *args, **kwargs):
        spider =super(JdSpider, cls).from_crawler(crawler, *args, **kwargs)
        # 创建 浏览器
        # option = FirefoxOptions()
        # option.headless = True # 设置浏览器无头模式 不显示UI
        # spider.driver = webdriver.Firefox(options=option)
        spider.driver = webdriver.Firefox()
        crawler.signals.connect(spider.spider_closed, signal=signals.spider_closed)
        return spider
    
    def spider_closed(self, spider):
        spider.driver.close() #关闭浏览器
        print("==========爬虫结束！")
        spider.logger.info('Spider closed:%s', www.spider.name)
        

    def parse(self, response):
        item = {}
        # print(response.text)
        # item["countdown"] = response.xpath('//div[@class="countdown-title"]/text()').extract_first()
        print(response.url)
```

- [middlewares.py](https://link.zhihu.com/?target=http%3A//middlewares.py) 配置

```python
class SeleniumTaobaoDownloaderMiddleware(object):
    def process_request(self, request, spider):
        spider.driver.get(request.url)
        # js 滚动加载数据
        # 由于页面数据加载需要进行滚动，但并不是所有js动态数据都需要滚动。
        # for x in range(1, 20, 2):
        #     height = float(x) / 10
        #     js = "document.documentElement.scrollTop = document.documentElement.scrollHeight * %f" % height
        #     spider.driver.execute_script(js)
        #     time.sleep(0.2)

        # 将获取的html构造成为一个Response对象，并返回。
        return HtmlResponse(
            url=request.url, 
            body=spider.driver.page_source, 
            request=request, 
            encoding='utf8'
            )


    def process_response(self, request, response, spider):
        return response
```

- [settings.py](https://link.zhihu.com/?target=http%3A//settings.py) 配置

```python
# Enable or disable downloader middlewares
# See Downloader Middleware
DOWNLOADER_MIDDLEWARES = {
#    'scrapy_selenium.middlewares.ScrapySeleniumDownloaderMiddleware': 543,
   'scrapy_selenium.middlewares.SeleniumTaobaoDownloaderMiddleware': 543,
}
```
