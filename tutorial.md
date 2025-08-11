# OpenVPN 部署教程

本教程将指导你从零开始部署 OpenVPN 服务器，让你能够搭建自己的 VPN 服务。

通过本教程，你将学会：
- 如何选择和配置合适的云服务器
- OpenVPN 服务器的完整安装流程
- 客户端配置和连接方法

## 第一步：准备云服务器

### 选择云服务商
本教程以 DigitalOcean 为例进行演示。你也可以选择其他云服务商，如 AWS、阿里云、腾讯云等。

### 服务器配置要求

**操作系统建议**
- 推荐使用：**Ubuntu 22.04 LTS x64**
- 原因：稳定性和兼容性较高
- 其他选择：如果你熟悉 Debian、CentOS 等系统，也可以根据实际情况调整安装步骤

**推荐配置（DigitalOcean 最低档）：**
- 价格：$4/月（$0.006/小时）
- 内存：512 MB
- CPU：1 核心
- 存储：10 GB SSD
- 流量：500 GB/月

> **性能提示**：如果你发现通过 VPN 隧道传输数据的速度较慢，建议选择 CPU 优化型的虚拟机（Droplet），因为数据的加密和解密是非常依赖 CPU 性能的操作。

### 重要：检查服务器端口设置

在开始安装 OpenVPN 之前，你需要确保服务器的这几个端口是开放的：

| 端口号 | 协议 | 用途说明 |
|--------|------|----------|
| 22 | SSH | 用于远程登录和管理服务器 |
| 80 | HTTP | 用于网页访问（OpenVPN 配置下载） |
| 443 | HTTPS | 用于安全的网页访问 |
| 1194 | UDP | OpenVPN 默认端口（数据传输） |

**如何检查端口是否开放？**

1. **检查防火墙状态：**
   ```bash
   sudo ufw status
   ```

2. **检查端口是否被监听：**
   ```bash
   # 使用 ss 命令（推荐，现代系统默认安装）
   sudo ss -tulpn | grep -E ':(22|80|443|1194)\s'
   
   # 或者安装并使用 netstat（如果系统没有预装）
   sudo apt update && sudo apt install net-tools
   sudo netstat -tulpn | grep -E ':(22|80|443|1194)\s'
   ```

3. **如果端口未开放，使用以下命令开放端口：**
   ```bash
   # 开放 SSH 端口（通常默认已开放）
   sudo ufw allow 22/tcp
   
   # 开放 HTTP 端口
   sudo ufw allow 80/tcp
   
   # 开放 HTTPS 端口
   sudo ufw allow 443/tcp
   
   # 开放 OpenVPN 端口
   sudo ufw allow 1194/udp
   
   # 启用防火墙
   sudo ufw enable
   ```

> **DigitalOcean 用户提示**：DigitalOcean 默认是"全开放"端口策略，除非你自己在 Cloud Firewall 添加规则，所以如果跟着我使用 DigitalOcean 就不必担心这类问题。

> **新手提示**：这些命令适用于 Ubuntu/Debian 系统。如果你使用其他系统，可能需要使用不同的防火墙管理工具（如 CentOS 的 firewalld）。

## 第二步：配置 OpenVPN 服务器

### 准备工作

1. **获取服务器 IP 地址**
   - 进入你的 VPS 控制面板
   - 复制服务器的 IPv4 地址（xxx.xxx.xxx.xxx）并保存起来

2. **连接到服务器**
   - 进入服务器的终端（Terminal）界面

### 安装 OpenVPN 和相关组件

1. **更新系统包列表：**
   ```bash
   sudo apt update
   ```

2. **安装 OpenVPN 和 Easy-RSA：**
   ```bash
   sudo apt install openvpn easy-rsa
   ```

3. **处理安装过程中的提示：**
   
   如果出现磁盘空间使用确认提示：
   ```
   After this operation, 6865 kB of additional disk space will be used.
   Do you want to continue? [Y/n]
   ```
   **请输入 `Y` 并按回车键继续安装**

   如果出现紫色配置界面（通常是服务配置选项）：
   **请选择第一个选项（默认选项）并按回车键确认**

> **安装提示**：整个安装过程通常需要几分钟时间，请耐心等待安装完成。

### 配置 OpenVPN 服务

OpenVPN 的主要配置文件位于 `/etc/openvpn/server.conf`

**确认当前目录位置：**

1. **检查当前所在目录：**
   ```bash
   pwd
   ```
   
   如果显示结果是 `/root`，说明你当前在 root 用户的家目录：
   ```
   root@ubuntu-s-1vcpu-512mb-10gb-sfo3-01:~# pwd
   /root
   ```

2. **切换到根目录：**
   ```bash
   cd /
   ```
   
   再次确认位置：
   ```bash
   pwd
   ```
   
   现在应该显示：
   ```
   root@ubuntu-s-1vcpu-512mb-10gb-sfo3-01:/# pwd
   /
   ```

3. **验证目录结构（可选）：**
   ```bash
   ls
   ```
   
   你应该能看到 `etc` 文件夹以及其他系统目录。

> **目录说明**：`/` 是 Linux 系统的根目录，`/etc` 是存放系统配置文件的目录，OpenVPN 的配置文件就在 `/etc/openvpn/` 路径下。

### 编辑配置文件

> **推荐使用方法2**：比较安全，避免遗漏某些重要设置

编辑 `/etc/openvpn/server.conf` 配置文件（如果没有的话就直接创建），主要配置以下几项：

**主要配置项说明：**
- **port**: 设置 OpenVPN 使用的端口，默认是 `1194`
- **proto**: 设置协议，通常是 `udp`
- **dev**: 设置虚拟设备，通常使用 `tun`
- **server**: 配置 VPN 网络的 IP 地址范围
- **ca**、**cert**、**key**: 设置证书和密钥文件

**最简易的 server.conf 配置内容：**
```
port 1194
proto udp
dev tun
ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key
dh /etc/openvpn/easy-rsa/pki/dh.pem
server 10.8.0.0 255.255.255.0
```

#### 配置文件创建方法

**方法1：直接创建简易配置**

首先切换到 `/etc/openvpn/` 目录：
```bash
cd /etc/openvpn/
```

然后使用以下命令创建配置文件：
```bash
cat <<EOF > server.conf
client-to-client
port 1194
proto udp
dev tun
ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key
dh /etc/openvpn/easy-rsa/pki/dh.pem
redirect-gateway def1 bypass-dhcp
server 10.8.0.0 255.255.255.0
dhcp-option DNS 8.8.8.8
EOF
```

> **重要提示**：`EOF` 必须顶格写，前面不能有空格或 Tab，否则 shell 无法识别结束符号。

**方法2：使用官方示例配置（推荐）**

如果担心出现未知问题，可以使用官方提供的配置模板。官方文档在安装 OpenVPN 时就已经存在了。

使用以下命令复制官方配置：
```bash
gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz > /etc/openvpn/server.conf
```

> **配置文件位置说明**：官方示例配置位于 `/usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz`

### 生成证书和密钥文件

在配置文件中，我们指定了几个重要的证书和密钥文件，现在需要生成这些文件：

**证书和密钥文件说明：**

| 配置文件指向 | 文件作用 |
|-------------|----------|
| `ca /etc/openvpn/easy-rsa/pki/ca.crt` | 根证书（CA） |
| `cert /etc/openvpn/easy-rsa/pki/issued/server.crt` | 服务器的证书 |
| `key /etc/openvpn/easy-rsa/pki/private/server.key` | 服务器的私钥 |
| `dh /etc/openvpn/easy-rsa/pki/dh.pem` | Diffie-Hellman 参数（可选） |

#### 证书生成步骤

**1. 复制 Easy-RSA 到 OpenVPN 目录**

```bash
make-cadir /etc/openvpn/easy-rsa
```

**2. 进入目录并初始化 PKI**

> **可选配置**：编辑 `vars` 文件（Windows 下为 `vars.bat`），设置 `KEY_COUNTRY`、`KEY_PROVINCE`、`KEY_CITY`、`KEY_ORG` 和 `KEY_EMAIL` 参数。这些参数不能留空。

初始化 PKI（公钥基础设施）：
```bash
cd /etc/openvpn/easy-rsa
./easyrsa init-pki
```

执行成功后会显示：
```
Note: using Easy-RSA configuration from: /etc/openvpn/easy-rsa/vars
init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/easy-rsa/pki
```

> **说明**：PKI 已初始化完成，生成的 `pki/` 文件夹将存放后续的证书和密钥。

**3. 生成证书和密钥文件**

##### 生成 CA（证书颁发机构）

```bash
./easyrsa build-ca
```

**交互过程：**
- `Enter New CA Key Passphrase:` - 为 CA 私钥设置密码（请牢记，后续签发证书需要用到）
- `Re-Enter New CA Key Passphrase:` - 再次输入密码确认
- `Common Name:` - CA 的名称，建议填写 `MyVPN-CA` 或其他易记名称（可直接回车使用默认值）

**操作示例：**
```bash
Enter New CA Key Passphrase: mycapassword
Re-Enter New CA Key Passphrase: mycapassword
Common Name (eg: your user, host, or server name) [Easy-RSA CA]: MyVPN-CA
```

**生成的文件：**
- `/etc/openvpn/easy-rsa/pki/ca.crt` - CA 证书
- `/etc/openvpn/easy-rsa/pki/private/ca.key` - CA 私钥

##### 生成服务器证书和私钥

```bash
./easyrsa build-server-full server nopass
```

> **参数说明**：
> - `server` - 服务器证书的名称（可修改为其他名称）
> - `nopass` - 生成的私钥不加密码（方便自动启动）

**交互过程：**
- `Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key:` - 输入之前创建 CA 时设置的密码
- `Common Name:` - 服务器证书名称，建议直接回车使用默认值 `server`

**操作示例：**
```bash
Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key: mycapassword
Common Name (eg: your user, host, or server name) [server]: 
```

**生成的文件：**
- `/etc/openvpn/easy-rsa/pki/private/server.key` - 服务器私钥
- `/etc/openvpn/easy-rsa/pki/reqs/server.req` - 证书请求文件

---

生成 KEY (用 CA 签发服务端证书)

   ```bash
   ./easyrsa sign-req server server
   ```

交互说明:
	•	Type the word ‘yes’ to continue: 输入 yes 并按下 Enter 确认签发。
	•	Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key: 输入 在 build-ca 阶段设置的 CA 密码，然后按 Enter。

例子:
   ```bash
   Type the word 'yes' to continue: yes
   Enter pass phrase for /etc/openvpn/easy-rsa/pki/private/ca.key: mycapassword
   ```

执行完成后会生成:
   •	/etc/openvpn/easy-rsa/pki/issued/server.crt(服务端证书)

---

生成 dh.pem (Diffie-Hellman 参数文件)[可选]
功能:VPN 连接时的安全性和加密强度

   ```bash
   ./easyrsa gen-dh
   ```

交互说明:
	./easyrsa gen-dh 这个步骤是 没有交互提示 的，跟 build-ca 或 sign-req 不一样。
   执行时它会直接在终端跑出一堆生成过程（主要是大素数计算）

例子:
   ```bash
   root@server:/etc/openvpn/easy-rsa# ./easyrsa gen-dh
   Generating DH parameters, 2048 bit long safe prime, generator 2
   This is going to take a long time
   ........................................+.......................................................................................
   DH parameters of size 2048 created at /etc/openvpn/easy-rsa/pki/dh.pem
   ```

执行完成后会生成:
   •	**生成的文件：**
- `/etc/openvpn/easy-rsa/pki/dh.pem` - Diffie-Hellman 参数文件



### 忘记 CA 密码的解决方案

如果忘记了 CA 密码，由于 CA 密钥设定密码后**必须输入正确密码才能签发证书**，只能通过以下步骤重新生成：

1. 删除 `pki/` 目录
2. 重新执行 `./easyrsa init-pki`
3. 重新执行 `./easyrsa build-ca` 并设置新密码
4. 重新生成所有服务器证书

### 配置 IP 转发和防火墙

为了使 VPN 客户端能够访问互联网，需要启用 IP 转发并配置防火墙规则。

#### 1. 启用 IP 转发

编辑 `/etc/sysctl.conf` 并确保存在以下配置（如被注释请去掉开头的 `#`）：

```bash
net.ipv4.ip_forward = 1
```

修改文件的方法：

```bash
sudo nano /etc/sysctl.conf
```

找到这一行：

```bash
#net.ipv4.ip_forward=1
```

去掉 `#` 保存退出，然后应用更改：

```bash
sudo sysctl -p
```

#### 2. 配置防火墙规则（允许 VPN 流量通过）

UFW(推荐｜简单)：

只需执行以下命令即可允许 OpenVPN 默认端口（UDP 1194）通过防火墙：
```bash
sudo ufw allow 1194/udp
```

输出结果:
```bash
Rules updated
Rules updated (v6)
```
就表示 IPv4 和 IPv6 都放行了。

iptables(更灵活)：

允许 NAT 转发:
```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

确保规则重启后仍然生效，保存 iptables 规则：

```bash
sudo iptables-save > /etc/iptables/rules.v4
```

如果出现以下错误：

```
-bash: /etc/iptables/rules.v4: No such file or directory
```

说明 `/etc/iptables/` 目录尚未创建，按以下步骤处理：

```bash
sudo mkdir -p /etc/iptables
sudo iptables-save > /etc/iptables/rules.v4
```



### 启动 OpenVPN 服务

1. **启动 OpenVPN 服务：**
```bash
sudo systemctl start openvpn@server
 ```

2. **设置开机自启动（可选）：**
```bash
sudo systemctl enable openvpn@server
```

### 检查 OpenVPN 服务状态

你可以通过以下命令检查 OpenVPN 服务是否正在运行：

```bash
sudo systemctl status openvpn@server
```

如果正常运行，你会看到服务状态显示为 "active (running)"。

---


## 创建客户端证书和密钥

> 下例以客户端名称 `client1` 为例，您可以替换为其他名字（如每个设备一个名字）。

### 1. 切换到 easy-rsa 目录

```bash
cd /etc/openvpn/easy-rsa
```

### 2. 生成客户端私钥与证书签名请求（CSR）

```bash
./easyrsa gen-req client1 nopass
```

执行完成后会生成以下文件：
- `/etc/openvpn/easy-rsa/pki/private/client1.key`（客户端私钥）
- `/etc/openvpn/easy-rsa/pki/reqs/client1.req`（客户端证书请求）

> 说明：`nopass` 表示客户端私钥不设密码，导入更方便；如需更高安全性，可去掉 `nopass` 自行设密。

### 3. 用 CA 签发客户端证书

```bash
./easyrsa sign-req client client1
```

交互说明：
- Type the word 'yes' to continue: 输入 `yes` 确认
- Enter pass phrase for .../ca.key: 输入 CA 密码

签发完成后会生成：
- `/etc/openvpn/easy-rsa/pki/issued/client1.crt`（客户端证书）

### 4. 准备生成 .ovpn 所需文件

从服务器上收集以下文件内容（建议先复制到临时目录便于打包）：
- CA 根证书：`/etc/openvpn/easy-rsa/pki/ca.crt`
- 客户端证书：`/etc/openvpn/easy-rsa/pki/issued/client1.crt`
- 客户端私钥：`/etc/openvpn/easy-rsa/pki/private/client1.key`
- 可选，静态密钥（如启用 tls-crypt）：`/etc/openvpn/ta.key`

例如：
```bash
mkdir -p ~/ovpn/client1
cp /etc/openvpn/easy-rsa/pki/ca.crt \
   /etc/openvpn/easy-rsa/pki/issued/client1.crt \
   /etc/openvpn/easy-rsa/pki/private/client1.key \
   ~/ovpn/client1/
# 若使用 tls-crypt 再复制一份 ta.key
[ -f /etc/openvpn/ta.key ] && cp /etc/openvpn/ta.key ~/ovpn/client1/
```

接着通过以下三行获取信息
```bash
cat ~/ovpn/client1/ca.crt
cat ~/ovpn/client1/client1.crt
cat ~/ovpn/client1/client1.key
```


## 配置客户端（应用/App）

### 生成 .ovpn 客户端配置文件（示例模板）
#### 可以复制[example.ovpn]直接修改

将下面模板保存为 `client1.ovpn`，并将占位符与内嵌证书内容替换为你的实际信息。

```ovpn
client
dev tun
proto udp
remote YOUR.SERVER.IP 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-CBC
auth SHA256
verb 3
# 若服务器/客户端启用了压缩才添加（现代配置通常不再启用）
# comp-lzo

<ca>
# 粘贴 ca.crt 的全部内容
-----BEGIN CERTIFICATE-----
...CA CERT HERE...
-----END CERTIFICATE-----
</ca>

<cert>
# 粘贴 client1.crt 的全部内容（包含 BEGIN/END CERTIFICATE）
-----BEGIN CERTIFICATE-----
...CLIENT CERT HERE...
-----END CERTIFICATE-----
</cert>

<key>
# 粘贴 client1.key 的全部内容（包含 BEGIN/END PRIVATE KEY）
-----BEGIN PRIVATE KEY-----
...CLIENT KEY HERE...
-----END PRIVATE KEY-----
</key>

# 如服务器使用 tls-crypt，在此内嵌 ta.key
# 并在服务器端与客户端均配置 tls-crypt 指令保持一致
# tls-crypt 字段用于混淆/加密控制信道，提高抗干扰能力
# 若未使用，请删除整个块与对应指令
# tls-crypt ta.key
<tls-crypt>
-----BEGIN OpenVPN Static key V1-----
...TA KEY HERE...
-----END OpenVPN Static key V1-----
</tls-crypt>
```

> 使用说明：
> - 将 `YOUR.SERVER.IP` 替换为你的服务器公网 IP 或域名。
> - 若服务器监听的端口/协议不同（如 TCP 或自定义端口），同步修改 `proto` 与 `remote`。
> - 若未使用 `tls-crypt`，请删除 `<tls-crypt>...</tls-crypt>` 块以及配置中的相关指令。
> - 在 Mac 上使用 Tunnelblick、在 Windows 上用 OpenVPN GUI、在 iOS/Android 上用官方 OpenVPN Connect，导入 `client1.ovpn` 即可连接。

# 开启防火墙（重要!）
sudo ufw allow 1194/udp           # 允许OpenVPN默认UDP端口
sudo ufw allow OpenSSH            # 允许SSH端口（通常是22端口）
sudo ufw allow in on tun0         # 允许tun0接口入站流量（VPN接口）
sudo ufw allow out on tun0        # 允许tun0接口出站流量
sudo ufw default allow routed     # 允许路由转发流量
sudo ufw enable                   # 启用防火墙（如果还没启用的话）
sudo ufw reload                   # 重新加载规则

# 关于 TLS 握手失败日志
当 OpenVPN Server 启动并对公网开放端口（默认 UDP 1194）后，可能会在 systemctl status openvpn@server 或 /var/log/syslog 看到类似日志:
```bash
TLS Error: TLS handshake failed
TLS Error: TLS key negotiation failed to occur within 60 seconds
OpenSSL: error:... peer did not return a certificate
```
这是正常现象
	•	只要你的 VPN 没有客户端连接，这些日志多半是全球sohai互联网扫描器、爬虫或错误配置的客户端随机发来的握手请求。
	•	因为它们没有正确的 .ovpn 配置文件、证书和密钥，所以 TLS 握手必然失败。
	•	对服务器正常运行没有影响。

---

# 大致上完成，运行成功。
# 后续可能会添加ipv6的设置，如果一直无法使用建议先关闭电脑本机的ipv6

```bash
#关闭本机wifi的ipv6(macos)
sudo networksetup -setv6off "Wi-Fi"

#开启本机wifi的ipv6(macos)
sudo networksetup -setv6automatic "Wi-Fi"
```


