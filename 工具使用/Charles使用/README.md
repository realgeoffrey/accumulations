# [Charles](https://www.charlesproxy.com/)使用

## 目录
1. [面板](#面板)
1. [显示捕获列表方式](#显示捕获列表方式)
1. [查找](#查找)
1. [代理配置](#代理配置)
1. [捕获系统请求](#捕获系统请求)
1. [捕获移动端请求](#捕获移动端请求)
1. [Map至本地、网址](#map至本地网址)
1. [网速模拟功能](#网速模拟功能)
1. [断点功能](#断点功能)
1. [重复发送请求](#重复发送请求)
1. [HTTPS分析](#https分析)
1. [捕获记录控制](#捕获记录控制)
1. [web界面](#web界面)

>破解：<https://github.com/8enet/Charles-Crack>

HTTP调试工具：通过将Charles设置成**系统的网络访问代理服务器**，使得所有的网络访问请求都通过它来完成，从而实现网络封包的截取和分析。

>1. 需要安装同样位数的**java**，所有浏览器都可以使用Charles来截取请求，前提是必须关闭防火墙和关闭浏览器代理插件（或设置使用系统代理）。
>2. 移动设备设置HTTP代理之后就能以代理环境并经过电脑hosts访问网络（HTTP代理至WAMPSERVER无法经过电脑hosts）。
>
>- Charles默认端口号：**8888**。

### 面板

![Charles图](./images/1.png)

>当你刷新页面时，只会在当前session中捕获请求。当关闭session后会导致已经在此session捕获的网站无法在其他session捕获，需要重新启动Charles或新建session。

### 显示捕获列表方式
1. Structure

    视图将网络请求按访问的域名分类

    ![Charles图](./images/3.png)

2. Sequence

    视图将网络请求按访问的时间排序（有**Filter**功能）

    ![Charles图](./images/4.png)

>针对一个域名或网址，右键选择**Focus**或**Ignore**可以专门查看或忽略请求。

### 查找

Ctrl/commad + f调出。

![Charles图](./images/5.png)

### 代理配置

![Charles图](./images/16.png)

### 捕获系统请求

设置是否成为系统代理，取消勾选**Window Proxy**或**macOS Proxy**后不再抓取本机请求（仅抓取8888端口号内容）。

![Charles图](./images/14.png)

### 捕获移动端请求

移动设备的调试，我们需要把HTTP代理到Charles上：

1. 使移动设备和本机在一个局域网内（不需要同一个IP段、也不需要是电脑发射的Wi-Fi，只要是同一个路由器下即可）；

    >若无法抓取信息，则可以重启Charles、重新设置HTTP代理或查看**Access Control Settings**是否允许访问。有可能某些路由器设置成无法捕获，就需要使用自备发射的Wi-Fi。
2. 移动设备的Wi-Fi设置：服务器IP设置成电脑IP，端口号设置成8888；

    ![Charles图](./images/13.png)
3. Charles设置允许接收的IP地址的范围（有新的不在允许范围内的IP地址请求时，会自动询问是否允许接收）：

    ![Charles图](./images/15.png)

    >进入设置**Proxy - Access Control Settings**，若接收的IP范围是192.168.1.xxx的话，则添加并设置成`192.168.1.0/24`；若全部范围都接收的话，则直接设置成`0.0.0.0/0`。

>Charles支持HTTPS和HTTP，不支持sockets。

### Map至本地、网址
- 设置路由

    1. Map来源：

        1. `路径/*`表示：路径下所有内容、子路径下所有内容。
        2. `路径/路由`表示：仅此路径下的一个路由。
    2. Map去向：

        - 都不要添加 ~~`*`~~（由来源匹配去向）。

1. Map至本地：

    1. 将**单文件**代理至本地的源码文件：

        ![Charles图](./images/7.png)

        点击「Map Local」后：

        ![Charles图](./images/8.png)

    2. 将**子目录**代理至本地，使用通配符`*`：

        ![Charles图](./images/9.png)

        本地地址选择个子目录，不需要通配符。

    - 校验是否代理成功，最省力的方式是点击工具条上的刷新按钮，刷新单个请求，若代理成功，则Charles会往「Notes」界面打个log:

        ![Charles图](./images/10.png)
2. Map至网址：

    选择「Map Remote」

    >若勾选了`preserve host in header fields`，则发起请求的请求头`host`会保持Map来源的值，否则`host`是Map去向的值。

    ![Charles图](./images/6.png)

    ![Charles图](./images/12.png)

### 网速模拟功能

![Charles图](./images/17.png)

### 断点功能

Charles能够截断发送请求前（篡改Request）和响应后（篡改Response），修改request和response的所有信息。

场景：发送HTTP请求，我们需要测试接口的各种边界情况（如：出错、超时等表现），Charles的断点+随意篡改，非常方便测试，右键选择「BreakPoints」，开启断点：

![Charles图](./images/19.png)

>小技巧：断点配合repeat功能：
>
>![Charles图](./images/20.png)

断点列表查看：

![Charles图](./images/21.png)

![Charles图](./images/22.png)

### 重复发送请求

repeat功能不需要刷新页面，只需要repeat请求，还可以进行压力测试。

**repeat**重复发送一次请求；**repeat Advances**可以自定义重复次数和重复间隔。

![Charles图](./images/23.png)

![Charles图](./images/24.png)

### HTTPS分析

若未安装Charles根证书，则仅能捕获加密后的HTTPS请求。

1. 安装、信任证书：

    1. 电脑安装证书

        1. 在 **Help -> SSL Proxying -> Install Charles Root Certificate**安装证书；
        2. 去系统设置信任此证书：

            1. macOS：在**钥匙串访问**中双击设置信任

                ![Charles图](./images/11.png)
            2. Windows：把证书安装至**受信任的根证书颁发机构**
    2. 移动设备安装证书

        移动设备设置好HTTP代码后，访问[chls.pro/ssl](chls.pro/ssl)下载证书并信任。

        >Android系统在设置中搜索「证书」关键字。
2. 设置需要针对的域名：

    设置**Proxy -> SSL Proxying Settings**，把需要抓包的`host+端口号443`添加进去。

    ![Charles图](./images/26.png)

### 捕获记录控制

捕获的请求太多，容易产生干扰，Charles可以对捕获记录进行过滤。

![Charles图](./images/25.png)

然后配置「exclude」：

![Charles图](./images/18.png)

### web界面

Charles有个有趣的web界面：

![Charles图](./images/2.png)

强大的是可以控制是否远程可以访问这个界面，还可以设置用户名和密码。
浏览器输入<http://control.charles/>

>tips：若charles是非正常状态下关闭的话，则有些浏览器的代理就不会被自动取消，会出现无法访问网络情况。重新打开Charles再正常关闭可以解决。
