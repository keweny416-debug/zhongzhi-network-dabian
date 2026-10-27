### 模块6 无线网络设备（从有线网络延伸到无线介质，认识WLAN标准、设备形态并完成家庭与企业AC+瘦AP两类组网）
#### 任务24 无线网络认知
##### 6.1 无线网络【p.158】
- 核心概念：无线网络是利用电磁波（射频信号）在自由空间传输数据，无需敷设物理线缆的通信网络。按覆盖范围可分为无线个域网（蓝牙/Zigbee）、无线局域网（WLAN，基于IEEE 802.11）、无线城域网与无线广域网（4G/5G）。它解决了移动终端上网、临时布网和难以布线区域的接入问题，带来了移动性与部署灵活性，但也存在速率受距离/遮挡影响、易受干扰、安全性弱于有线等特点。
- 关键命令/配置步骤：本小节为概念课，无命令。演示/流程要点：现场拿出手机连接教室Wi-Fi，说明信号就是由电磁波在空中传播；对比有线网必须插网线。
- 教学要点：生活类比——无线网络像广播电台，广播站（AP）在空中发信号，带收音机（手机）的人走到哪都能听，但墙太厚、太远就听不清。板书要点：电磁波介质、WPAN/WLAN/WMAN/WWLAN分类、移动性、干扰与安全。课堂演示：现场断开教室有线，用手机连Wi-Fi上网，直观演示无线接入。
- 答辩角度：一、无线网络相比有线优缺点？答：优移动灵活部署快；缺点速率不稳、易干扰、安全弱。二、常见无线网络按范围分哪几类？答：个域网、局域网、城域网、广域网。前后关联：是模块6开篇，引出6.2无线局域网聚焦WLAN。
##### 6.2 无线局域网【p.160】
- 核心概念：无线局域网（WLAN）是以无线介质替代部分有线介质构建的局域网，核心标准为IEEE 802.11系列（如802.11n/ac/ax即Wi-Fi 4/5/6）。基本组成包括：STA（终端，如手机/笔记本无线网卡）、AP（接入点，把无线帧转换成有线帧）、DS（分布式系统，通过有线交换机把多个AP连起来）。终端通过关联（Association）接入AP，使用SSID（服务集标识，即Wi-Fi名称）识别不同无线网络，通过WEP/WPA/WPA2/WPA3进行加密认证。
- 关键命令/配置步骤：本小节为概念课，无命令。演示/流程要点：在eNSP或真机上列出手机Wi-Fi列表，讲解SSID就是列表里看到的那些名字；现场用抓包思路说明关联过程。
- 教学要点：生活类比——SSID就像每家奶茶店的招牌，路过能看到一串店名（Wi-Fi名），你想进哪家就点哪家并报暗号（密码）。板书要点：802.11标准、STA/AP/DS三要素、SSID、Association关联、WPA2加密。课堂演示：手机Wi-Fi列表展示多个SSID，连入教室Wi-Fi，说明802.11n/ac速率差异。
- 答辩角度：一、SSID作用？答：无线局域网的名称标识，供终端选择接入。二、WLAN基本组成三要素？答：STA、AP、DS。前后关联：上承6.1无线网络分类，引出6.3常见无线局域网设备。
##### 6.3 常见无线局域网设备【p.161】
- 核心概念：常见WLAN设备主要包括：FAT AP（胖AP，自带管理与拨号功能，家庭无线路由器即此类，适合小规模）、FIT AP（瘦AP，本身不做配置，由AC统一管理，适合中大规模部署）、AC（无线控制器，集中管理大量瘦AP、下发配置与射频策略）、无线网卡（STA端设备）、POE交换机（通过网线同时给AP供电与传数据）以及各类室内/室外天线。胖AP自治、瘦AP+AC集中管理是两种主流组网形态。
- 关键命令/配置步骤：本小节为概念课，无命令。演示/流程要点：实物或图片展示家用无线路由器（胖AP一体）与企业级瘦AP+AC设备外观，对比"一根天线盒子"和"可被集中网管的吸顶AP"。
- 教学要点：生活类比——胖AP像一位全能的个体店长，自己管店自己收银；瘦AP像连锁店里的服务员，只管迎客，所有政策由总部（AC）统一下发。板书要点：FAT/FIT AP区别、AC作用、POE供电、STA网卡、两种组网形态对比。课堂演示：实物展示家用路由器与企业AP，讲解"胖AP自治、瘦AP被AC管"。
- 答辩角度：一、胖AP和瘦AP怎么选？答：家庭小店用胖AP即插即用；企业多AP漫游用瘦AP+AC集中管理。二、AC是干什么的？答：集中管理瘦AP，下发SSID/安全/VAP模板，负责AP上线与漫游。前后关联：上承6.2 WLAN组成，引出6.4家庭（胖AP）与6.5企业（AC+瘦AP）两个实验。
#### 任务25 家庭无线网络搭建实验
##### 6.4 家庭无线网络搭建实验【p.164】
- 核心概念：家庭无线网络搭建是用一台家用无线路由器（胖AP一体设备）完成：光猫/宽带网线接入路由器WAN口，LAN口接台式机或通过Wi-Fi接手机；在Web管理界面配置上网方式（PPPoE拨号/动态IP/静态IP），设置无线SSID与加密方式（推荐WPA2-PSK，AES），并修改管理员默认密码。这是把前面所学SSID、加密、网关路由知识在生活场景中的综合落地。
- 关键命令/配置步骤：家用路由器为Web图形界面配置，无华为VRP命令。演示/流程要点：登录家用路由器管理页（教材具体管理地址/参数待核）→设置向导选PPPoE填宽带账号密码→无线设置填SSID、选WPA2-PSK/AES、设密码→修改管理员密码→手机连接验证上网。
- 教学要点：生活类比——装家里Wi-Fi像装一门"家庭广播"：先把宽带线插进WAN口（接信号源），再给自己广播起个名字（SSID）、上把锁（WPA2密码），最后别用出厂默认密码（防被蹭网）。板书要点：WAN/LAN口区分、PPPoE拨号、SSID命名、WPA2-PSK加密、修改管理员密码。课堂演示：教师机浏览器登录家用路由器后台，走一遍设置向导，手机连入后打开网页验证。
- 答辩角度：一、为什么WLAN推荐WPA2-PSK而不用WEP？答：WEP已被破解不安全，WPA2-PSK采用AES加密强度高。二、WAN口和LAN口能不能混插？答：不能，WAN接上级宽带，LAN接内网终端，插错会无法上网。前后关联：上承6.3胖AP形态，是企业级AC+瘦AP（6.5）的对照实验。
#### 任务26 AC+瘦AP组网
##### 6.5 AC+瘦AP网络组建【p.171】
- 核心概念：AC+瘦AP组网是企业级WLAN主流方案：瘦AP本身不配置，上电后通过DHCP或广播/组播自动发现AC，在AC上注册上线；AC统一向AP下发SSID模板、安全模板、VAP模板（把SSID与射频绑定），并通过CAPWAP隧道管理AP。组网大致步骤为：配置AC与AP路由可达（管理VLAN/AC源接口IP）、AP上线认证（MAC认证或SN认证）、配置SSID模板与安全模板（WPA2-PSK）、配置VAP模板并绑定到AP组、终端关联SSID上网。教材具体命令细节待核。
- 关键命令/配置步骤：
```
// 以下为华为WLAN通行配置流程示意，教材具体命令细节待核
[AC6005] capwap source interface vlanif 100     // 指定AC与AP建立CAPWAP隧道的源接口
[AC6005] wlan                                    // 进入WLAN视图
[AC6005-wlan-view] ap-group name ap-group1        // 创建AP组
[AC6005-wlan-ap-group-ap-group1] quit
[AC6005-wlan-view] regulatory-domain-profile name domain1   // 国家码/域模板（待核）
[AC6005-wlan-regulate-domain-domain1] quit
[AC6005-wlan-view] ssid-profile name ssid1       // 创建SSID模板
[AC6005-wlan-ssid-prof-ssid1] ssid My-WiFi       // 设置SSID名称
[AC6005-wlan-ssid-prof-ssid1] quit
[AC6005-wlan-view] security-profile name sec1    // 创建安全模板
[AC6005-wlan-sec-prof-sec1] security wpa2 psk pass-phrase Huawei@123 aes
[AC6005-wlan-sec-prof-sec1] quit
[AC6005-wlan-view] vap-profile name vap1         // 创建VAP模板，绑定SSID与安全模板
[AC6005-wlan-vap-prof-vap1] forward-mode tunnel
[AC6005-wlan-vap-prof-vap1] ssid-profile ssid1
[AC6005-wlan-vap-prof-vap1] security-profile sec1
[AC6005-wlan-vap-prof-vap1] quit
[AC6005-wlan-view] ap-group name ap-group1
[AC6005-wlan-ap-group-ap-group1] vap-profile vap1 wlan 1 radio all
[AC6005-wlan-ap-group-ap-group1] quit
<AC6005> display ap all                          // 查看AP上线状态，状态应为nor/已正常
```
- 教学要点：生活类比——AC是连锁总部，AP是店员：店员（瘦AP）进店先找总部报到（自动发现AC、注册上线），总部再统一发工作服与话术（SSID/安全/VAP模板），顾客连进来的Wi-Fi其实是总部统一的招牌。板书要点：CAPWAP隧道、AP自动发现并注册、AC源接口、SSID/安全/VAP三模板绑定AP组、display ap all看上线。课堂演示：eNSP中AC+AP拓扑，先配置路由与DHCP让AP拿到管理地址，再在AC上display ap all看到AP从idle变nor（上线成功），手机搜出My-WiFi连入。
- 答辩角度：一、瘦AP为什么不能单独用？答：它无本地配置，必须由AC通过CAPWAP隧道集中下发配置与管理。二、AP上线失败先查什么？答：查AC与AP管理VLAN路由可达、查capwap source源接口、查AP是否被认证（MAC/SN白名单）。三、VAP模板起什么作用？答：把SSID模板与安全模板绑定到AP射频，决定该射频对外广播哪个Wi-Fi。前后关联：是模块6收束与企业WLAN综合应用，与6.4家庭胖AP实验形成"胖/瘦AP"两种形态的完整对照。

