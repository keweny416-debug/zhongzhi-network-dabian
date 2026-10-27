### 模块2 交换机的配置与应用（从交换原理到VLAN划分与跨交换机VLAN贯通）
#### 任务6 交换及交换机
##### 2.1 交换及交换机【p.039】
- 核心概念：交换机（Switch）是工作在数据链路层（二层）的网络设备，它根据数据帧中的MAC地址转发数据，而不是像集线器那样把数据从所有端口广播出去。交换的本质是：交换机收到一个数据帧后，查看目的MAC地址，查自己的MAC地址表，只把帧从对应端口发出去，而不是泛洪到所有端口。这样显著减少了冲突、提升了带宽利用率。交换机把一个共享式网络变成多个端口独占带宽的交换式网络，是现代局域网的核心设备。
- 关键命令/配置步骤：本小节为概念，无命令。演示/要点：对比"集线器(广播所有端口)"与"交换机(按MAC定向转发)"的区别；引出交换机MAC地址表的作用。
```text
### 集线器 vs 交换机 对比要点（板书）
- 集线器Hub  ：共享带宽，一个口来数据所有口都发，易冲突
- 交换机Switch：独享带宽，按目的MAC只发对应端口
```
- 教学要点：生活类比——集线器像大喇叭喊话，全教室都听见；交换机像前台按快递单只把包裹递给收件人，不打扰别人。板书要点：二层设备、按MAC转发、MAC地址表、独享带宽。课堂演示：用eNSP连两台PC互ping，再在交换机上`display mac-address`看学到的MAC与端口对应关系。
- 答辩角度：一、"交换机和集线器本质区别？"——集线器物理层共享冲突域，交换机链路层按MAC定向转发、分割冲突域。二、"交换机工作在OSI哪一层？"——数据链路层（二层）。前后关联：是2.2工作原理与2.3交换方式的概念起点。
##### 2.2 交换机工作原理【p.040】
- 核心概念：交换机的工作原理可概括为"学习、转发、泛洪"三个动作。学习指：交换机收到数据帧时，把帧里的源MAC地址和接收端口记到MAC地址表中。转发指：若目的MAC已在表中，就从对应端口单播发出。泛洪（未知单播泛洪）指：若目的MAC不在表中，交换机向除入端口外的所有端口广播该帧，直到目标回应后再学习其位置。此外，广播帧和组播帧一律泛洪。正是这套机制让交换机能"越用越聪明"，逐步建立起MAC与端口的映射表。
- 关键命令/配置步骤：本小节为原理，无配置命令。演示/要点：用命令查看MAC地址表的建立过程。
```text
<Huawei> display mac-address          # 查看MAC地址表(MAC-端口-VLAN)
### 观察方法：先不ping时表为空，PC互ping后再查，可见动态学到的表项
```
- 教学要点：生活类比——交换机像新来的前台，谁来寄件先记下他在哪个门（学习），下次知道直接送哪个门（转发），不知道的人就问一圈（泛洪）。板书要点：源MAC学习→查表单播转发→查表失败泛洪。课堂演示：eNSP中先查mac-address为空，PC互ping后再查，让学生看到表项被动态"学"出来。
- 答辩角度：一、"目的MAC不在表里怎么办？"——泛洪到所有端口(除入端口)，目标应答后再学习。二、"MAC地址表如何老化？"——长时间无流量的表项会自动老化删除（默认300秒，教材具体数值待核）。前后关联：承接2.1交换概念，是2.3三种交换方式与2.10 VLAN的原理基础。
##### 2.3 三种交换方式【p.042】
- 核心概念：交换机转发数据帧有三种方式：一、直通交换（Cut-through）——只看完帧的目的MAC地址就立即转发，延迟小但不纠错；二、存储转发（Store-and-forward）——先把整个帧收下来校验无误（检查CRC）再转发，速度略慢但可靠；三、无碎片交换（Fragment-free）——介于两者之间，只先读取帧的前64字节再转发，可过滤大部分冲突碎片。现代交换机多以存储转发为主。三种方式的取舍本质是"快"与"准"的权衡。
- 关键命令/配置步骤：本小节为原理对比，无设备命令。演示/要点：列表对比三种方式的读取深度与优缺点。
```text
### 三种交换方式对比（板书表）
1. 直通交换      ：看目的MAC即发，延迟最低，不检错
2. 存储转发     ：收完整帧校验CRC后再发，可靠，延迟略高(主流)
3. 无碎片交换   ：读前64字节再发，折中
```
- 教学要点：生活类比——直通像快递员看一眼门牌就扔出去（快但可能扔错），存储转发像先拆开验货确认包裹完好再送（慢但稳妥），无碎片是只查最容易出问题的那一段。板书要点：三种方式名称、读取深度、优缺点。课堂演示：结合CRC校验说明"为什么会有错帧"，引导学生理解存储转发为何成为主流。
- 答辩角度：一、"哪种方式延迟最低？哪种最可靠？"——直通延迟最低，存储转发最可靠。二、"存储转发为什么慢？"——要等收完整帧并做CRC校验，再决定转发或丢弃。前后关联：承接2.2工作原理，属交换技术原理的横向对比。
##### 2.4 交换机指标【p.042】
- 核心概念：交换机的主要性能指标包括：一、背板带宽（交换容量）——交换机内部数据总线能同时处理的数据量，单位Gbps，越大越不易拥堵；二、包转发率（吞吐量）——每秒能转发的数据包数量，单位Mpps；三、端口数量与类型——电口/光口数量，决定接入能力；四、MAC地址表容量——能记忆多少个MAC地址；五、是否支持VLAN、三层路由等功能。选购或规划时，背板带宽与包转发率是判断交换机是否"线速转发"的关键。指标要与终端规模匹配，不能盲目追高。
- 关键命令/配置步骤：本小节为设备指标，无配置命令。可在设备上查看自身信息辅助理解：
```text
<Huawei> display version            # 查看设备型号、VRP版本、运行时间
<Huawei> display device             # 查看设备硬件信息(板卡、端口)
```
- 教学要点：生活类比——指标像汽车参数：背板带宽是马路宽度，包转发率是单位时间过车数量，端口数是车道数，路越宽车越快但也要按出行需求选。板书要点：背板带宽、包转发率、端口数、MAC容量、支持功能五项。课堂演示：用display version让学生认识真实设备型号，把抽象指标和实物对应。
- 答辩角度：一、"背板带宽和包转发率关系？"——背板带宽决定内部吞吐上限，包转发率决定实际每秒转发能力，两者都够才叫线速。二、"MAC地址表满了会怎样？"——新MAC无法学习，可能导致泛洪增多、性能下降。前后关联：承接1.2设备选择，是选购交换机的量化依据。
##### 2.5 交换机技术发展【p.043】
- 核心概念：交换机技术经历了从早期集线器式共享网络，到二层交换、三层交换、再到现代多层交换与SDN的演进。早期网络用集线器共享带宽，冲突严重；二层交换机实现按MAC转发、分割冲突域；三层交换机把路由功能集成进交换芯片，实现VLAN间高速转发；后续发展出支持QoS、堆叠、虚拟化、SDN（软件定义网络）等特性。了解发展脉络，有助于理解今天为什么需要VLAN、为什么需要三层交换，以及网络设备从"硬连线配置"走向"可编程集中控制"的趋势。
- 关键命令/配置步骤：本小节为技术史，无设备命令。演示/要点：按时间线梳理四代形态。
```text
### 交换机技术演进线（板书）
集线器(共享) → 二层交换(MAC转发) → 三层交换(集成路由) → 多层/SDN(可编程)
```
- 教学要点：生活类比——像交通从"大家挤一条土路"（集线器）到"分车道"（二层交换）到"立交桥自动分流"（三层交换）再到"智能红绿灯统一调度"（SDN）。板书要点：四代演进、每层解决什么问题。课堂演示：用一条时间线串起"为什么需要VLAN"，为模块2后半段做铺垫。
- 答辩角度：一、"二层和三层交换最大区别？"——三层交换带路由功能，能做VLAN间转发，二层只能同VLAN转发。二、"SDN核心思想？"——控制平面与数据平面分离，集中统一调度网络。前后关联：收束任务6，引出2.6开始的真实配置。
#### 任务7 交换机初始化配置及基本管理
##### 2.6 初始化配置及基本管理【p.045】
- 核心概念：新交换机开箱后是空白状态，需要先做初始化配置：设置设备名、配置管理IP（用于远程管理）、设置登录密码等，这就是初始化配置。二层交换机本身端口默认在VLAN1中，要远程管理需给VLANIF接口（管理VLAN）配IP。基本管理包括：修改主机名、配置Console与远程登录口令、保存配置、恢复出厂设置。掌握初始化配置，是让一台新设备从"裸机"变成"可管理、可配置"设备的第一步。
- 关键命令/配置步骤：交换机初始化配置完整序列如下：
```text
<Huawei> system-view
[Huawei] sysname SW1                       # 改名为SW1
[SW1] interface Vlanif 1                   # 进入管理VLAN接口(二层交换机管理口)
[SW1-Vlanif1] ip address 192.168.1.2 255.255.255.0
[SW1-Vlanif1] quit
[SW1] user-interface console 0             # 配置Console口登录
[SW1-ui-console0] authentication-mode password
[SW1-ui-console0] set authentication password cipher Admin@123
[SW1-ui-console0] quit
[SW1] quit
<SW1> save                                 # 保存配置
```
- 教学要点：生活类比——初始化配置像给新手机贴膜、设密码、连WiFi，先把名字、密码、管理地址设好，后面才好用。板书要点：改名、配管理IP(Vlanif)、设密码、save保存。课堂演示：eNSP拖一台新S5700，从零敲出sysname和管理IP，并用另一台PC telnet/同网段访问验证。
- 答辩角度：一、"二层交换机的管理IP配在哪里？"——配在Vlanif（管理VLAN接口）上，不是物理端口。二、"配完不save重启会怎样？"——配置丢失，回到出厂空白。三、reset saved-configuration后如何恢复出厂？——再reboot。前后关联：承接1.6/1.9安全与管理，是2.7 VLAN配置的准备动作。
#### 任务8 VLAN
##### 2.7 VLAN功能及作用【p.049】
- 核心概念：VLAN（Virtual Local Area Network，虚拟局域网）是在物理局域网基础上，通过交换机把一个物理网络逻辑划分成多个广播域的技术。它解决的核心问题是：默认情况下交换机所有端口都在VLAN1，一个广播帧会泛洪到全网，既浪费带宽又带来安全隐患。VLAN把不同部门的端口逻辑隔离，使它们即使接在同一台交换机上也互不通广播，必须经过三层设备才能互访。VLAN的作用包括：隔离广播域、增强网络安全、灵活按部门分组、简化管理变更。
- 关键命令/配置步骤：本小节为功能概念，无命令。演示/要点：对比"同一交换机默认全在VLAN1"与"划分VLAN后广播被隔离"。
```text
### VLAN作用要点（板书）
- 默认：所有端口属VLAN1，广播泛洪全网
- 划VLAN后：广播只在本VLAN内转发，不同VLAN二层隔离
- 不同VLAN互访 → 需三层交换机/路由器路由
```
- 教学要点：生活类比——VLAN像在同一栋写字楼里用虚拟隔墙把财务部、技术部隔开，大家虽然在一栋楼（同一台交换机），但不串门，要往来必须经过前台（三层路由）。板书要点：逻辑广播域隔离、安全、灵活分组、默认VLAN1。课堂演示：eNSP中不划VLAN时两台PC同交换机可互访，引出"为什么要隔离"的需求。
- 答辩角度：一、"VLAN解决了什么问题？"——分割广播域、隔离不同部门、提升安全与性能。二、"不同VLAN默认能通信吗？"——不能，二层隔离，必须三层路由。前后关联：是2.8实现方法与2.9命令的功能依据。
##### 2.8 VLAN实现方法【p.051】
- 核心概念：VLAN在交换机上的实现主要涉及两种端口链路类型：一、Access端口——通常连接终端（PC），只能属于一个VLAN，收发的是不带标签（untagged）的普通帧；二、Trunk端口——通常连接交换机与交换机（或交换机与路由器），可同时承载多个VLAN的流量，帧中带有802.1Q标签来区分属于哪个VLAN。实现VLAN的思路是：先在交换机上创建VLAN，再把接入端口划到对应VLAN（Access），交换机之间用Trunk放行这些VLAN，跨交换机时同一VLAN才能贯通。802.1Q是VLAN标签的国际标准协议。
- 关键命令/配置步骤：本小节为实现原理，核心命令预告如下：
```text
### VLAN实现的两条链路类型（概念+预告命令）
### Access：接PC，属单一VLAN，去标签
[SW1] interface GigabitEthernet0/0/1
[SW1-GigabitEthernet0/0/1] port link-type access
[SW1-GigabitEthernet0/0/1] port default vlan 10
### Trunk：交换机互联，带802.1Q标签，放行多VLAN
[SW1-GigabitEthernet0/0/1] port link-type trunk
[SW1-GigabitEthernet0/0/1] port trunk allow-pass vlan 10 20
```
- 教学要点：生活类比——Access口像单人单隔间的电话（只能一个部门用），Trunk口像多车道高速公路（在一条线上用标签同时跑多个部门的车），802.1Q标签就是车上的部门编号。板书要点：Access(接终端/单VLAN/无标签)、Trunk(交换机互联/多VLAN/带标签802.1Q)。课堂演示：在eNSP上先建VLAN再演示Access与Trunk的区别。
- 答辩角度：一、Access口和Trunk口用在哪？"——Access接终端，Trunk接交换机之间。二、"为什么交换机互联要用Trunk？"——一条链路要同时传多个VLAN的数据，必须靠标签区分。前后关联：承接2.7功能，是2.9命令与2.10/2.11配置的实现原理。
##### 2.9 常见VLAN命令【p.052】
- 核心概念：华为VRP中操作VLAN有一组固定命令：`vlan` 用于创建并进入VLAN视图，`vlan batch` 用于批量创建多个VLAN；在接口下用 `port link-type access/trunk` 设置端口类型，用 `port default vlan` 把Access口划入VLAN，用 `port trunk allow-pass vlan` 让Trunk口放行指定VLAN。查看命令有 `display vlan`（看VLAN与端口归属）、`display port vlan`（看端口链路类型）。掌握这组命令，是完成所有VLAN配置的基础工具集。
- 关键命令/配置步骤：VLAN常用命令全集如下：
```text
<SW1> system-view
[SW1] vlan 10                          # 创建VLAN10并进入其视图
[SW1-vlan10] quit
[SW1] vlan batch 20 30                 # 批量创建VLAN20、30
[SW1] interface GigabitEthernet0/0/1
[SW1-GigabitEthernet0/0/1] port link-type access
[SW1-GigabitEthernet0/0/1] port default vlan 10   # 接入VLAN10
[SW1-GigabitEthernet0/0/1] quit
### 查看验证：
<SW1> display vlan                     # 列出所有VLAN及所属端口
<SW1> display port vlan                # 查看端口的link-type与PVID
```
- 教学要点：生活类比——vlan 10像"开一个编号10的房间"，port default vlan 10像"把1号门牌号挂到10号房间上"，display vlan像查房间入住名单。板书要点：建VLAN(vlan/batch)、划端口(access+default vlan)、查(display vlan)。课堂演示：eNSP里敲完命令用display vlan现场对照，看端口是否出现在对应VLAN下。
- 答辩角度：一、"display vlan里端口前的U和T代表什么？"——U是Access(untagged)，T是Trunk(tagged)。二、"如何一次性删/建多个VLAN？"——用vlan batch。三、"undo vlan 10会怎样？"——删除VLAN10，其下端口会回到默认VLAN1。前后关联：是2.10基于端口划VLAN与2.11跨交换机VLAN的命令清单。
#### 任务9 基于端口划分VLAN
##### 2.10 基于端口划分VLAN步骤与方法【p.054】
- 核心概念：基于端口划分VLAN是最常用、最基础的VLAN划分方式：把交换机上的物理端口固定分配到某个VLAN，该端口接入的终端就属于这个VLAN。它的优点是配置简单、适合位置固定的办公网络；缺点是用户换端口后VLAN归属要重新配置。典型场景：把接财务的端口划到VLAN10，接技术部的端口划到VLAN20。整个过程分三步——建VLAN、把端口设为Access、把Access口加入指定VLAN，最后用display vlan验证。
- 关键命令/配置步骤：基于端口划分VLAN完整配置序列如下：
```text
<SW1> system-view
[SW1] vlan batch 10 20                 # 先创建VLAN10、VLAN20
[SW1] interface GigabitEthernet0/0/1
[SW1-GigabitEthernet0/0/1] port link-type access
[SW1-GigabitEthernet0/0/1] port default vlan 10    # 端口1 → VLAN10
[SW1-GigabitEthernet0/0/1] quit
[SW1] interface GigabitEthernet0/0/2
[SW1-GigabitEthernet0/0/2] port link-type access
[SW1-GigabitEthernet0/0/2] port default vlan 20    # 端口2 → VLAN20
[SW1-GigabitEthernet0/0/2] quit
[SW1] quit
<SW1> display vlan                     # 验证：端口1在VLAN10，端口2在VLAN20
```
```text
### 验证现象（PC配同网段不同VLAN）
### PC1(接0/0/1, VLAN10, 192.168.10.11) 与 PC2(接0/0/2, VLAN20, 192.168.20.11)
### 即使同接一台交换机，因VLAN不同也ping不通 → 证明VLAN隔离生效
```
- 教学要点：生活类比——像把教室座位按小组贴标签，1号座贴"财务组"、2号座贴"技术组"，坐在哪组就归哪组，换座位就要换标签。板书要点：三步走——建VLAN、设Access、划入VLAN；验证看display vlan和ping不通现象。课堂演示：eNSP把PC1划VLAN10、PC2划VLAN20，配不同网段IP，现场ping不通，再让同VLAN的两台ping通。
- 答辩角度：一、"为什么同交换机不同VLAN ping不通？"——VLAN是二层广播域隔离，交换机不把一个VLAN的帧转发到另一个VLAN。二、"基于端口划分的缺点？"——用户移动设备换端口需重新配置VLAN。三、"同VLAN两台PC为什么能通？"——它们同属一个广播域，交换机可按MAC单播转发。前后关联：承接2.9命令，引出2.11跨交换机时同VLAN如何贯通。
#### 任务10 跨交换机相同VLAN配置
##### 2.11 跨交换机相同VLAN配置方法【p.059】
- 核心概念：当同一部门分布在两台交换机上时，需要让两台交换机上同属VLAN10的主机能够跨交换机通信，这就是跨交换机相同VLAN配置。方法是：在两台交换机上都创建相同的VLAN，把各自接部门终端的端口设为Access并划入该VLAN，再把两台交换机相连的端口设为Trunk，并在Trunk上放行该VLAN。Trunk链路通过802.1Q标签携带不同VLAN的数据，从而实现"一条交换机互联线同时打通多个VLAN"。若只配Access而不配Trunk，跨交换机的同VLAN主机将无法通信。
- 关键命令/配置步骤：跨交换机相同VLAN完整配置（两台交换机SW1、SW2，中间用G0/0/1互联）：
```text
### ===== SW1 =====
<SW1> system-view
[SW1] vlan batch 10 20                       # 两台交换机VLAN编号要一致
[SW1] interface GigabitEthernet0/0/2           # 接财务PC的端口
[SW1-GigabitEthernet0/0/2] port link-type access
[SW1-GigabitEthernet0/0/2] port default vlan 10
[SW1-GigabitEthernet0/0/2] quit
[SW1] interface GigabitEthernet0/0/1           # 与SW2互联的Trunk口
[SW1-GigabitEthernet0/0/1] port link-type trunk
[SW1-GigabitEthernet0/0/1] port trunk allow-pass vlan 10 20
[SW1-GigabitEthernet0/0/1] quit
### ===== SW2（镜像配置，互联口同样设Trunk并放行相同VLAN）=====
<SW2> system-view
[SW2] vlan batch 10 20
[SW2] interface GigabitEthernet0/0/2
[SW2-GigabitEthernet0/0/2] port link-type access
[SW2-GigabitEthernet0/0/2] port default vlan 10
[SW2-GigabitEthernet0/0/2] quit
[SW2] interface GigabitEthernet0/0/1
[SW2-GigabitEthernet0/0/1] port link-type trunk
[SW2-GigabitEthernet0/0/1] port trunk allow-pass vlan 10 20
[SW2-GigabitEthernet0/0/1] quit
### 验证：
<SW1> display port vlan        # 确认互联口为trunk且放行vlan 10 20
### 同属VLAN10的两台PC(不同交换机)配同网段IP，可ping通；未放行的VLAN不通
```
- 教学要点：生活类比——两台楼之间要让同一条"财务专线"打通，楼里各自的房间(Access口)要对好编号，楼与楼之间的走廊(Trunk口)要明确允许财务和技术两条专线同时通过。板书要点：两端建相同VLAN→接PC口设Access→互联口设Trunk并allow-pass相同VLAN。课堂演示：eNSP搭两台交换机，各接一台VLAN10的PC，先不配Trunk看ping不通，再配Trunk看打通，突出Trunk的关键作用。
- 答辩角度：一、"跨交换机同VLAN不通，最常见漏配是什么？"——互联口没设成Trunk或没allow-pass对应VLAN。二、"Trunk默认放行哪些VLAN？"——默认只放行VLAN1，其他需手动allow-pass（教材具体默认行为待核）。三、"两端VLAN编号必须一致吗？"——跨交换机同VLAN通信，编号必须一致。前后关联：收束模块2，是后续三层交换实现VLAN间路由的直接前置。

