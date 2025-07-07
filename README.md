# wireguard-teleoperation
1 系统要求
----------------------------------
1.1 硬件要求
5G CPE 设备：两台 ZTE MC8020 Pro3 5G CPE（支持 2.4GHz 和 5GHz 频段，最高无线接入速率 5400Mbps）。
操作端与受控端计算机：支持 Ubuntu、Windows 或 macOS 系统的计算机，用于部署 WireGuard 客户端。
网络环境：操作端和受控端需接入 5G 网络，建议带宽分别为 100Mbps 至 1000Mbps（根据实验需求选择）。

1.2 软件要求
操作系统：
Google Cloud 虚拟机：Ubuntu（推荐 20.04 LTS 或更高版本，架构为 x86/64，内存 40GB，机型 e2-custom-2-1024，部署在 asia-east2-b 区域，分配香港公共 IP）。
操作端与受控端计算机：支持 Ubuntu、Windows 或 macOS。

2 部署步骤
----------------------------------------
准备 5G CPE 设备
设备采购与设置：
采购两台 ZTE MC8020 Pro3 5G CPE 设备，分别部署在操作端和受控端。
将 5G CPE 连接到 5G 网络，确保设备能够接收运营商的 5G 信号并转换为 WiFi 信号。

2.2 配置 Google Cloud 虚拟机
创建虚拟机：
登录 Google Cloud Platform（GCP）控制台，创建一个新的 Ubuntu 虚拟机实例。
配置虚拟机参数：
架构：x86/64
内存：40GB
机型：e2-custom-2-1024
区域：asia-east2-b（香港）
分配一个公共 IP 地址（静态 IP，推荐香港 IP）。
启动虚拟机并记录其公共 IP 地址。
启用 IP 转发：
登录虚拟机（通过 SSH 或 GCP 控制台）。
编辑 /etc/sysctl.conf 文件，启用 IP 转发：sudo nano /etc/sysctl.conf
添加或修改以下行：net.ipv4.ip_forward=1
应用更改：sudo sysctl -p

2.3 安装与配置 WireGuard
在 Google Cloud 虚拟机上安装 WireGuard（服务器端）：
更新系统并安装 WireGuard：
sudo apt update
sudo apt install wireguard

生成服务器的公钥和私钥：wg genkey | tee /etc/wireguard/privatekey | wg pubkey > /etc/wireguard/publickey
创建 WireGuard 配置文件（例如 /etc/wireguard/wg0.conf）：sudo nano /etc/wireguard/wg0.conf

示例配置文件内容：

[Interface]
PrivateKey = <服务器私钥>
Address = 10.0.0.1/24
ListenPort = 51820

[Peer]
PublicKey = <操作端公钥>
AllowedIPs = 10.0.0.2/32

[Peer]
PublicKey = <受控端公钥>
AllowedIPs = 10.0.0.3/32

启动 WireGuard 服务：
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0

在操作端与受控端计算机上安装 WireGuard（客户端）：

根据操作系统安装 WireGuard：

Ubuntu：
sudo apt update
sudo apt install wireguard

Windows/macOS：从 WireGuard 官网下载并安装客户端软件。
为每台客户端生成公钥和私钥：wg genkey | tee privatekey | wg pubkey > publickey

创建客户端配置文件（例如 client1.conf 和 client2.conf）：

[Interface]
PrivateKey = <客户端私钥>
Address = 10.0.0.2/24  # 操作端使用 10.0.0.2，受控端使用 10.0.0.3
DNS = 8.8.8.8

[Peer]
PublicKey = <服务器公钥>
Endpoint = <服务器公共 IP>:51820
AllowedIPs = 0.0.0.0/0, ::/0

启动 WireGuard 客户端：
Ubuntu：sudo wg-quick up ./client1.conf
Windows/macOS：通过 WireGuard 客户端 GUI 导入配置文件并激活。

交换公钥：
将操作端和受控端的公钥添加到服务器的配置文件中。
将服务器的公钥添加到操作端和受控端的客户端配置文件中。

测试 VPN 连接：
在操作端和受控端分别 ping 对方分配的 VPN IP 地址（例如 10.0.0.2 和 10.0.0.3）：
ping 10.0.0.2
ping 10.0.0.3
