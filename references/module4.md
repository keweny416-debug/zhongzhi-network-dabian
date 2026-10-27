### 模块4 交换机高级配置（从二层交换走向三层转发，实现VLAN间互通、地址自动分配与网络高可用）
#### 任务15 三层交换机
##### 4.1 网络拓扑结构【p.093】
- **核心概念**：园区网典型拓扑为"核心层—汇聚层—接入层"三层结构。接入层直接连接终端PC；汇聚层汇聚接入流量、实施策略与VLAN间路由；核心层高速转发。传统方案中VLAN间互通靠外置路由器（单臂路由），但带宽受限、成瓶颈；三层交换机把路由功能集成到交换机内部，用硬件ASIC线速转发，解决单臂路由性能问题。
- **关键命令/配置步骤**：无命令（纯概念小节）。演示要点：eNSP搭"接入交换机—三层核心交换机—路由器出口"拓扑，标出VLAN10/VLAN20接入位置。
- **教学要点**：生活类比——像公司组织架构：基层员工（接入PC）→部门主管（汇聚）→总经理（核心），决策在高层、执行在基层。板书要点：核心/汇聚/接入三层；单臂路由瓶颈；三层交换线速解决互通。课堂演示：画拓扑图标注各层设备与VLAN走向。
- **答辩角度**：一、单臂路由为何是瓶颈？答：所有VLAN间流量挤在路由器一个物理接口的子接口上，带宽受限。二、三层结构优点？答：分层设计、易扩展、故障隔离。**前后关联**：为4.2三层交换机原理、4.4 VLAN间通信提供组网背景。
##### 4.2 三层交换机概念与原理【p.095】
- **核心概念**：三层交换机=二层交换+三层路由一体化设备，能根据IP地址在VLAN之间高速转发数据包。其核心原理是"一次路由、多次交换"：第一个跨VLAN数据包由CPU做路由处理并把"源IP:源端口→目的IP:目的端口"的转发条目缓存到硬件TCAM表，后续同流数据包直接由ASIC硬件查表转发，不再经过CPU，从而达到线速。实现上靠VLANIF（VLAN接口）充当各VLAN的网关。
- **关键命令/配置步骤**：无命令（纯概念小节，基础配置预览）。
  ```
  [SW] interface Vlanif 10
  [SW-Vlanif10] ip address 192.168.10.1 24    # VLAN10的网关
  ```
- **教学要点**：生活类比——像"第一次问路靠人脑算（CPU路由），算完画个导航存在硬件里，以后同款路线秒过（ASIC交换）"。板书要点：二层交换+三层路由、一次路由多次交换、TCAM硬件缓存、VLANIF作网关。课堂演示：`display ip interface brief`看到Vlanif10/Vlanif20接口。
- **答辩角度**：一、三层交换机如何实现VLAN间路由？答：为每个VLAN建VLANIF接口配IP作网关，跨VLAN包由VLANIF转发。二、"一次路由多次交换"好处？答：首包后硬件线速转发，解决路由器软件转发慢的问题。**前后关联**：紧接4.1拓扑，是4.4 VLAN间通信的原理支撑，与3.x传统路由器转发形成对比。
##### 4.3 特点【p.095】
- **核心概念**：三层交换机主要特点：一、高速度硬件转发（ASIC/TCAM线速）；二、高带宽，支持大量千兆/万兆端口；三、VLAN间路由一次路由多次交换，解决单臂路由瓶颈；四、集交换与路由于一体，省去外置路由器；五、支持动态路由协议（OSPF等）但弱于专用路由器的广域网/VPN能力。适用于园区网核心/汇聚，不适合做广域网出口。
- **关键命令/配置步骤**：无命令（纯概念小节）。
- **教学要点**：生活类比——三层交换机像"既能装又快的城市立交桥"，市内各方向（VLAN）直接高速转换，不必都开去城外（路由器）绕。板书要点：硬件线速、高带宽、VLAN间路由、省路由器、广域能力弱。课堂演示：对比表——三层交换机 vs 传统路由器（转发性能/接口类型/适用位置）。
- **答辩角度**：一、三层交换机能当出口路由器吗？答：简单NAT可做，但广域网接口/VPN能力弱，出口仍建议专用路由器。二、与二层交换机本质区别？答：三层有IP路由功能和VLANIF网关，二层只能同VLAN转发。**前后关联**：巩固4.2原理，指导4.4组网选型，呼应3.3路由器种类。
#### 任务16 不同VLAN间通信
##### 4.4 实现不同VLAN间通信【p.097】
- **核心概念**：默认情况下不同VLAN二层隔离、不能互通，需通过三层设备做VLAN间路由实现通信。用三层交换机实现时，为每个VLAN创建VLANIF接口并配IP，把该IP作为对应VLAN内PC的默认网关；PC把跨VLAN的包发给本网关，三层交换机查路由后转发到目标VLANIF。前提：各接入端口已正确划分VLAN、Trunk链路允许相关VLAN通过。
- **关键命令/配置步骤**：
  ```
  [SW] vlan 10
  [SW-vlan10] quit
  [SW] interface Vlanif 10
  [SW-Vlanif10] ip address 192.168.10.1 24
  [SW] interface Vlanif 20
  [SW-Vlanif20] ip address 192.168.20.1 24
  # 接入端口划入VLAN（示例）
  [SW] interface GigabitEthernet0/0/1
  [SW-GigabitEthernet0/0/1] port link-type access
  [SW-GigabitEthernet0/0/1] port default vlan 10
  [SW] display ip interface brief      # 验证VLANIF Up
  ```
  PC1(192.168.10.10/24,网关192.168.10.1) 与 PC2(192.168.20.10/24,网关192.168.20.1)互ping验证。
- **教学要点**：生活类比——VLAN像两栋隔开的大楼，三层交换机是"楼下连廊检票口（网关）"，想串楼得在检票口登记转发。板书要点：VLANIF作各VLAN网关、PC网关指向本网关IP、Trunk放行VLAN。课堂演示：两PC不同VLAN先ping不通，配VLANIF后通，再`display arp`看网关MAC。
- **答辩角度**：一、PC网关为什么要设成VLANIF的IP？答：跨VLAN包必须先送到本网关，由三层设备转发。二、VLANIF接口什么状态算Up？答：该VLAN内至少有一个Access端口Up且VLAN已创建。**前后关联**：依赖模块2的VLAN划分与Trunk，承接4.2三层交换原理，是4.6 DHCP、4.8 VRRP的承载对象。
#### 任务17 DHCP
##### 4.5 DHCP基本原理【p.100】
- **核心概念**：DHCP（动态主机配置协议）自动为终端分配IP地址、子网掩码、网关、DNS等参数，免去手工逐台配置，适合主机多、流动性大的网络。分配过程为经典四步：发现（Discover广播）→提供（Offer）→请求（Request）→确认（ACK），即DORA。服务器从地址池中选一个空闲IP租约给主机，租约到期前可续租。解决的问题是"IP地址统一管理、避免冲突、提高效率"。
- **关键命令/配置步骤**：无命令（纯概念小节）。演示要点：用报文抓包/eNSP解释DORA四步广播与单播。
- **教学要点**：生活类比——DHCP像"酒店前台自动登记发房卡"，客人（PC）到店广播问"有空房吗"，前台（服务器）给间房（IP）并发饭卡（网关/DNS）。板书要点：DORA四步、地址池、租约、自动分配IP/掩码/网关/DNS。课堂演示：PC设为DHCP自动获取后观察获取到的IP。
- **答辩角度**：一、DORA四步各是什么？答：Discover发现、Offer提供、Request请求、ACK确认。二、DHCP优点？答：免手工、防IP冲突、便于统一管理。**前后关联**：为4.6交换机作DHCP服务器、4.7跨VLAN DHCP中继奠定原理，依赖4.4的VLANIF网关。
##### 4.6 配置交换机DHCP【p.101】
- **核心概念**：在三层交换机上启用DHCP功能后，可作为DHCP服务器为各VLAN自动分配地址。两种地址分配方式：接口地址池（dhcp select interface，以VLANIF接口所在网段为地址池，适合单VLAN简单场景）和全局地址池（ip pool，可统一定义network/gateway-list/dns-list，多VLAN复用）。必须先`dhcp enable`全局开启。
- **关键命令/配置步骤**：
  ```
  [SW] dhcp enable                       # 全局开启DHCP
  # 方式一：接口地址池
  [SW] interface Vlanif 10
  [SW-Vlanif10] dhcp select interface    # 用接口网段作地址池
  [SW-Vlanif10] dhcp server dns-list 114.114.114.114
  # 方式二：全局地址池
  [SW] ip pool vlan10
  [SW-ip-pool-vlan10] network 192.168.10.0 mask 24
  [SW-ip-pool-vlan10] gateway-list 192.168.10.1
  [SW-ip-pool-vlan10] dns-list 114.114.114.114
  [SW-ip-pool-vlan10] quit
  [SW] interface Vlanif 10
  [SW-Vlanif10] dhcp select global       # 引用全局地址池
  ```
  验证：PC设自动获取，`ipconfig`查看获得的IP/网关/DNS。
- **教学要点**：生活类比——接口池像"楼层吧台直接发本楼层房卡"，全局池像"前台统一房本统一登记发卡"。板书要点：dhcp enable、dhcp select interface/global、ip池network/gateway-list/dns-list。课堂演示：PC自动获取到192.168.10.x并能跨VLAN通。
- **答辩角度**：一、dhcp select interface与global区别？答：interface用接口所在网段作池，global引用命名地址池更灵活可多VLAN复用。二、不配dhcp enable会怎样？答：DHCP功能不生效，PC获取不到地址。**前后关联**：基于4.4的VLANIF网关，为4.7中继（服务器不在同网段）做准备。
##### 4.7 DHCP中继【p.104】
- **核心概念**：DHCP的Discover是广播包，路由器/三层交换机默认不跨VLAN转发广播，导致其他VLAN的请求到不了集中部署的DHCP服务器。DHCP中继（Relay）解决"DHCP服务器与客户端不在同一网段"的问题：客户端网关VLANIF收到广播请求后，把它转成单播转发给指定DHCP服务器IP，并把服务器回应转回对应VLAN。这样一台服务器可服务多个VLAN。
- **关键命令/配置步骤**：
  ```
  [SW] dhcp enable
  # 在客户端所在VLANIF上配置中继，指向DHCP服务器IP
  [SW] interface Vlanif 20
  [SW-Vlanif20] dhcp select relay             # 本接口用中继模式
  [SW-Vlanif20] dhcp relay server-ip 192.168.10.10   # DHCP服务器地址
  # 服务器侧需有对应VLAN20网段的全局地址池（network 192.168.20.0/24等）
  ```
- **教学要点**：生活类比——客户（PC）在自己楼层大喊问房，本楼没有前台（服务器），楼管（中继）把喊话翻译成电话，打到楼下总台（服务器），再把答复传回来。板书要点：广播不跨VLAN、dhcp select relay、dhcp relay server-ip、服务器侧配对应网段地址池。课堂演示：服务器只在VLAN10，VLAN20客户端配中继后成功获取192.168.20.x地址。
- **答辩角度**：一、为什么需要DHCP中继？答：DHCP请求是广播，三层设备不转发广播，跨VLAN必须由中继转单播。二、中继配在哪个设备？答：配在客户端网关所在三层设备的对应VLANIF上。**前后关联**：是4.6的跨网段延伸，与4.4 VLANIF、3.9静态路由配合使跨网段可达。
#### 任务18 冗余与生成树
##### 4.8 VRRP【p.107】
- **核心概念**：VRRP（虚拟路由冗余协议）把多台路由器/三层交换机虚拟成一台"虚拟路由器"，对外提供一个虚拟IP作为PC的统一网关。正常时由主设备（Master）负责转发，备份设备（Backup）待命；当Master故障，Backup自动接管虚拟IP，PC网关不用改，实现网关冗余、避免单点故障。通过VRID标识一个备份组，优先级决定谁当Master（默认100，越大越优）。
- **关键命令/配置步骤**：无命令（纯概念小节，配置见4.9）。
- **教学要点**：生活类比——VRRP像"正副店长"：日常正店长（Master）接待，正店长突然不在，副店长（Backup）立刻顶上去，客人（PC）找的还是同一个店门牌号（虚拟网关IP）。板书要点：虚拟网关IP、VRID备份组、Master/Backup、优先级选举、故障自动切换。课堂演示：两台三层交换机做VRRP，PC网关设为虚拟IP。
- **答辩角度**：一、VRRP解决什么问题？答：网关单点故障，主挂备接、业务不中断。二、虚拟IP与物理IP关系？答：虚拟IP是PC网关，可与某接口IP相同或独立，Master持有它。**前后关联**：基于4.4多网关/VLANIF场景，与4.10生成树共同构成高可用双核心组网。
##### 4.9 配置VRRP【p.108】
- **核心概念**：在承担网关的三层交换机VLANIF接口上配置VRRP备份组，指定虚拟IP；通过调整优先级让指定设备当Master。主设备配较高优先级，备用保持默认100。配置后`display vrrp`查看状态（Master/Backup）与虚拟IP。主设备故障时备用自动升为Master，配合上行链路检测（track）可实现更精细切换。
- **关键命令/配置步骤**：
  ```
  # Master（优先级调高）
  [SW-Master] interface Vlanif 10
  [SW-Master-Vlanif10] ip address 192.168.10.2 24
  [SW-Master-Vlanif10] vrrp vrid 1 virtual-ip 192.168.10.254
  [SW-Master-Vlanif10] vrrp vrid 1 priority 120
  # Backup（默认优先级100）
  [SW-Backup] interface Vlanif 10
  [SW-Backup-Vlanif10] ip address 192.168.10.3 24
  [SW-Backup-Vlanif10] vrrp vrid 1 virtual-ip 192.168.10.254
  [SW-Master] display vrrp                 # 查看Master/Backup状态
  ```
- **教学要点**：生活类比——给正副店长都挂同一个店招牌（虚拟IP），正店长权重高（priority 120）当值，副店长等接班。板书要点：vrrp vrid 1 virtual-ip、priority调高为主、display vrrp看角色。课堂演示：关掉Master的VLANIF，观察Backup切换为Master，PC长ping不掉线。
- **答辩角度**：一、两台VRRP设备优先级相同会怎样？答：比较接口IP大小，IP大者当选Master。二、虚拟IP必须与接口IP同网段吗？答：是，必须在该VLANIF所在网段内。**前后关联**：落地4.8原理，与4.10生成树解决"双核心链路冗余+防环路"配套。
##### 4.10 认识生成树协议【p.111】
- **核心概念**：交换网络为提高可靠性常做冗余链路（双核心），但二层冗余会形成环路，引发广播风暴、MAC表动荡、帧重复拷贝。生成树协议STP通过在逻辑上阻塞某些端口，把环形拓扑修剪成无环的树型拓扑；当活动链路故障时，被阻塞端口自动放开恢复连通。过程：选根桥（Bridge ID最小）→每非根桥选根端口→每网段选指定端口→其余端口阻塞。
- **关键命令/配置步骤**：无命令（纯概念小节）。验证命令预览：
  ```
  [SW] display stp          # 查看根桥、端口角色与状态
  ```
- **教学要点**：生活类比——STP像"立交桥交警把多余路口先封掉只留一条主路，主路堵了再开备用口"，既不堵成环又有后路。板书要点：解决二层环路/广播风暴、逻辑阻塞冗余口、选举根桥→根端口→指定端口→阻塞、故障自动恢复。课堂演示：eNSP把两交换机用两线连起来，`display stp`看到一个口被Discarding阻塞。
- **答辩角度**：一、二层环路会导致什么后果？答：广播风暴、MAC地址表反复震荡、帧重复接收。二、STP如何防环？答：选举根桥后把既非根端口也非指定端口的端口逻辑阻塞。**前后关联**：与4.8 VRRP共同支撑双核心高可用，为4.11/4.12/4.13三种STP配置铺垫。
#### 任务19 生成树配置
##### 4.11 配置生成树【p.115】
- **核心概念**：华为交换机默认已开启MSTP模式，可按需改为经典STP。通过修改桥优先级（stp priority）可人为指定谁当根桥（数值越小越优先，默认32768），或用`stp root primary/secondary`一键指定根。配置后用`display stp brief`查看各端口角色（ROOT/DESIGNATED/ALTERNATE）与状态（FORWARDING/DISCARDING）。
- **关键命令/配置步骤**：
  ```
  [SW1] stp enable
  [SW1] stp mode stp                 # 设置为经典STP
  [SW1] stp priority 4096           # 调小优先级，争取当根桥
  # 或等价：[SW1] stp root primary
  [SW1] display stp brief            # 查看端口角色与状态
  ```
- **教学要点**：生活类比——像"把最有经验的老店长（priority最小）定为总店，其他店听它调度路口"。板书要点：stp enable、stp mode stp、stp priority/root primary、display stp brief看角色。课堂演示：把SW1优先级改小后`display stp`确认它成为根桥，阻塞口位置随之变化。
- **答辩角度**：一、priority取值特点？答：必须为4096的倍数，越小越优先当根。二、根桥位置该放哪？答：放在网络中心/核心，减少次优路径。**前后关联**：落地4.10选举原理，是4.12 RSTP、4.13 MSTP的配置基线。
##### 4.12 配置快速生成树【p.119】
- **核心概念**：RSTP（快速生成树，IEEE 802.1w）在STP基础上实现更快收敛，端口状态简化、引入边缘端口（Edge Port）、P/A快速协商机制，链路故障后秒级收敛而非原STP的约30~50秒。接终端的端口设为边缘端口后直接进入转发、不参与STP计算，提升终端接入速度。
- **关键命令/配置步骤**：
  ```
  [SW1] stp enable
  [SW1] stp mode rstp                # 改为RSTP模式
  [SW1] interface GigabitEthernet0/0/1     # 接PC的接入端口
  [SW1-GigabitEthernet0/0/1] stp edged-port enable   # 设为边缘端口
  [SW1] display stp brief
  ```
- **教学要点**：生活类比——STP像"堵车后等好几个红灯才放行"，RSTP像"交警一看情况直接开绿波快速放行"。板书要点：stp mode rstp、收敛快、边缘端口edged-port直接转发。课堂演示：对比STP与RSTP下断链后恢复转发的快慢（display stp看状态翻转）。
- **答辩角度**：一、RSTP比STP快在哪？答：P/A协商+边缘端口，省去监听/学习等待，收敛秒级。二、边缘端口接错了会怎样？答：边缘端口再接交换机会临时参与STP，故应只接终端。**前后关联**：是4.11的优化升级，为4.13多实例MSTP的多VLAN负载分担做过渡。
##### 4.13 配置多实例生成树【p.121】
- **核心概念**：MSTP（多实例生成树，IEEE 802.1s）把多个VLAN映射到一个生成树实例（Instance），不同实例可阻塞不同端口，从而实现VLAN间流量负载分担——部分VLAN走左路、部分走右路，链路都被利用且无环。需先进入域配置视图（stp region-configuration），配置域名、实例与VLAN映射，再`active region-configuration`生效。
- **关键命令/配置步骤**：
  ```
  [SW1] stp mode mstp
  [SW1] stp region-configuration
  [SW1-mst-region] region-name MYDOMAIN
  [SW1-mst-region] instance 1 vlan 10        # VLAN10映射到实例1
  [SW1-mst-region] instance 2 vlan 20        # VLAN20映射到实例2
  [SW1-mst-region] active region-configuration   # 生效配置
  [SW1-mst-region] quit
  [SW1] stp instance 1 root primary          # 实例1以SW1为根
  [SW1] display stp instance 1 brief        # 查看实例1端口状态
  ```
  注意：同一MST域内所有交换机的region-name、VLAN-Instance映射、修订号必须一致。
- **教学要点**：生活类比——MSTP像"给不同部门发不同行车路线图：VLAN10走东门、VLAN20走西门"，两条路都不堵、都利用起来。板书要点：stp mode mstp、region-configuration、instance n vlan n映射、active生效、多实例负载分担。课堂演示：实例1根在SW1、实例2根在SW2，观察两VLAN流量走不同上行链路。
- **答辩角度**：一、MSTP相比RSTP优势？答：按实例多生成树，实现VLAN级负载分担，链路不浪费。二、域配置不生效常见原因？答：忘了active region-configuration，或域内各交换机参数不一致。**前后关联**：是STP家族的高级形态，综合运用4.4 VLAN划分、4.10防环原理，形成园区高可用冗余组网的收官。

