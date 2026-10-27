### 模块9 综合实训（把VLAN、三层交换、路由、DHCP与无线接入整合成一张可运行的园区网）
#### 任务33 部署三层局域网
##### 9.1 部署三层局域网【页码待核：在线无公开页码】
- 核心概念：三层局域网通过三层交换机的VLANIF（VLAN接口）实现不同VLAN间路由，替代传统单臂路由，消除单臂路由的带宽瓶颈。完整部署流程：规划VLAN与网段→接入端口划入VLAN→创建VLANIF并配置IP作网关→开启DHCP自动分配地址→配置静态路由上联出口。它综合了VLAN划分、VLAN间路由、DHCP与静态路由四大前序知识点。
- 关键命令/配置步骤：
```text
system-view
vlan batch 10 20                       # 批量创建VLAN
interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 10                  # 接入端口划入VLAN10
quit
interface Vlanif10                      # 创建VLAN10的三层网关
 ip address 192.168.10.1 24
 dhcp select interface                # 基于接口地址池自动分配
quit
interface Vlanif20
 ip address 192.168.20.1 24
 dhcp select interface
quit
ip route-static 0.0.0.0 0.0.0.0 192.168.100.254   # 默认路由上联出口
dhcp enable                            # 全局开启DHCP
display ip interface brief            # 查看各VLANIF状态
display ip routing-table              # 查看路由表
```
- 教学要点：生活类比——三层交换像立交桥，不同VLAN（不同路）在三层交换机上直接互通，不用绕回单臂路由器那个"转盘"。板书要点：五步规划表（VLAN号→网段→端口→VLANIF→DHCP）+静态路由一行。课堂演示：eNSP中两台PC分属VLAN10/VLAN20，配完后互相ping通，再`display ip routing-table`看到直连路由。
- 答辩角度：
  一、问：VLANIF与单臂路由优劣？答：VLANIF在交换机内部三层转发，无单臂路由的单链路瓶颈，性能更高、拓扑更简洁。
  二、问：DHCP select interface含义？答：以VLANIF自身IP为网段地址池网关，自动给该VLAN主机分配同网段地址。
  三、问：不同VLAN主机互通前提？答：VLANIF网关配置正确、主机默认网关指向VLANIF IP、全局dhcp enable。
  前后关联：综合全书VLAN、三层交换、DHCP、静态路由；与9.2无线扩展构成有线+无线完整园区网。
#### 任务34 扩展无线网络
##### 9.2 扩展无线网络【页码待核：在线无公开页码】
- 核心概念：无线扩展在有线园区网基础上叠加无线接入，主流为AC+Fit AP架构（AC集中管理、AP瘦 AP即插即用），小型场景可用FAT AP（胖AP独立工作）。部署要点：规划SSID与对应VLAN、配置AP上线、绑定安全模板与SSID模板、让无线流量汇入有线三层网互通。它解决"最后一百米"移动终端覆盖问题，是9.1有线三层网的无线延伸。
- 关键命令/配置步骤：纯概念/流程小节，无完整命令（教材细节待核）。流程要点：①规划SSID名称与映射VLAN；②AP接入有线网并向AC注册上线；③AC上配置SSID模板、安全模板（加密方式）并绑定到AP组；④无线客户端关联SSID后由DHCP获地址，与有线VLAN三层互通。
- 教学要点：生活类比——AC+Fit AP像"总部+连锁店"：AC是总部统一发指令，AP是各分店只干活不拿主意；FAT AP像夫妻店，自己管自己。板书要点：两架构（AC+Fit / FAT）+四步流程（规划SSID→AP上线→绑模板→互通验证）。课堂演示：eNSP中用AC+AP拓扑，无线SSID发出信号，笔记本关联后ping通有线PC。
- 答辩角度：
  一、问：Fit AP与FAT AP区别？答：Fit AP本身不配置，由AC集中管理，便于大规模部署；FAT AP独立配置，适合小型场景。
  二、问：无线SSID与VLAN关系？答：一个SSID通常对应一个VLAN，无线流量标记对应VLAN后进入有线三层网，实现与有线端同域互通。
  三、问：无线接入还需考虑什么安全？答：加密（WPA2/WPA3）、密码、隐藏SSID、隔离非法AP。
  前后关联：承接9.1有线三层局域网，完成"有线+无线"园区网闭环；与模块7无线侧接入安全呼应。

---
