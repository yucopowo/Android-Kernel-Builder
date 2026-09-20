<div align = center>

<img src="./.assets/DogDayAndroid.png" width="200" height="175" alt="banner">

<h1>小米 8 (dipper) 构建 KernelSU</h1>

![License](https://img.shields.io/static/v1?label=License&message=BY-NC-SA&logo=creativecommons&color=green)
![Language](https://img.shields.io/github/languages/top/Xiaomi-sdm845-KSU/Android-Kernel-Builder)
![Issues](https://img.shields.io/github/issues/Xiaomi-sdm845-KSU/Android-Kernel-Builder)
![Pull Requests](https://img.shields.io/github/issues-pr/Xiaomi-sdm845-KSU/Android-Kernel-Builder)
<br>

这是个 Github Action 自动构建小米 8（dipper）KernelSU 内核的项目, 自动同步最新 KernelSU

包含: MI 8 (dipper)
<br>
</div>

---


# 内核选择


MIUI rom 建议选择 dipper-Xiaomi_Kernel_OpenSource-sdm845_构建时间.zip

如: 小米 8 MIUI 12.5 选择 dipper-Xiaomi_Kernel_OpenSource-sdm845_********.zip

类原生rom 建议选择 dipper-NGK_android_kernel_xiaomi_sdm845_构建时间.zip

如: 小米 8 类原生rom 选择 dipper-NGK_android_kernel_xiaomi_sdm845_********.zip

LineageOS rom 建议选择 dipper-android_kernel_xiaomi_sdm845_构建时间.zip

如: 小米 8 LineageOS 22.2 选择 dipper-android_kernel_xiaomi_sdm845_********.zip

# 项目结构

这份 `check-config.sh` 的脚本输出主要用于检查 Linux 内核是否满足运行 Docker 等容器技术的要求。

### 📋 内核配置参数逻辑意义表

| 参数名称 | 逻辑意义 |
| :--- | :--- |
| **cgroup hierarchy** | **控制组挂载点**。Docker 需要通过 cgroups 来限制和隔离进程组的资源（CPU、内存等）。 |
| **CONFIG_NAMESPACES** | **命名空间总开关**。用于实现进程隔离，使容器拥有独立的进程 ID、网络、文件系统等视图。 |
| **CONFIG_NET_NS** | **网络命名空间**。允许容器拥有独立的网络协议栈（网卡、IP、路由表等）。 |
| **CONFIG_PID_NS** | **进程命名空间**。允许容器拥有独立的进程 ID 空间，容器内的 PID 1 独立于宿主机。 |
| **CONFIG_IPC_NS** | **进程间通信命名空间**。隔离信号量、消息队列和共享内存，防止容器间 IPC 冲突。 |
| **CONFIG_UTS_NS** | **UTS 命名空间**。允许容器拥有独立的主机名和域名。 |
| **CONFIG_CGROUPS** | **控制组支持**。内核用于限制、记录和隔离进程组资源使用的核心机制。 |
| **CONFIG_CGROUP_CPUACCT** | **CPU 计费**。生成关于 cgroup 中任务使用的 CPU 资源的报告。 |
| **CONFIG_CGROUP_DEVICE** | **设备访问控制**。允许或拒绝 cgroup 中的任务对特定设备节点（如 /dev/sda）的访问。 |
| **CONFIG_CGROUP_FREEZER** | **进程冻结**。支持挂起（冻结）和恢复 cgroup 中的任务，常用于容器暂停/恢复功能。 |
| **CONFIG_CGROUP_SCHED** | **CPU 调度**。为 cgroup 提供 CPU 时间片的公平调度。 |
| **CONFIG_CPUSETS** | **CPU 集合**。将进程绑定到特定的 CPU 核心和内存节点上，用于 NUMA 亲和性设置。 |
| **CONFIG_MEMCG** | **内存控制器**。限制 cgroup 中进程可使用的内存总量，防止内存溢出影响宿主机。 |
| **CONFIG_KEYS** | **密钥保留服务**。用于在内核中存储密钥（如 DNS 解析密钥），供进程间共享。 |
| **CONFIG_VETH** | **虚拟以太网设备**。用于创建虚拟网线对，连接容器的网络命名空间与宿主机的网桥。 |
| **CONFIG_BRIDGE** | **802.1d 以太网桥接**。允许将多个网络接口连接在一起，实现容器与外部网络的通信。 |
| **CONFIG_BRIDGE_NETFILTER** | **网桥 netfilter**。允许 iptables 对经过网桥的数据包进行过滤和处理。 |
| **CONFIG_IP_NF_FILTER** | **IPv4 包过滤**。iptables 的核心模块，用于定义允许或拒绝数据包的规则。 |
| **CONFIG_IP_NF_MANGLE** | **IPv4 包修改**。用于修改数据包的特定字段（如 TTL、TOS）。 |
| **CONFIG_IP_NF_TARGET_MASQUERADE** | **IP 伪装**。一种特殊的 NAT，常用于动态 IP 环境（如拨号上网），实现容器上网。 |
| **CONFIG_IP6_NF_FILTER** | **IPv6 包过滤**。针对 IPv6 协议的 iptables 过滤模块。 |
| **CONFIG_IP6_NF_MANGLE** | **IPv6 包修改**。针对 IPv6 协议的 iptables 修改模块。 |
| **CONFIG_IP6_NF_TARGET_MASQUERADE** | **IPv6 伪装**。针对 IPv6 协议的 NAT 伪装功能。 |
| **CONFIG_NETFILTER_XT_MATCH_ADDRTYPE** | **地址类型匹配**。iptables 扩展，用于匹配数据包的源或目标地址类型（如本地、广播）。 |
| **CONFIG_NETFILTER_XT_MATCH_CONNTRACK** | **连接跟踪匹配**。允许 iptables 根据连接状态（如 ESTABLISHED, NEW）进行过滤。 |
| **CONFIG_NETFILTER_XT_MATCH_IPVS** | **IPVS 匹配**。用于匹配 IP 虚拟服务器（IPVS）的连接属性，常用于 Kubernetes Service。 |
| **CONFIG_NETFILTER_XT_MARK** | **包标记匹配**。用于匹配或设置数据包的 Netfilter 标记。 |
| **CONFIG_IP_NF_RAW** | **IPv4 原始表**。iptables 的 raw 表，用于在连接跟踪之前处理数据包（如 NOTRACK）。 |
| **CONFIG_IP_NF_NAT** | **IPv4 网络地址转换**。实现源地址转换（SNAT）和目标地址转换（DNAT）。 |
| **CONFIG_NF_NAT** | **Netfilter NAT 核心**。NAT 功能的通用核心层。 |
| **CONFIG_IP6_NF_RAW** | **IPv6 原始表**。针对 IPv6 的 raw 表支持。 |
| **CONFIG_IP6_NF_NAT** | **IPv6 网络地址转换**。针对 IPv6 的 NAT 功能。 |
| **CONFIG_POSIX_MQUEUE** | **POSIX 消息队列**。一种进程间通信机制，Docker 有时用于守护进程与容器间的通信。 |
| **CONFIG_NF_NAT_IPV4** | **IPv4 NAT 模块**。专门用于 IPv4 的 NAT 实现。 |
| **CONFIG_NF_NAT_NEEDED** | **NAT 需求标志**。指示系统是否需要 NAT 功能。 |
| **CONFIG_USER_NS** | **用户命名空间**。允许容器内的 root 用户映射为宿主机的普通用户，提高安全性。 |
| **CONFIG_SECCOMP** | **安全计算模式**。限制进程可调用的系统调用，减少内核攻击面。 |
| **CONFIG_SECCOMP_FILTER** | **Seccomp 过滤器**。允许基于参数精细过滤系统调用。 |
| **CONFIG_CGROUP_PIDS** | **进程数限制**。限制 cgroup 中可以创建的最大进程/线程数量，防止 Fork 炸弹。 |
| **CONFIG_MEMCG_SWAP** | **Swap 限制**。允许限制 cgroup 使用的 Swap 空间总量。 |
| **CONFIG_MEMCG_SWAP_ENABLED** | **Swap 限制默认开启**。表示 Swap 限制功能默认处于启用状态。 |
| **CONFIG_IOSCHED_CFQ** | **CFQ IO 调度器**。完全公平队列 IO 调度算法，用于管理磁盘 IO 请求。 |
| **CONFIG_CFQ_GROUP_IOSCHED** | **CFQ 组调度**。允许对 cgroup 进行 IO 带宽限制和权重分配。 |
| **CONFIG_BLK_CGROUP** | **块设备控制组**。通用块设备层的 cgroup 接口，用于 IO 限制。 |
| **CONFIG_BLK_DEV_THROTTLING** | **块设备限流**。限制块设备的读写速率（bps）和 IOPS。 |
| **CONFIG_CGROUP_PERF** | **性能监控**。允许 perf 工具对 cgroup 进行性能分析。 |
| **CONFIG_CGROUP_HUGETLB** | **大页内存控制**。限制 cgroup 使用的大页内存（HugeTLB）数量。 |
| **CONFIG_NET_CLS_CGROUP** | **网络分类器**。基于 cgroup 对网络数据包进行分类，用于 QoS。 |
| **CONFIG_CGROUP_NET_PRIO** | **网络优先级**。设置 cgroup 中进程产生的网络流量的优先级。 |
| **CONFIG_CFS_BANDWIDTH** | **CFS 带宽控制**。用于限制 cgroup 在完全公平调度器下的 CPU 使用配额（如 `--cpu-quota`）。 |
| **CONFIG_FAIR_GROUP_SCHED** | **公平组调度**。确保 CPU 时间在任务组（cgroup）之间公平分配。 |
| **CONFIG_IP_NF_TARGET_REDIRECT** | **重定向目标**。iptables 功能，将数据包重定向到本地端口（常用于透明代理）。 |
| **CONFIG_IP_SCTP** | **SCTP 协议支持**。流控制传输协议，某些应用容器可能需要。 |
| **CONFIG_IP_VS** | **IP 虚拟服务器**。Linux 内核的负载均衡技术，Kubernetes IPVS 模式依赖此功能。 |
| **CONFIG_IP_VS_NFCT** | **IPVS 连接跟踪**。IPVS 与 Netfilter 连接跟踪的集成。 |
| **CONFIG_IP_VS_PROTO_TCP/UDP** | **IPVS 协议支持**。IPVS 对 TCP 和 UDP 协议的支持。 |
| **CONFIG_IP_VS_RR** | **IPVS 轮询调度**。IPVS 的一种调度算法（Round Robin）。 |
| **CONFIG_SECURITY_SELINUX** | **SELinux 支持**。内核级强制访问控制安全模块。 |
| **CONFIG_SECURITY_APPARMOR** | **AppArmor 支持**。另一种内核级强制访问控制安全模块（Ubuntu 常用）。 |
| **CONFIG_NFT_CT** | **Nftables 连接跟踪**。Nftables（iptables 的后继者）的连接跟踪表达式支持。 |
| **CONFIG_NFT_FIB** | **Nftables FIB 查找**。Nftables 的转发信息库查找支持。 |
| **CONFIG_NFT_MASQ** | **Nftables 伪装**。Nftables 的 NAT 伪装支持。 |
| **CONFIG_NFT_NAT** | **Nftables NAT**。Nftables 的通用 NAT 支持。 |
| **CONFIG_NF_TABLES** | **Nftables 框架**。新的包过滤框架，用于替代 iptables。 |
| **CONFIG_EXT4_FS** | **Ext4 文件系统**。广泛使用的 Linux 日志文件系统。 |
| **CONFIG_EXT4_FS_POSIX_ACL** | **Ext4 ACL 支持**。访问控制列表，用于更精细的文件权限管理。 |
| **CONFIG_EXT4_FS_SECURITY** | **Ext4 安全属性**。支持扩展文件属性，用于存储 SELinux 标签等安全信息。 |
| **CONFIG_VXLAN** | **虚拟可扩展局域网**。用于创建覆盖网络，Docker Overlay 网络依赖此功能。 |
| **CONFIG_BRIDGE_VLAN_FILTERING** | **网桥 VLAN 过滤**。允许网桥根据 VLAN ID 过滤流量。 |
| **CONFIG_IPVLAN** | **IPvlan 驱动**。一种网络虚拟化驱动，允许多个虚拟接口共享一个物理接口的 MAC 地址。 |
| **CONFIG_MACVLAN** | **Macvlan 驱动**。允许为物理接口创建具有不同 MAC 地址的虚拟接口。 |
| **CONFIG_DUMMY** | **Dummy 驱动**。创建一个虚拟的“空”网络接口，常用于路由测试。 |
| **CONFIG_NF_NAT_FTP/TFTP** | **FTP/TFTP NAT 辅助**。帮助 NAT 正确处理 FTP 和 TFTP 这种动态端口的协议。 |
| **CONFIG_NF_CONNTRACK_FTP/TFTP** | **FTP/TFTP 连接跟踪**。帮助内核跟踪 FTP 和 TFTP 的连接状态。 |
| **CONFIG_BTRFS_FS** | **Btrfs 文件系统**。一种写时复制文件系统，Docker 曾用作存储驱动。 |
| **CONFIG_OVERLAY_FS** | **Overlay 文件系统**。联合文件系统，Docker 默认的存储驱动（Overlay2）依赖此功能。 |
| **/dev/zfs, zfs command** | **ZFS 支持**。ZFS 文件系统和相关命令，用于 ZFS 存储驱动。 |
| **/proc/sys/kernel/keys/root_maxkeys** | **最大密钥数限制**。系统允许的最大密钥数量，影响容器内密钥环的使用。 |



```shell
localhost:~# ./check-config.sh 
info: reading kernel config from /proc/config.gz ...

Generally Necessary:
- cgroup hierarchy: nonexistent??
    (see https://github.com/tianon/cgroupfs-mount)
- CONFIG_NAMESPACES: enabled
- CONFIG_NET_NS: enabled
- CONFIG_PID_NS: enabled
- CONFIG_IPC_NS: enabled
- CONFIG_UTS_NS: enabled
- CONFIG_CGROUPS: enabled
- CONFIG_CGROUP_CPUACCT: enabled
- CONFIG_CGROUP_DEVICE: enabled
- CONFIG_CGROUP_FREEZER: enabled
- CONFIG_CGROUP_SCHED: enabled
- CONFIG_CPUSETS: enabled
- CONFIG_MEMCG: enabled
- CONFIG_KEYS: enabled
- CONFIG_VETH: enabled
- CONFIG_BRIDGE: enabled
- CONFIG_BRIDGE_NETFILTER: enabled
- CONFIG_IP_NF_FILTER: enabled
- CONFIG_IP_NF_MANGLE: enabled
- CONFIG_IP_NF_TARGET_MASQUERADE: enabled
- CONFIG_IP6_NF_FILTER: enabled
- CONFIG_IP6_NF_MANGLE: enabled
- CONFIG_IP6_NF_TARGET_MASQUERADE: enabled
- CONFIG_NETFILTER_XT_MATCH_ADDRTYPE: enabled
- CONFIG_NETFILTER_XT_MATCH_CONNTRACK: enabled
- CONFIG_NETFILTER_XT_MATCH_IPVS: enabled
- CONFIG_NETFILTER_XT_MARK: enabled
- CONFIG_IP_NF_RAW: enabled
- CONFIG_IP_NF_NAT: enabled
- CONFIG_NF_NAT: enabled
- CONFIG_IP6_NF_RAW: enabled
- CONFIG_IP6_NF_NAT: enabled
- CONFIG_NF_NAT: enabled
- CONFIG_POSIX_MQUEUE: enabled
- CONFIG_NF_NAT_IPV4: enabled
- CONFIG_NF_NAT_NEEDED: enabled

Optional Features:
- CONFIG_USER_NS: enabled
- CONFIG_SECCOMP: enabled
- CONFIG_SECCOMP_FILTER: enabled
- CONFIG_CGROUP_PIDS: enabled
- CONFIG_MEMCG_SWAP: enabled
- CONFIG_MEMCG_SWAP_ENABLED: enabled
- CONFIG_IOSCHED_CFQ: enabled
- CONFIG_CFQ_GROUP_IOSCHED: enabled
- CONFIG_BLK_CGROUP: enabled
- CONFIG_BLK_DEV_THROTTLING: enabled
- CONFIG_CGROUP_PERF: enabled
- CONFIG_CGROUP_HUGETLB: missing
- CONFIG_NET_CLS_CGROUP: enabled
- CONFIG_CGROUP_NET_PRIO: enabled
- CONFIG_CFS_BANDWIDTH: missing
- CONFIG_FAIR_GROUP_SCHED: enabled
- CONFIG_IP_NF_TARGET_REDIRECT: enabled
- CONFIG_IP_SCTP: missing
- CONFIG_IP_VS: enabled
- CONFIG_IP_VS_NFCT: enabled
- CONFIG_IP_VS_PROTO_TCP: enabled
- CONFIG_IP_VS_PROTO_UDP: enabled
- CONFIG_IP_VS_RR: enabled
- CONFIG_SECURITY_SELINUX: enabled
- CONFIG_SECURITY_APPARMOR: missing
- CONFIG_NFT_CT: missing
- CONFIG_NFT_FIB_IPV4: missing
- CONFIG_NFT_FIB_IPV6: missing
- CONFIG_NFT_FIB: missing
- CONFIG_NFT_MASQ: missing
- CONFIG_NFT_NAT: missing
- CONFIG_NF_TABLES: missing
- CONFIG_EXT4_FS: enabled
- CONFIG_EXT4_FS_POSIX_ACL: enabled
- CONFIG_EXT4_FS_SECURITY: enabled
- Network Drivers:
  - "bridge":
    - sysctl net.ipv4.ip_forward: disabled
    - sysctl net.ipv6.conf.all.forwarding: disabled
    - sysctl net.ipv6.conf.default.forwarding: disabled
  - "overlay":
    - CONFIG_VXLAN: enabled
    - CONFIG_BRIDGE_VLAN_FILTERING: enabled
      Optional (for encrypted networks):
      - CONFIG_CRYPTO: enabled
      - CONFIG_CRYPTO_AEAD: enabled
      - CONFIG_CRYPTO_GCM: enabled
      - CONFIG_CRYPTO_SEQIV: enabled
      - CONFIG_CRYPTO_GHASH: enabled
      - CONFIG_XFRM: enabled
      - CONFIG_XFRM_USER: enabled
      - CONFIG_XFRM_ALGO: enabled
      - CONFIG_INET_ESP: enabled
      - CONFIG_NETFILTER_XT_MATCH_BPF: enabled
      - CONFIG_INET_XFRM_MODE_TRANSPORT: enabled
  - "ipvlan":
    - CONFIG_IPVLAN: enabled
  - "macvlan":
    - CONFIG_MACVLAN: enabled
    - CONFIG_DUMMY: enabled
  - "ftp,tftp client in container":
    - CONFIG_NF_NAT_FTP: enabled
    - CONFIG_NF_CONNTRACK_FTP: enabled
    - CONFIG_NF_NAT_TFTP: enabled
    - CONFIG_NF_CONNTRACK_TFTP: enabled
- Storage Drivers:
  - "btrfs":
    - CONFIG_BTRFS_FS: missing
    - CONFIG_BTRFS_FS_POSIX_ACL: missing
  - "overlay":
    - CONFIG_OVERLAY_FS: enabled
  - "zfs":
    - /dev/zfs: missing
    - zfs command: missing
    - zpool command: missing

Limits:
- /proc/sys/kernel/keys/root_maxkeys: 1000000


```
---


# 项目结构


| 文件 | 说明 |
| --- | --- |
| `repos.dipper-MIUI.json` | MIUI 内核配置（Xiaomi_Kernel_OpenSource-sdm845, 88-zstd 分支） |
| `repos.dipper-NGK.json` | 类原生内核配置（NGK_android_kernel_xiaomi_sdm845, t-caf-ksu 分支） |
| `repos.dipper-LineageOS.json` | LineageOS 内核配置（LineageOS/android_kernel_xiaomi_sdm845, lineage-22.2 分支） |
| `.github/workflows/build KernelSU_v0.9.5.yml` | 按 `repos*.json` 全量构建 |
| `.github/workflows/build_MIU-Kernel.yml` | 只构建 MIUI 内核（读 `repos*-MIUI.json`） |
| `.github/workflows/build_NGK-Kernel.yml` | 只构建 NGK 内核（读 `repos*-NGK.json`） |
| `.github/workflows/build_LineageOS-Kernel.yml` | 只构建 LineageOS 内核（读 `repos*-LineageOS.json`） |
| `.github/workflows/del.yml` | 清理旧的 workflow 运行记录 |

> **关于 `defconfigFragments`**：LineageOS 内核的基础 defconfig（`vendor/xiaomi/mi845_defconfig`）
> 并不包含触摸屏、指纹、GPS 等机型专属项，这些放在 `arch/arm64/configs/vendor/xiaomi/dipper.config`
> 里，必须额外合并，否则会编出一个「能编译但刷进手机屏幕指纹全废」的内核。
> 所以在 `repos.dipper-LineageOS.json` 的 **`kernelSource` 里**用 `defconfigFragments` 声明该片段
> （workflow 读的是 `matrix.repos.kernelSource.defconfigFragments`），
> workflow 会在 defconfig 之后自动执行 `scripts/kconfig/merge_config.sh` + `olddefconfig`。
> 没有该字段的机型配置行为完全不变。
>
> 该片段里含 `CONFIG_MACH_XIAOMI_SDM845=y` / `CONFIG_MACH_XIAOMI_E1N=y`，它们决定内核
> 走**小米分支**还是**高通参考板分支**（见 `arch/arm64/boot/dts/qcom/Makefile` 开头的
> `ifeq ($(CONFIG_MACH_XIAOMI_SDM845),y)`）。若合并失效，会去编 `sdm845-v2-qvr-evt.dtb`
> 等参考板 DTB，并报 `Reference to non-existent node or label "ts_int_active"` 这类 DTC 错误
> —— **看到这个报错，就是片段没合并成功。**
>
> 这个报错的来历（已逐行核对）：`sdm845-qvr.dtsi` 的触摸屏节点写的是
> `pinctrl-0 = <&ts_int_active &ts_reset_active>`，而 `sdm845-pinctrl.dtsi` 里注册的标签
> 其实只有 `ts_int_active1` / `ts_reset_active1` —— 那个 `ts_int_active` 是**节点名**
> （后面没有冒号，所以不是标签）。dtc 解析 `&X` 时查的是**标签表**，因此失败。
> 这是上游参考板分支里的一处死代码缺陷，**我们永远不该编它**。
>
> 所以：**不要**去 DTS 里补标签、也**不要**改 Makefile 删掉那条 `dtb-y`
> —— Makefile 已经被 `ifeq ($(CONFIG_MACH_XIAOMI_SDM845),y)` 正确门控了，
> 正确做法就是让配置走小米分支。
>
> **两道防线**（4 个 workflow 里都有，专门防上面这种「静默跳过」）：
> 1. **矩阵生成阶段**：用 `jq` 校验 `defconfigFragments` 只能出现在 `kernelSource` 里，
>    写到顶层就直接报错退出。之前正是写错层级导致取到 `null`、合并被静默跳过、
>    CI 报的却是 DTC 错误，完全看不出根因。
> 2. **合并阶段**：校验片段文件真实存在，路径写错时显式失败。
>    `merge_config.sh` 遇到不存在的文件不会报错，只会静默什么都不做。

> **关于 KernelSU 版本**：所有 workflow 里的 `KERNELSU_VERSION` 固定为 `v0.9.5`，**不要改成 `main`**。
> `main` 分支已重构（变成 `kernel/hook/` + `kernel/core/`），不再兼容本项目的 4.9 内核，会直接编译失败：
> - `syscall_fn_t` 只在 `__x86_64__` 下自行定义，arm64 依赖内核提供，而 4.9 内核里没有这个类型
> - `MODULE_IMPORT_NS` 未做版本保护，而 4.9 内核没有这个宏
>
> `v0.9.5` 是**最后一个**带 `kernel_compat.h` 且对上述两项都做了版本保护的版本
> （`v1.0.0` 起保护被移除）。想升级请先确认新版本仍支持 4.9 内核。
> 刷入后，KernelSU 管理器 App 也建议使用与内核模块对应的 v0.9.5。


---


# 致谢


- [Akitlove](https://github.com/Akitlove) : 此项目所用99%+代码来自此作者,包括但不限于内核修复、Github Action
- [KernelSU](https://github.com/tiann/KernelSU) : KernelSU

# 许可

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。
