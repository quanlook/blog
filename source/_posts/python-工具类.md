---
title: Python-工具类
date: 2022-12-17 23:37:32
tags: 
  - python
keywords: "python"
cover: https://tuapi.eees.cc/api.php?category=dongman&type=302
toc: True
categories: 
  - python
---

# Python 工具类

## 纯真IP解析

```sh
F:.
|   IPAddress.py
|
\---data
        qqwry.dat
```

`IPAddress.py`

```python
#! /usr/bin/env python
# -*- coding: utf-8 -*-
import os
import requests
import socket
import struct

import logging


logger = logging.getLogger('util')


class IPAddresss:
    def __init__(self, ipdbFile):
        self.ipdb = open(ipdbFile, "rb")
        str = self.ipdb.read(8)
        (self.firstIndex, self.lastIndex) = struct.unpack('II', str)
        self.indexCount = int((self.lastIndex - self.firstIndex) / 7 + 1)
        # print self.getVersion(), u" 纪录总数: %d 条 "%(self.indexCount)

    def getVersion(self):
        s = self.getIpAddr(0xffffff00)
        return s

    def getAreaAddr(self, offset=0):
        if offset:
            self.ipdb.seek(offset)
        str = self.ipdb.read(1)
        (byte,) = struct.unpack('B', str)
        if byte == 0x01 or byte == 0x02:
            p = self.getLong3()
            if p:
                return self.getString(p)
            else:
                return ""
        else:
            self.ipdb.seek(-1, 1)
            return self.getString(offset)

    def getAddr(self, offset, ip=0):
        self.ipdb.seek(offset + 4)
        countryAddr = ""
        areaAddr = ""
        str = self.ipdb.read(1)
        (byte,) = struct.unpack('B', str)
        if byte == 0x01:
            countryOffset = self.getLong3()
            self.ipdb.seek(countryOffset)
            str = self.ipdb.read(1)
            (b,) = struct.unpack('B', str)
            if b == 0x02:
                countryAddr = self.getString(self.getLong3())
                self.ipdb.seek(countryOffset + 4)
            else:
                countryAddr = self.getString(countryOffset)
            areaAddr = self.getAreaAddr()
        elif byte == 0x02:
            countryAddr = self.getString(self.getLong3())
            areaAddr = self.getAreaAddr(offset + 8)
        else:
            countryAddr = self.getString(offset + 4)
            areaAddr = self.getAreaAddr()
        return countryAddr + " " + areaAddr

    def dump(self, first, last):
        if last > self.indexCount:
            last = self.indexCount
        for index in range(first, last):
            offset = self.firstIndex + index * 7
            self.ipdb.seek(offset)
            buf = self.ipdb.read(7)
            (ip, of1, of2) = struct.unpack("IHB", buf)
            address = self.getAddr(of1 + (of2 << 16))
            # 把GBK转为utf-8
            address = str(address, 'gbk').encode("utf-8")
            logger.info("%d %s %s" % (index, self.ip2str(ip), address))

    def setIpRange(self, index):
        offset = self.firstIndex + index * 7
        self.ipdb.seek(offset)
        buf = self.ipdb.read(7)
        (self.curStartIp, of1, of2) = struct.unpack("IHB", buf)
        self.curEndIpOffset = of1 + (of2 << 16)
        self.ipdb.seek(self.curEndIpOffset)
        buf = self.ipdb.read(4)
        (self.curEndIp,) = struct.unpack("I", buf)

    def getIpAddr(self, ip):
        L = 0
        R = self.indexCount - 1
        while L < R - 1:
            M = int((L + R) / 2)
            self.setIpRange(M)
            if ip == self.curStartIp:
                L = M
                break
            if ip > self.curStartIp:
                L = M
            else:
                R = M
        self.setIpRange(L)
        # version information, 255.255.255.X, urgy but useful
        if ip & 0xffffff00 == 0xffffff00:
            self.setIpRange(R)
        if self.curStartIp <= ip <= self.curEndIp:
            address = self.getAddr(self.curEndIpOffset)
            # 把GBK转为utf-8
            address = str(address)
        else:
            address = "未找到该IP的地址"
        return address

    def getIpRange(self, ip):
        self.getIpAddr(ip)
        range = self.ip2str(self.curStartIp) + ' - ' \
                + self.ip2str(self.curEndIp)
        return range

    def getString(self, offset=0):
        if offset:
            self.ipdb.seek(offset)
        str = b''
        ch = self.ipdb.read(1)
        (byte,) = struct.unpack('B', ch)
        while byte != 0:
            str += ch
            ch = self.ipdb.read(1)
            (byte,) = struct.unpack('B', ch)
        return str.decode('gbk')

    def ip2str(self, ip):
        return str(ip >> 24) + '.' + str((ip >> 16) & 0xff) + '.' + str((ip >> 8) & 0xff) + '.' + str(ip & 0xff)

    def str2ip(self, s):
        (ip,) = struct.unpack('I', socket.inet_aton(s))
        return ((ip >> 24) & 0xff) | ((ip & 0xff) << 24) | ((ip >> 8) & 0xff00) | ((ip & 0xff00) << 8)

    def getLong3(self, offset=0):
        if offset:
            self.ipdb.seek(offset)
        str = self.ipdb.read(3)
        (a, b) = struct.unpack('HB', str)
        return (b << 16) + a



def get_ipAddress(ip):
    """
    docstring
    """
    QQWRY_PATH = os.path.dirname(__file__) + "./data/qqwry.dat"
    ips = IPAddresss(QQWRY_PATH)
    return ips.getIpAddr(ips.str2ip(ip))

def ipAPI(ip):
    """
    https://ip-api.com/docs/api:json
    """
    url="http://ip-api.com/json/{}".format(ip)
    data = requests.get(url).text
    return data

if __name__ == "__main__":
    print(get_ipAddress("8.8.8.8"))
    
```

## 配置代理

```python
# -*- coding: utf-8 -*-#

import requests
import demjson
import socket
import socks
socks.set_default_proxy(socks.SOCKS5, "192.168.1.100", 1081)
socket.socket = socks.socksocket

def main():
    # 1.国际化显示：归属地可切换展示
    # http://ip-api.com/json/　　# 国际化英文显示
    # http://ip-api.com/json/?lang=zh-CN　　# 中文显示
    # http://ip-api.com/json/115.191.200.34?lang=zh-CN　　# 查询某个ip的信息
    urls = ['http://ip-api.com/json/', 
            'http://ip-api.com/json/?lang=zh-CN',
            'http://sspanel.net/ip.php', 
            'http://45.32.164.128/ip.php', 
            'https://www.google.com'
    ]
    html = requests.get(urls[1]).content.decode("utf8")
    # demjson.encode(html,"utf-8")
    return html
```
