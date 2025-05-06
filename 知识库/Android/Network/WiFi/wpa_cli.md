`wpa_cli` 是一个用于与 [[wpa_supplicant]] 进行交互的命令行工具，[[wpa_supplicant]] 是一个用于连接到 Wi-Fi 网络的客户端程序，它实现了 WPA、WPA2 和 WPA3 协议。



`wpa_cli` 工具允许用户在命令行界面中连接到、断开、扫描 Wi-Fi 网络，以及查看和管理 [[wpa_supplicant]] 的状态。它通常用于无图形界面的 Linux 系统中，或者在网络配置需要更多控制时使用。

一些常用的 `wpa_cli` 命令包括：

- `wpa_cli -i <interface> status`：查看指定无线网络接口的状态。
- `wpa_cli -i <interface> scan`：扫描可用的 Wi-Fi 网络。
- `wpa_cli -i <interface> list_networks`：列出已保存的网络配置。
- `wpa_cli -i <interface> add_network`：添加一个新的网络配置。
- `wpa_cli -i <interface> set_network <network_id> ssid '"<ssid>"'`：设置网络的 SSID。
- `wpa_cli -i <interface> set_network <network_id> psk '"<psk>"'`：设置网络的预共享密钥（WPA-PSK）。
- `wpa_cli -i <interface> enable_network <network_id>`：启用一个网络配置。
- `wpa_cli -i <interface> select_network <network_id>`：选择并连接到一个网络。
- `wpa_cli -i <interface> disconnect`：断开当前连接。

使用 `wpa_cli` 时，通常需要指定无线网络接口（通过 `-i` 选项），并且可能需要 root 权限来执行某些命令。在实际操作中，`wpa_cli` 的使用会根据具体的网络环境和配置需求而有所不同。