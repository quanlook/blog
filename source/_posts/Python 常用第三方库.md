---
title: Python-常用的第三方库整理
date: 2022-12-17 22:37:32
tags: 
  - python
keywords: "python"
cover: https://cdn.jsdelivr.net/gh/quanlook/ImgCdn/blog/202210182359259.jpg
toc: True
categories: 
  - python
---

# Python 常用的第三方库整理

## 配置pip加速

Window

```sh
mkdir ~/.pip/pip.ini

[global]
timeout =6000
index-url =http://pypi.douban.com/simple/
[install]
use-mirrors =true 
mirrors =http://pypi.douban.com/simple/ 
trusted-host =pypi.douban.com
```

Linux

```sh
mkdir ~/.pip
sudo tee ~/.pip/pip.conf <<-'EOF'
[global]
timeout =6000
index-url =http://pypi.douban.com/simple/
[install]
use-mirrors =true 
mirrors =http://pypi.douban.com/simple/ 
trusted-host =pypi.douban.com
EOF

```

## python 虚拟环境

## 常用的第三方库

### id-validator

**中华人民共和国居民身份证**、**中华人民共和国港澳居民居住证**以及**中华人民共和国台湾居民居住证**

号码验证工具（Python 版）支持 15 位与 18 位号码。仅支持 Python 3

> pypi：<https://pypi.org/project/id-validator/>
>
> Github：<https://github.com/jxlwqq/id-validator.py>

```shell
pip install id-validator
```

### 获取身份证号信息

当身份证号合法时，返回分析信息（地区、出生日期、星座、生肖、性别、校验位），不合法返回 `False`：

```python
from id_validator import validator

validator.get_info('440308199901101512') # 18 位
validator.get_info('610104620927690')    # 15 位
```

返回信息格式如下：

```json
{
'address_code'   : '440308',                   # 地址码
'abandoned'      : 0,                          # 地址码是否废弃，1 为废弃的，0 为正在使用的
'address'        : '广东省深圳市盐田区',          # 地址
'address_tree'   : ['广东省', '深圳市', '盐田区'] # 省市区三级列表
'age'            : 21,                          # 年龄，当前的年份减去出生年份，例：2020-1999=21
'birthday_code'  : '1999-01-10',               # 出生日期
'constellation'  : '摩羯座',                    # 星座
'chinese_zodiac' : '卯兔',                      # 生肖
'sex'            : 1,                          # 性别，1 为男性，0 为女性
'length'         : 18,                         # 号码长度
'check_bit'      : '2'                         # 校验码
}
```

### 城市地址解析工具cpca

一个用于提取简体中文字符串中省，市和区并能够进行映射，检验和简单绘图的python模块

> chinese_province_city_area_mapper
>
> <https://github.com/DQinYuan/chinese_province_city_area_mapper>

```sh
pip install cpca
```

```python
import cpca
location_str = ["徐汇区虹漕路461号58号楼5楼", "泉州市洛江区万安塘西工业区", "北京朝阳区北苑华贸城"]
st = "广西|桂林|兴安县|界首镇|界首街|界首市场|3号"
location_str1 = [st.replace("|", "")]
df = cpca.transform(location_str)
print(df)

df = cpca.transform(location_str1)
print(df)
shengName=df.iat[0,0]
shiName=df.iat[0,1]
print(shengName,shiName,df.iat[0,2])
--------
     省    市    区             地址  adcode
0  上海市  市辖区  徐汇区  虹漕路461号58号楼5楼  310104
1  福建省  泉州市  洛江区        万安塘西工业区  350504
2  北京市  市辖区  朝阳区          北苑华贸城  110105
         省    市    区            地址  adcode
0  广西壮族自治区  桂林市  兴安县  界首镇界首街界首市场3号  450325
广西壮族自治区 桂林市 兴安县
```

### phone

手机号归属地

> 手机号码库
>
> <https://github.com/ls0f/phone>
>
> <https://pypi.org/project/phone/>

```sh
pip install phone
```

```python
from phone import Phone
p  = Phone()
p.find(1888888)
```

### faker

 `Faker` 伪造数据工具库
> <https://github.com/joke2k/faker>
>
> <https://pypi.org/project/Faker/>
>
> [**官方文档**](http://faker.readthedocs.io/en/master/)

```sh
pip install Faker
```

```python
from faker import Faker

fake = Faker("zh_CN")


#如上面例子，每次调用 fake 实例的 `name()`方法时，都会产生不同随机姓名。fake 实例还有很多方法可用，这些方法分为以下几类：

# - address 地址
# - person 人物类：性别、姓名等
# - barcode 条码类
# - color 颜色类
# - company 公司类：公司名、公司email、公司名前缀等
# - credit_card 银行卡类：卡号、有效期、类型等
# - currency 货币
# - date_time 时间日期类：日期、年、月等
# - file 文件类：文件名、文件类型、文件扩展名等
# - internet 互联网类
# - job 工作
# - lorem 乱数假文
# - misc 杂项类
# - phone_number 手机号码类：手机号、运营商号段
# - python python数据
# - profile 人物描述信息：姓名、性别、地址、公司等
# - ssn 社会安全码(身份证号码)
# - user_agent 用户代理

### person 人物

>>> fake.name() # 姓名
'单玉珍'
>>> fake.last_name() # 姓
'潘'
>>> fake.first_name() # 名
'琴'
>>> fake.name_male() # 男性姓名
'官平'
>>> fake.last_name_male() # 男性姓
'安'
>>> fake.first_name_male() # 男性名
'文'
>>> fake.name_female() # 女性姓名
'许颖'
```
