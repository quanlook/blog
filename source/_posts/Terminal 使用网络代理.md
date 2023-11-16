---
title: Terminal 使用网络代理
date: 2020-10-24  12:57:17
cover: 
tags: 
  - proxy
  - 代理
  - 命令行
  - Terminal
---


# Terminal 使用网络代理

windows

cmd

```
set http_proxy=http://127.0.0.1:1189
set https_proxy=http://127.0.0.1:1189
set http_proxy=http://127.0.0.1:1080

set https_proxy=http://127.0.0.1:1080

set http_proxy_user=user
set http_proxy_pass=pass

set https_proxy_user=user
set https_proxy_pass=pass

# 恢复
set http_proxy=

set https_proxy=

# Ubuntu 下命令为 export
# export http_proxy=http://127.0.0.1:1080


环境：shadowsocks、windows
本地ss端口设置(这里1080)

cmd命令行:(不用socks5)(临时设置)(也可放置环境变量)
set http_proxy=http://127.0.0.1:1080
set https_proxy=http://127.0.0.1:1080

ps:一定要用cmd命令行，千万别用powershell !!!
简易测试命令：curl https://www.google.com（别用ping）

PowerShell 的：

$env:http_proxy="http://127.0.0.1:1080"
$env:https_proxy="http://127.0.0.1:1080"
亲测可用
```

**PowerShell**

```powershell
# NOTE: registry keys for IE 8, may vary for other versions
$regPath = 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings'

function Clear-Proxy
{
    Set-ItemProperty -Path $regPath -Name ProxyEnable -Value 0
    Set-ItemProperty -Path $regPath -Name ProxyServer -Value ''
    Set-ItemProperty -Path $regPath -Name ProxyOverride -Value ''

    [Environment]::SetEnvironmentVariable('http_proxy', $null, 'User')
    [Environment]::SetEnvironmentVariable('https_proxy', $null, 'User')
}

function Set-Proxy
{
    $proxy = 'http://example.com'

    Set-ItemProperty -Path $regPath -Name ProxyEnable -Value 1
    Set-ItemProperty -Path $regPath -Name ProxyServer -Value $proxy
    Set-ItemProperty -Path $regPath -Name ProxyOverride -Value '<local>'

    [Environment]::SetEnvironmentVariable('http_proxy', $proxy, 'User')
    [Environment]::SetEnvironmentVariable('https_proxy', $proxy, 'User')
}
```

linux

```sh
export ALL_PROXY=socks5://127.0.0.1:1080
export all_proxy='http://127.0.0.1:1080来代理

export http_proxy=socks:***
export https_proxy=socks:**

export http_proxy="http://localhost:port"
export https_proxy="http://localhost:port"

export http_proxy="socks5://127.0.0.1:1080"
export https_proxy="socks5://127.0.0.1:1080"
export ALL_PROXY=socks5://127.0.0.1:1080

alias setproxy="export ALL_PROXY=socks5://127.0.0.1:1080" 
alias unsetproxy="unset ALL_PROXY"

git config --global http.proxy 'socks5://127.0.0.1:1080' 
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

mac

# [终端使用代理加速的正确方式（Shadowsocks）](https://segmentfault.com/a/1190000039686752)

我们在终端使用`Homebrew`、`git`、`npm`等命令时，总会因为网络问题而安装失败。

尤其是安装`Homebrew`，据我了解很多朋友是花了很长时间来解决，心里不知道吐槽该死的网络多少遍了。

虽然设置镜像确实有用，但是没有普适性，今天就介绍下让终端也走代理的方法，这样可以通杀很多情况。

文本提到的代理是指搭配`Shadowsocks`使用的情况，其他软件也有一定共同之处。

## macOS & Linux

通过设置`http_proxy`、`https_proxy`，可以让终端走指定的代理。
配置脚本如下，在终端直接执行，只会临时生效：

```bash
export http_proxy=http://127.0.0.1:1080
export https_proxy=$http_proxy
```

`1080`是`http`代理对应的端口，请不要照抄作业，根据你的实际情况修改。

**你可以在Shadowsocks的设置界面中查找代理端口信息。**

### 便捷脚本

这里提供一个便捷脚本，里面包含打开、关闭功能：

```bash
function proxy_on() {
    export http_proxy=http://127.0.0.1:1080
    export https_proxy=$http_proxy
    echo -e "终端代理已开启。"
}

function proxy_off(){
    unset http_proxy https_proxy
    echo -e "终端代理已关闭。"
}
```

通过`proxy_on`启动代理，`proxy_off`关闭代理。

接下来需要把脚本写入`.bash_profile`或`.zprofile`，这样就可以永久生效。

> 你可能会问，怎么写入脚本，耐心点，下文提供了安装脚本的方法。

至于你应该写入哪个文件，请根据命令`echo $SHELL`返回结果判断：

- `/bin/bash` => `.bash_profile`
- `/bin/zsh` => `.zprofile`

然后执行**安装脚本**（追加内容+生效），注意一定根据要上面结果修改`.zprofile`名称：

```bash
cat > ~/.zprofile << EOF
function proxy_on() {
    export http_proxy=http://127.0.0.1:1080
    export https_proxy=$http_proxy
    echo -e "终端代理已开启。"
}

function proxy_off(){
    unset http_proxy https_proxy
    echo -e "终端代理已关闭。"
}
EOF

source ~/.zprofile
```

可以执行`curl cip.cc`验证：

```bash
IP    : xxx
地址    : 中国  台湾  台北市
运营商    : cht.com.tw

数据二    : 台湾省 | 中华电信(HiNet)数据中心

数据三    : 中国台湾 | 中华电信

URL    : http://www.cip.cc/xxx
```

看到网上说通过`curl -I http://www.google.com`可能会遇到`403`问题，使用`Google`域名验证时需要注意下这个情况。

## Windows

根据网上的文章，在`Windows`下使用全局代理方式也会对`cmd`生效（未经验证）。

### cmd

```bash
set http_proxy=http://127.0.0.1:1080
set https_proxy=http://127.0.0.1:1080
```

还原命令：

```bash
set http_proxy=
set https_proxy=
```

### Git Bash

设置方法同"macOS & Linux"

### PowerShell

```bash
$env:http_proxy="http://127.0.0.1:1080"
$env:https_proxy="http://127.0.0.1:1080"
```

还原命令(未验证)：

```bash
$env:http_proxy=""
$env:https_proxy=""
```

## 其他代理设置

### git代理

```bash
# 设置
git config --global http.proxy 'socks5://127.0.0.1:1080' 
git config --global https.proxy 'socks5://127.0.0.1:1080'

# 恢复
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### npm

```bash
# 设置

npm config set proxy http://server:port
npm config set https-proxy http://server:port

# 恢复
npm config delete proxy
npm config delete https-proxy
```

### git clone ssh 如何走代理

#### macOS

打开`~/.ssh/config`，如果没有这个文件，自己手动创建。

```gcode
# 全局
# ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p
# 只为特定域名设定
Host github.com
    ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p
```

#### Windows

打开`C:\Users\UserName\.ssh\config`文件，没有看到的话，同样手动创建。

```gcode
# 全局
# ProxyCommand connect -S 127.0.0.1:1080 %h %p
# 只为特定域名设定
Host github.com
    ProxyCommand connect -S 127.0.0.1:6600 %h %p
```
