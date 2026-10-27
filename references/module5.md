### 模块5 路由器高级配置（在静态路由/默认路由基础上，引入动态路由协议、广域网链路协议与地址转换，解决多路由器组网与跨网访问问题）
#### 任务20 RIP协议
##### 5.1 RIP概念【p.129】
- 核心概念：RIP（Routing Information Protocol，路由信息协议）是一种典型的内部网关协议（IGP），属于距离矢量（Distance Vector）路由协议。它以"跳数（Hop Count）"作为唯一度量值，每经过一台路由器跳数加1，最大有效跳数为15，跳数达到16即视为不可达。RIP路由器周期性（默认30秒）向邻居广播整张路由表，依靠"传闻"学习路由。它配置简单、开销小，适用于小型、拓扑简单的网络；缺点是收敛慢、易产生路由环路、规模受限。
- 关键命令/配置步骤：本小节为概念课，无命令。演示/流程要点：在eNSP中用三台路由器串成线性拓扑，不配置任何动态协议，观察每台路由器路由表中只有直连网段、互访不通；再引入RIP后路由条目自动出现，导入"为什么需要动态路由"这一课题。
- 教学要点：生活类比——把RIP比作小区里每户人家只问对门"你知道去火车站怎么走吗"，逐级口口相传，路太远（超过15户接力）就传话失败。板书要点：IGP/EGP定位、距离矢量、跳数度量、15跳上限、周期广播。课堂演示：eNSP三台路由器环形或线形拓扑，展示"无协议时ping不通→配RIP后通"的对比。
- 答辩角度：一、RIP为什么最大15跳？答：16跳表示不可达，用于抑制环路与计数到无穷问题。二、RIP v1与v2区别？答：v1有类不发掩码，v2无类携带掩码并支持组播更新。前后关联：承接本书前面静态路由章节，引出5.2 RIP配置。
##### 5.2 RIP配置【p.129】
- 核心概念：RIP配置是在路由器上启用RIP进程、指定版本并宣告直连主类网段，使相邻路由器之间自动交换路由条目。宣告network时RIP只填写主类网络地址（不写掩码），属于该主类的所有参与接口都会启用RIP。版本必须统一为v2才能正确传递无类路由信息，并建议关闭自动汇总以支持子网路由。
- 关键命令/配置步骤：
```
<Huawei> system-view
[Huawei] sysname R1
[R1] rip 1                       // 启动RIP进程，进程号本地有效
[R1-rip-1] version 2             // 使用RIP v2
[R1-rip-1] network 192.168.1.0   // 宣告直连主类网络
[R1-rip-1] network 192.168.2.0
[R1-rip-1] quit
[R1] quit
<R1> display rip 1               // 查看RIP进程信息
<R1> display ip routing-table protocol rip   // 查看RIP学习到的路由
```
- 教学要点：生活类比——network宣告相当于"报名参加小区居委会"，只有登记过的门牌（直连网段）才会被邻居听到。板书要点：rip进程号、version 2、network主类网段、验证命令display。课堂演示：eNSP中R1、R2、R3依次配置，在R1上display ip routing-table protocol rip观察学到对端网段；故意把两台路由器版本一为v1一为v2，观察路由学不到，引出版本一致性。
- 答辩角度：一、network后面为什么不写掩码？答：RIP v1/v2均按主类网络宣告，接口只要属于该主类即参与。二、RIP路由表中标记字母是什么？答：路由协议标识为R。前后关联：上承5.1 RIP概念，下接5.3 OSPF，形成距离矢量与链路状态的对比。
#### 任务21 OSPF协议
##### 5.3 OSPF【p.132】
- 核心概念：OSPF（Open Shortest Path First，开放式最短路径优先）是一种链路状态（Link State）内部网关协议。每台路由器生成描述自身链路状态的LSA（链路状态通告），向本区域内所有路由器泛洪，各自维护相同的链路状态数据库（LSDB），再用SPF（Dijkstra）算法以自己为根计算最短路径树。其度量为开销（Cost，带宽相关），无跳数限制，收敛快，支持区域分层（Area 0骨干区为核心），用组播地址224.0.0.5、224.0.0.6交换报文。适合中大型企业网。
- 关键命令/配置步骤：本小节为概念课，无命令。演示/流程要点：用"通讯录共享"思想讲解——每户都把自己周围的路牌贴到公告栏，全员拿到相同地图后各自画最短路线；对比RIP只听对门一家之言。
- 教学要点：生活类比——OSPF像城市规划局把每条道路的路况公布在公共信息牌上，所有司机拿到同一张地图再各自选最短路径，而不是只问前面那个人。板书要点：链路状态、LSDB、SPF算法、Cost度量、Area 0骨干、组播更新、无跳数限制。课堂演示：eNSP中展示OSPF邻居建立过程，用display ospf peer看邻居状态由Init到Full。
- 答辩角度：一、OSPF为什么快于RIP收敛？答：链路状态泛洪+SPF计算，不依赖逐跳传闻。二、OSPF中Area 0的作用？答：骨干区域，所有非骨干区域必须直连它，避免区域间环路。前后关联：与5.1 RIP做横向对比，引出5.4 OSPF配置。
##### 5.4 OSPF配置【p.133】
- 核心概念：OSPF配置核心是启动OSPF进程、指定Router ID、在对应区域内用"网段+反掩码（wildcard）"宣告参与接口。与RIP不同，OSPF的network后面写的是目的网段加反掩码（如0.0.0.255），并明确划入area区域号。配置完成后相邻路由器先建立邻居关系，再同步链路状态数据库，最终生成路由表。
- 关键命令/配置步骤：
```
<Huawei> system-view
[Huawei] sysname R1
[R1] router-id 1.1.1.1
[R1] ospf 1 router-id 1.1.1.1
[R1-ospf-1] area 0
[R1-ospf-1-area-0.0.0.0] network 192.168.1.0 0.0.0.255
[R1-ospf-1-area-0.0.0.0] network 192.168.2.0 0.0.0.255
[R1-ospf-1-area-0.0.0.0] quit
[R1-ospf-1] quit
[R1] quit
<R1> display ospf peer brief          // 查看OSPF邻居
<R1> display ospf routing             // 查看OSPF路由表
<R1> display ip routing-table protocol ospf
```
- 教学要点：生活类比——network加反掩码像"圈地登记"，精确圈出哪些接口、归到哪个区域。板书要点：ospf进程号、router-id、area 0、network网段+反掩码、Full邻居状态。课堂演示：eNSP中两台路由器同区域配OSPF，display ospf peer观察Full状态；故意把一端划入area 1、另一端area 0且不直连骨干，观察邻居起不来。
- 答辩角度：一、反掩码和子网掩码关系？答：反掩码=255.255.255.255减去子网掩码，0位表示需精确匹配。二、邻居停在Init/2-Way/ExStart是什么意思？答：状态未到Full说明参数不一致或区域不匹配。前后关联：上承5.3 OSPF原理，下接5.5路由重分发（多协议共存时必须引入）。
##### 5.5 路由重分发【p.136】
- 核心概念：路由重分发（Route Redistribution）指在一台同时运行多种路由协议（或静态/直连）的路由器上，把一种协议学到的路由"注入"到另一种协议中。不同路由协议度量值体系不同（RIP看跳数、OSPF看开销、静态是手工指定），不能直接互通，必须在ASBR（自治系统边界路由器）上用import-route命令引入，并按需设置默认开销。它解决企业网"既有老RIP网段、又新建OSPF网段"的混合组网互通问题。
- 关键命令/配置步骤：本小节为概念课，无命令。演示/流程要点：在eNSP中左半区跑RIP、右半区跑OSPF，中间一台路由器双协议同时跑，观察两侧路由表互不可见，从而引出"必须重分发"。
- 教学要点：生活类比——路由重分发像翻译官，在说方言（RIP）和说普通话（OSPF）的两个村子之间传话，两边度量口径不同要换算。板书要点：ASBR、import-route、度量不可直接互通、双向重分发。课堂演示：eNSP观察未重分发时单侧路由表缺项，重分发后双方都学到对方网段。
- 答辩角度：一、为什么重分发要注意防止环路？答：双点双向重分发可能产生次优路径和环路，需用路由策略控制。二、重分发静态路由用什么命令？答：import-route static。前后关联：上承5.2 RIP与5.4 OSPF两种协议，引出5.6重分发配置实操。
##### 5.6 重分发配置【p.136】
- 核心概念：在同时运行RIP与OSPF的ASBR上，把RIP路由引入OSPF、把OSPF路由引入RIP，使两个协议域相互学习到对方网段。注意OSPF引入外部路由后默认标记为OSPF外部路由（Type 2），路由表中以"O_ASE"呈现；RIP引入外部路由跳数按默认值传递。重分发通常需双向配置才能实现两端全网可达。
- 关键命令/配置步骤：
```
[R1] ospf 1
[R1-ospf-1] import-route rip 1          // 把RIP路由引入OSPF
[R1-ospf-1] quit
[R1] rip 1
[R1-rip-1] import-route ospf 1           // 把OSPF路由引入RIP
[R1-rip-1] import-route direct            // 引入直连（按需）
[R1-rip-1] quit
<R1> display ip routing-table             // 查看O_ASE与R标记路由
<R1> display ospf routing
```
- 教学要点：生活类比——翻译官要两头都传话，只翻一个方向对方还是听不懂，所以要双向import-route。板书要点：在哪个协议视图下import-route对方、O_ASE外部路由、双向重分发、验证命令。课堂演示：eNSP先只在OSPF里引入RIP，观察OSPF域通而RIP域不通；再补反向引入，全网全通。
- 答辩角度：一、路由表中O_ASE是什么？答：OSPF引入的外部路由。二、重分发后ping不通先查什么？答：先查是否双向都做了import-route，再查ACL/路由策略是否过滤。前后关联：直接承接5.5路由重分发原理，是RIP与OSPF两个任务的综合应用点。
#### 任务22 PPP协议
##### 5.7 PPP【p.140】
- 核心概念：PPP（Point-to-Point Protocol，点对点协议）是工作在广域网串行链路上的数据链路层协议，用于取代早期的SLIP。PPP由三部分组成：封装方法（在串行帧上封装网络层数据）、LCP（链路控制协议，负责建立/配置/检测数据链路）、NCP（网络控制协议族，如IPCP协商IP地址）。PPP支持PAP、CHAP两种认证方式，提供比HDLC更安全、可协商、可认证的点到点连接，广泛用于专线、拨号、路由器串口互联。
- 关键命令/配置步骤：本小节为概念课，无命令。演示/流程要点：在eNSP中用两台路由器Serial串口背靠背相连，默认封装为HDLC，改为PPP后再做认证演示。
- 教学要点：生活类比——PPP像两家之间修了一条专用电话线，通话前先"对上暗号（认证）"，再谈正事（传IP数据）。板书要点：数据链路层、串行链路、LCP/NCP、PAP/CHAP认证、替代SLIP。课堂演示：eNSP串口两端，display interface Serial1/0/0看封装协议，把link-protocol从hdlc改成ppp。
- 答辩角度：一、PPP和HDLC区别？答：PPP支持认证与协商、跨厂商互通，HDLC是Cisco扩展私有且无认证。二、PPP三层结构？答：封装、LCP、NCP。前后关联：承接广域网串行链路知识，引出5.8 PAP与5.9 CHAP两种认证。
##### 5.8 PAP【p.141】
- 核心概念：PAP（Password Authentication Protocol，密码认证协议）是PPP的一种两次握手认证方式。被认证方（主叫端）以明文方式反复向认证方发送用户名和密码，直到认证通过或失败。PAP认证过程中用户名密码在链路上明文传输，安全性较低，适用于对安全要求不高或老式拨号环境。
- 关键命令/配置步骤：
```
// 认证方（服务端）配置
[R1] interface Serial1/0/0
[R1-Serial1/0/0] link-protocol ppp
[R1-Serial1/0/0] ppp authentication-mode pap
[R1-Serial1/0/0] quit
[R1] aaa
[R1-aaa] local-user huawei password cipher Huawei@123
[R1-aaa] local-user huawei service-type ppp
[R1-aaa] quit
// 被认证方（客户端）配置
[R2] interface Serial1/0/0
[R2-Serial1/0/0] link-protocol ppp
[R2-Serial1/0/0] ppp pap local-user huawei password cipher Huawei@123
[R2-Serial1/0/0] quit
<R2> display ppp all
```
- 教学要点：生活类比——PAP像直接把账号密码写在纸条上递给门卫，路上谁都能看到，所以不安全。板书要点：两次握手、明文传输、服务端aaa建用户、客户端ppp pap local-user。课堂演示：eNSP串口两端配PAP，把客户端密码故意写错，观察链路起不来；改回正确后通。
- 答辩角度：一、PAP为什么不安全？答：用户名密码明文在链路上传输，易被抓包窃取。二、PAP是几次握手？答：两次握手，由被认证方主动发送。前后关联：上承5.7 PPP，下接5.9 CHAP做安全对比。
##### 5.9 CHAP【p.144】
- 核心概念：CHAP（Challenge Handshake Authentication Protocol，挑战握手认证协议）是PPP的三次握手认证方式。认证方先向对端发送一个随机挑战报文（Challenge），被认证端用共享密码对挑战报文做哈希运算后回送结果，认证方用相同算法比对。整个过程中密码本身不在链路上传输，只传输哈希摘要，安全性远高于PAP。
- 关键命令/配置步骤：
```
// 认证方（服务端）配置
[R1] interface Serial1/0/0
[R1-Serial1/0/0] link-protocol ppp
[R1-Serial1/0/0] ppp authentication-mode chap
[R1-Serial1/0/0] quit
[R1] aaa
[R1-aaa] local-user huawei password cipher Huawei@123
[R1-aaa] local-user huawei service-type ppp
[R1-aaa] quit
// 被认证方（客户端）配置
[R2] interface Serial1/0/0
[R2-Serial1/0/0] link-protocol ppp
[R2-Serial1/0/0] ppp chap user huawei
[R2-Serial1/0/0] ppp chap password cipher Huawei@123
[R2-Serial1/0/0] quit
<R2> display ppp all
```
- 教学要点：生活类比——CHAP像门卫出一道随机数学题，你用只有双方知道的"算法+密码"算出答案交回去，门卫自己也算一遍对答案，全程不暴露密码本身。板书要点：三次握手、挑战/响应、哈希摘要不传输密码、ppp chap user与ppp chap password。课堂演示：eNSP对比PAP与CHAP，同拓扑分别配置，说明生产环境默认选CHAP。
- 答辩角度：一、CHAP与PAP核心区别？答：CHAP三次握手、不传明文密码；PAP两次握手、明文传输。二、CHAP中随机挑战报文作用？答：防止重放攻击，每次挑战值不同。前后关联：承接5.8 PAP对比收尾PPP任务，自然过渡到模块5最后NAT。
#### 任务23 NAT
##### 5.10 了解NAT【p.147】
- 核心概念：NAT（Network Address Translation，网络地址转换）是在路由器出口处把内网私有IP地址转换为合法公网IP地址的技术。它要解决两个问题：一是缓解IPv4公网地址枯竭，让多台内网主机共享少量公网地址上网；二是隐藏内网拓扑结构，对外网屏蔽内网主机真实地址，提升安全性。常见类型有静态NAT（一对一固定映射）、动态NAT（地址池动态分配）、NAPT/PAT（多对多端口复用，即Easy IP/Overload）以及NAT Server（端口映射，把内网服务器发布到外网）。
- 关键命令/配置步骤：本小节为概念课，无命令。演示/流程要点：用eNSP搭"内网PC—出口路由器—公网路由器—服务器"拓扑，先不配NAT时内网私有地址ping不通公网，引出地址转换需求。
- 教学要点：生活类比——NAT像学校传达室，所有学生（内网私有IP）外出都登记成同一个传达室电话（公网IP），外面的人只看到传达室，不知道具体是哪个学生。板书要点：私有地址转公网、静态/动态/NAPT/NAT Server、缓解IPv4枯竭、隐藏内网。课堂演示：eNSP展示内网192.168.x.x访问公网前不通，配置NAT后通。
- 答辩角度：一、NAT带来什么优点？答：节省公网地址、隐藏内网、增强安全。二、NAT的缺点？答：增加延迟、破坏端到端可达性、对ICMP/部分应用层协议需ALG。前后关联：是模块5收束小节，引出5.11静态NAT实验与5.12对外发布服务器。
##### 5.11 静态NAT实验【p.148】
- 核心概念：静态NAT是建立一对一的固定地址映射，把某个内网私有IP永久绑定到某个公网IP。适用于需要固定对外地址的服务器或特定主机。配置方法是在连接公网的出接口上用nat static命令指定global公网地址与inside内网地址的对应关系。配置后内网主机访问外网时源地址被替换为公网地址，外部访问该公网地址也会被转发到对应内网主机。
- 关键命令/配置步骤：
```
[R1] interface GigabitEthernet0/0/0            // 连接公网的接口
[R1-GigabitEthernet0/0/0] nat static global 200.1.1.10 inside 192.168.1.10
[R1-GigabitEthernet0/0/0] quit
<R1> display nat static                         // 查看静态NAT映射
<R1> display nat session all                    // 查看NAT转换会话表
```
- 教学要点：生活类比——静态NAT像给某一位学生分配一个专属的传达室固定分机号，谁打这个分机都转给他本人。板书要点：出接口下nat static global公网 inside内网、一对一固定映射、display nat static/session。课堂演示：eNSP在内网PC访问公网服务器，在出口路由器display nat session all观察源IP被转换成200.1.1.10。
- 答辩角度：一、nat static配置在哪个接口？答：配置在连接外网（公网）的出接口上。二、静态NAT与动态NAT区别？答：静态手工一对一固定，动态从地址池自动分配。前后关联：上承5.10 NAT概念，下接5.12带端口映射的NAT Server+Easy IP综合实验。
##### 5.12 配置NAT实现外网访问网站服务器【p.149】
- 核心概念：本实验综合两类NAT：一是用NAPT/Easy IP（nat outbound配合ACL）让内网多台主机共享出接口公网地址访问外网；二是用NAT Server（nat server）把内网的网站服务器发布到公网，使外网用户能通过公网地址访问内网服务器的WWW服务。ACL用于定义哪些内网网段允许被转换，NAT Server负责外部到内部的目的地址映射。
- 关键命令/配置步骤：
```
[R1] acl number 2001
[R1-acl-basic-2001] rule 5 permit source 192.168.1.0 0.0.0.255
[R1-acl-basic-2001] quit
[R1] interface GigabitEthernet0/0/0            // 公网出接口
[R1-GigabitEthernet0/0/0] nat outbound 2001    // 内网多主机共享出接口地址上网
[R1-GigabitEthernet0/0/0] nat server protocol tcp global 200.1.1.10 www inside 192.168.1.10 www
[R1-GigabitEthernet0/0/0] quit
<R1> display nat outbound                      // 查看NAPT配置
<R1> display nat server                        // 查看服务器映射
<R1> display nat session all                   // 查看转换会话
```
- 教学要点：生活类比——nat outbound像传达室给所有外出学生统一换牌；nat server像在外公告栏留一个"对外窗口电话"，外面人打进来转接到办公室（服务器）。板书要点：ACL定义感兴趣流、nat outbound做源转换、nat server做目的端口映射、WWW服务对应TCP 80。课堂演示：eNSP中外网PC用浏览器访问http://200.1.1.10能打开内网网站，同时内网PC也能访问外网，display nat session观察两条方向转换。
- 答辩角度：一、nat outbound和nat server方向区别？答：outbound转换内网出方向源地址；server转换外网入方向目的地址，发布服务器。二、外网访问内网服务器先通不了查什么？答：查nat server地址/端口、查服务器网关与防火墙、查回程路由。前后关联：是模块5路由器高级配置的综合出口，结束后进入模块6无线网络。
