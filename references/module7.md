### 模块7 网络安全接入（从操作系统基线、接入端口、ACL过滤到防火墙隔离，构建由内到外的四道接入防线）
#### 任务27 操作系统安全
##### 7.1 操作系统概述【p.180】
- 核心概念：操作系统是管理计算机硬件与软件资源、充当用户与硬件之间接口的系统软件。网络设备（路由器、交换机、防火墙）自身也运行专用操作系统，华为网络设备的通用操作系统为VRP（Versatile Routing Platform），其命令行界面即用户配置设备的入口。操作系统安全指通过身份认证、访问控制、补丁更新、服务最小化等手段，防止系统被未授权访问或恶意篡改。中职网络专业同时涉及Windows客户端安全与网络设备操作系统安全两条线。
- 关键命令/配置步骤：纯概念小节，无命令。演示要点：在eNSP中启动一台路由器，进入`<Huawei>`用户视图与`[Huawei]`系统视图，说明这就是VRP操作系统的CLI；对比Windows的图形界面，点明"网络设备用命令行管理"。
- 教学要点：生活类比——操作系统像小区物业，管着大门（接口）、水电（资源）和住户（进程），物业松懈外人就能随意进家。板书要点：操作系统定义→两大作用（资源管理、用户接口）→华为VRP。课堂演示：`display version`查看VRP版本号，让学生直观看到"设备也有操作系统"。
- 答辩角度：
  一、问：华为网络设备操作系统叫什么？答：VRP通用路由平台，VRP版本号可用`display version`查看。
  二、问：操作系统安全包含哪几个层面？答：身份认证、访问控制、补丁更新、服务最小化、日志审计。
  前后关联：承接前面模块的设备基础操作，为7.2操作系统安全配置做铺垫。
##### 7.2 操作系统安全配置【p.182】
- 核心概念：操作系统安全配置即按安全基线加固系统。Windows侧主要包括设置强密码与账户锁定策略、关闭默认共享、开启系统防火墙、及时打补丁、禁用不必要服务与端口、设置屏保恢复密码。网络设备侧通过AAA认证、限制Telnet/SSH接入、配置console与vty登录密码、关闭闲置端口实现。目标是缩小攻击面，让"默认状态"不再是"危险状态"。
- 关键命令/配置步骤：
```text
### Windows 侧常用加固（管理员命令提示符/PowerShell）
net share                       # 查看默认共享
net share ipc$ /del             # 删除IPC$默认共享（重启后恢复，需组策略固化）
net share admin$ /del
netsh advfirewall set allprofiles state on   # 开启所有配置文件的防火墙
net start                      # 查看正在运行的服务
sc config telnet start= disabled ; net stop telnet   # 禁用Telnet服务
### 华为设备侧登录加固（VRP）
system-view
user-interface console 0
 authentication-mode password
 set authentication password cipher YourPass@123
user-interface vty 0 4
 authentication-mode aaa
 protocol inbound ssh          # 只允许SSH，禁止明文Telnet
```
- 教学要点：生活类比——安全配置像给家门换锁、装门禁、关掉多余的侧门，而不是换一栋楼。板书要点：Windows基线四件事（强密码、关共享、开防火墙、禁服务）；设备基线两句话（console加密、vty走SSH）。课堂演示：先`net share`看到默认共享，删除后再看；设备上配置vty只允许SSH。
- 答辩角度：
  一、问：为什么禁用Telnet改用SSH？答：Telnet明文传输口令，SSH加密，防止嗅探。
  二、问：Windows默认共享有哪些风险？答：ipc$/admin$等默认共享可被用于远程枚举与攻击，应关闭或通过组策略固化。
  三、问：账户锁定策略解决什么问题？答：防暴力破解，连续失败N次锁定账户。
  前后关联：承接7.1操作系统概述，开启模块7安全接入话题，与7.3端口安全构成"主机—接入点"两道防线。
#### 任务28 交换机端口安全
##### 7.3 交换机端口安全【p.187】
- 核心概念：交换机端口安全（Port Security）工作在数据链路层，通过限定接入端口允许学习的MAC地址数量与具体MAC，防止非法设备私自接入内网、防范MAC地址泛洪攻击。当端口收到超出限定数量或不在白名单中的MAC时，设备按预设保护动作（protect丢弃、restrict告警并丢弃、shutdown/error-down关闭端口）处理。它是接入层防止"随便插根网线就能上网"的第一道防线。
- 关键命令/配置步骤：纯概念小节，无命令。演示要点：先在eNSP中让一台交换机端口学习多个MAC，再说明一旦开启端口安全并限定MAC数，超出即被限制。
- 教学要点：生活类比——端口安全像小区单元门的门禁，只让登记过的住户进，没登记的刷不开；登记人数超过上限就报警或锁门。板书要点：作用层（数据链路层）→三要素（最大MAC数、绑定MAC、保护动作）。课堂演示：用`display mac-address`对比开启端口安全前后的MAC表变化。
- 答辩角度：
  一、问：端口安全能防什么攻击？答：防私接设备、防MAC泛洪（CAM表溢出）攻击。
  二、问：三种保护动作区别？答：protect仅丢包不告警；restrict丢包并告警；shutdown直接关闭端口（error-down）。
  前后关联：承接7.2主机侧安全，转向网络接入点安全，为7.4实验做概念铺垫。
##### 7.4 端口安全加固实验【p.187】
- 核心概念：本实验在接入交换机端口上开启端口安全，限定最大MAC数为1并启用 sticky（ sticky MAC，将动态学习到的MAC固化为安全MAC），把端口与合法终端网卡绑定。当其他设备接入该端口时，触发保护动作。通过`display port-security`与`display mac-address`验证配置生效。
- 关键命令/配置步骤：
```text
system-view
sysname SW-ACCESS
vlan batch 10
interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 10
 port-security enable                # 开启端口安全
 port-security max-mac-num 1        # 只允许1个MAC
 port-security mac-address sticky    # 固化动态MAC为安全MAC
 port-security protect-action restrict   # 超限丢包并告警
quit
display port-security               # 验证端口安全配置
display mac-address                 # 查看安全MAC表
```
- 教学要点：生活类比——sticky MAC像把合法住户的指纹录入门禁，下次不用重录，换个人就刷不开。板书要点：四步走（开端口安全→限MAC数→绑sticky→设动作）。课堂演示：eNSP中把PCA接到G0/0/1，`display mac-address`看到学到PCA的MAC；再换PCB接入，观察报文被丢、告警产生。
- 答辩角度：
  一、问：max-mac-num设1意味着什么？答：该端口只允许一台设备一个MAC接入，再接第二台就触发保护。
  二、问：sticky MAC与静态绑定区别？答：sticky把动态学到的MAC自动固化保存，静态需手工敲`port-security mac-address xxx`，sticky更省事。
  三、问：触发shutdown后端口如何恢复？答：先`shutdown`再`undo shutdown`，或配置error-down自动恢复。
  前后关联：承接7.3概念，输出可复现实验，与7.5 ACL在"接入控制"思路上呼应。
#### 任务29 ACL访问控制列表
##### 7.5 访问控制列表【p.191】
- 核心概念：访问控制列表ACL是路由器/交换机上按规则对数据包进行过滤的集合，通过匹配源地址、目的地址、端口等条件执行permit（允许）或deny（拒绝）。基本ACL编号2000—2999，只匹配源IP，因其粒度粗、常靠近目的端部署；高级ACL编号3000—3999，匹配五元组，可靠近源端部署。ACL必须绑定到接口的某个方向（inbound/outbound）才生效。它解决"谁能访问谁、哪种流量能过"的过滤问题。
- 关键命令/配置步骤：纯概念小节，无命令。演示要点：用一张规则表类比——每条rule是一条"通行证"，从上到下匹配，命中即停，末尾隐含deny any。
- 教学要点：生活类比——ACL像机场安检的规则清单，先查身份证（源地址）再查违禁品，符合的放行、不符合的拦下，查到第一条匹配的就执行。板书要点：分类（基本2000+、高级3000+）+匹配顺序+绑定方向。课堂演示：口头给出"拒绝192.168.1.2访问任何地方"，让学生判断该用基本ACL。
- 答辩角度：
  一、问：基本ACL与高级ACL部署位置为何不同？答：基本ACL只能匹配源IP，若放靠近源端会误拦，故靠近目的端；高级ACL粒度细，靠近源端即可精准拦截。
  二、问：ACL末尾默认动作是什么？答：隐含deny any，未匹配的流量全部拒绝。
  前后关联：承接端口安全（二层接入控制），上升到三层流量过滤，为7.6基本ACL实验做准备。
##### 7.6 基本ACL配置实验【p.192】
- 核心概念：本实验通过基本ACL 2000拒绝指定源主机192.168.1.2访问网络，其余主机放行。规则编号决定匹配顺序（rule 5先于rule 10），通配符掩码0表示精确匹配一台主机。最后用`traffic-filter`在接口入方向绑定ACL生效。
- 关键命令/配置步骤：
```text
system-view
acl number 2000
 rule 5 deny source 192.168.1.2 0     # 拒绝主机192.168.1.2（通配符0=精确匹配）
 rule 10 permit source any             # 其余全部放行
quit
interface GigabitEthernet0/0/2
 traffic-filter inbound acl 2000       # 在该接口入方向绑定ACL
quit
display acl 2000                      # 验证规则与命中计数
```
- 教学要点：生活类比——规则编号像排队叫号，5号窗口先处理，10号后处理；排在前面的先命中。板书要点：三步（建ACL→写规则→接口绑方向）。课堂演示：eNSP中PCA(192.168.1.2)ping网关失败、PCB成功；`display acl 2000`看到rule 5有命中计数。
- 答辩角度：
  一、问：通配符掩码0与0.0.0.0含义？答：0即0.0.0.0，表示精确匹配该主机，等同`source host`。
  二、问：traffic-filter的inbound与outbound区别？答：inbound是进入接口的流量先过滤再转发，outbound是出接口前过滤；基本ACL一般绑靠近目的端的inbound。
  三、问：为什么permit要放deny后面？答：规则按编号顺序匹配，若permit any在前则后面deny永远不命中。
  前后关联：承接7.5 ACL概念，落地为基本ACL实验；与7.7/7.8防火墙在"流量过滤"思路上对比——ACL是设备内嵌过滤，防火墙是专业隔离设备。
#### 任务30 防火墙
##### 7.7 防火墙【p.196】
- 核心概念：防火墙是部署在不同安全区域之间、按安全策略对流经流量进行访问控制的网络安全设备。华为USG防火墙将接口划入安全区域，常见trust（内网，安全级别85）、untrust（外网，级别5）、dmz（服务区，级别50）。区域间默认"高安全区域可访问低安全区域，反向默认拒绝"。通过域间安全策略（security-policy）、NAT、入侵防御等实现网络隔离与防护。它解决"内外网之间精细化受控互通"的问题。
- 关键命令/配置步骤：纯概念小节，无命令。演示要点：画出trust—防火墙—untrust—dmz四区域拓扑，标注安全级别数值，说明默认访问方向。
- 教学要点：生活类比——防火墙像城市的城墙与城门：内网是城里（trust），外网是城外（untrust），DMZ是城外的使馆区（放对外服务器），城门按通行证放行。板书要点：三区域（trust/untrust/dmz）+安全级别+域间策略。课堂演示：说明"内网用户主动上网放行、外网主动连内网默认拒绝"。
- 答辩角度：
  一、问：DMZ区放什么服务器？答：对外提供服务的服务器，如Web、邮件，即使被攻破也不波及内网。
  二、问：防火墙与ACL区别？答：ACL是路由器/交换机内嵌的包过滤，防火墙是独立设备，具备安全区域、状态检测、NAT、UTM等综合能力。
  前后关联：承接7.6设备内嵌ACL，升级为专业安全设备，为7.8防火墙综合配置实验做概念铺垫。
##### 7.8 防火墙综合配置实验【p.197】
- 核心概念：本实验将防火墙G1/0/0划入trust、G1/0/1划入untrust，通过security-policy编写域间策略，允许trust访问untrust、拒绝untrust主动访问内网。核心是"先划区域、再写策略"，接口必须先入区域才参与域间匹配。具体NAT与教材细节（教材细节待核）以USG通行CLI为准。
- 关键命令/配置步骤：
```text
system-view
sysname USG6000
interface GigabitEthernet1/0/0
 ip address 192.168.1.1 24
quit
firewall zone trust
 add interface GigabitEthernet1/0/0
quit
interface GigabitEthernet1/0/1
 ip address 202.103.0.1 24
quit
firewall zone untrust
 add interface GigabitEthernet1/0/1
quit
security-policy
 rule name trust_to_untrust
  source-zone trust
  destination-zone untrust
  action permit
 rule name untrust_to_trust
  source-zone untrust
  destination-zone trust
  action deny
quit
display security-policy rule all
```
- 教学要点：生活类比——先给每条通道"划分归属"（接口进区域），再定"谁能走哪条道"（写策略）；顺序不能反。板书要点：两步（划区域→写策略）+动作permit/deny。课堂演示：trust内主机ping通untrust外网口，untrust侧主机主动ping trust口被拒；`display security-policy rule all`看命中计数。
- 答辩角度：
  一、问：接口不加入安全区域会怎样？答：接口属于local区域，默认不参与域间策略匹配，业务流量不通。
  二、问：安全策略与包过滤差异？答：安全策略是基于区域+五元组的集中策略，支持状态检测，比传统packet-filter更精细。
  三、问：内网用户要上外网还需配什么？答：通常还需NAT转换私网地址为公网地址（教材细节待核）。
  前后关联：承接7.7防火墙概念，综合区域+策略+NAT，是模块7安全接入的收口。
