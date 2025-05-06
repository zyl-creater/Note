在 Linux 系统中，`ip` 命令是一个功能强大的工具，用于配置和管理网络接口、路由、邻居、隧道等网络相关参数。它取代了传统的 `ifconfig` 和 `route` 命令，提供了更全面和灵活的网络配置选项。

以下是一些常用的 `ip` 命令及其用途：

1. `ip address`：用于显示和管理网络接口的 IP 地址。
    
    - 显示所有接口的 IP 地址：
        ```
        ip address show
        ```
        
    - 为接口添加 IP 地址：
        ```
        ip address add 192.168.1.100/24 dev eth0
        ```
        
2. `ip link`：用于显示和管理网络接口的链路层信息。
    
    - 显示所有接口的状态：
        ```
        ip link show
        ```
        
    - 启用或禁用接口：
        ```
        ip link set eth0 up
        ip link set eth0 down
        ```
        
3. `ip route`：用于显示和管理路由表。
    
    - 显示路由表：
        ```
        ip route show
        ```
        
    - 添加默认路由：
        ```
        ip route add default via 192.168.1.1
        ```
        
4. `ip rule`：用于显示和管理路由策略数据库。
    
    - 显示路由策略规则：
        ```
        ip rule show
        ```
        
5. `ip neighbor`：用于显示和管理邻居缓存（ARP 表）。
    
    - 显示邻居缓存：
        ```
        ip neighbor show
        ```
        
6. `ip tunnel`：用于显示和管理 IP 隧道。
    
    - 创建一个 GRE 隧道：
        ```
        ip tunnel add gre1 mode gre remote 192.168.1.2 local 192.168.1.1
        ```
        
7. `ip maddr`：用于显示和管理多播地址。
    
    - 显示多播地址：
```
 ip maddr show
```
8. `ip monitor`：用于监控网络设备的状态变化。
    
    - 监控接口状态变化：
        ```
        ip monitor link
        ```
这些命令提供了对 Linux 网络堆栈的详细控制和监控，是网络管理和故障排除的重要工具。大多数 `ip` 命令的操作都需要管理员权限，因此通常需要使用 `sudo` 来执行。