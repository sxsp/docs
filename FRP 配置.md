# FRP 配置

### 1. 检查服务器CPU架构

​	执行下列命令检查CPU架构，常见为 **x86_64**，后续教程以 **x86_64** 为例。

```
uname -m
```



### 2. 下载 frp

​	执行下列命令，其中VERSION或许需要跟随时代潮流更新。

```
cd /tmp
VERSION=0.69.1
wget "https://github.com/fatedier/frp/releases/download/v${VERSION}/frp_${VERSION}_linux_amd64.tar.gz"
```

​	检查文件：

```
ls -lh frp_0.69.1_linux_amd64.tar.gz
```

​	验证文件，其中 **7be257b72dbbc60bcb3e0e25a5afd1dfac7b63f897084864d3c956dd3d5674e1** 为官方提供的 SHA-256。

```
echo "7be257b72dbbc60bcb3e0e25a5afd1dfac7b63f897084864d3c956dd3d5674e1  frp_0.69.1_linux_amd64.tar.gz" | sha256sum -c -
```

​	正常会显示：

```
frp_0.69.1_linux_amd64.tar.gz: OK
```



### 3. 解压并安装 frps

​	解压：

```
tar -xzf frp_0.69.1_linux_amd64.tar.gz
```

​	查看：

```
ls frp_0.69.1_linux_amd64
```

​	应当包含：

```
frps
frpc
frps.toml
frpc.toml
LICENSE
```

​	将程序复制到系统目录：

```
sudo install -m 755 frp_0.69.1_linux_amd64/frps /usr/local/bin/frps
```

​	确认文件存在：

```
ls -l /usr/local/bin/frps
```

---

​	生成认证 **token**：

```
openssl rand -hex 32
```

​	会生成类似：**0ef45412b6d9b63f4663eb1fc15994135e1c764a18430a78c5a8307333209c11**，保存好，创建服务器配置文件并编辑：

```
sudo mkdir -p /etc/frp
sudo vim /etc/frp/frps.toml
```

​	写入以下内容：

```bash
# frpc 连接 frps 的控制端口
bindAddr = "0.0.0.0"
bindPort = 7000

# 客户端认证
auth.method = "token"
auth.token = "替换成刚才生成的64位Token"

# 强制客户端使用 TLS 连接
transport.tls.force = true

# 只允许客户端使用这些公网映射端口
# 目前只开放 6000，用于 Windows SSH
allowPorts = [
  { single = 6000 },
  { single = 6001 }
]

# frps 管理页面
# 只监听本机，不直接暴露到公网
webServer.addr = "127.0.0.1"
webServer.port = 7500
webServer.user = "frpadmin"
webServer.password = "Frpguo2003"
```

---

​	验证服务器配置：

```bash
sudo /usr/local/bin/frps verify -c /etc/frp/frps.toml
```

​	正常应显示：

```bash
frps: the configuration file /etc/frp/frps.toml syntax is ok
```

​	启动测试，若无报错并有监听信息，则 **frps** 启动成功，可 **Ctrl+C** 停止测试。

```bash
sudo /usr/local/bin/frps -c /etc/frp/frps.toml
```

​	（重启 **frps** 任务）：

```bash
sudo systemctl restart frps
```



### 4. 配置frps开机自启动

​	创建专用账户：

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin frp 2>/dev/null || true
```

​	保存配置文件，使普通用户无法直接读取 **Token**。

```bash
sudo chown root:frp /etc/frp/frps.toml
sudo chmod 640 /etc/frp/frps.toml
```

---

​	创建 **systemd** 服务：

```bash
sudo vim /etc/systemd/system/frps.service
```

​	写入：

```bash
[Unit]
Description=frp server
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=frp
Group=frp
ExecStart=/usr/local/bin/frps -c /etc/frp/frps.toml
Restart=on-failure
RestartSec=5s
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

​	依次执行：

```bash
sudo systemctl daemon-reload # 重新加载systemd
sudo systemctl enable --now frps # 设置开机启动并立即启动
sudo systemctl status frps --no-pager # 查看状态
```

​	正常可看到：

```
Active: active (running)
```

​	实时查看日志，使用 **Ctrl+C** 退出：

```
sudo journalctl -u frps -f
```



### 5. 开放公网服务器端口

​	在控制台开放两个 **tcp** 端口：7000，用于 **frpc** 连接 **frps**，6000，用于公网访问 **Windows SSH**。

​	运行 **sudo firewall-cmd --state** 命令发现防火墙开启后，执行以下命令设置端口和验证：

```bash
sudo firewall-cmd --permanent --add-port=7000/tcp
sudo firewall-cmd --permanent --add-port=6000/tcp
sudo firewall-cmd --reload

sudo firewall-cmd --list-ports
```



### 6. 内网 **Windows** 部署 **frpc**

​	*管理员打开 PowerShell：Win+R 输入 powershell 后按 Ctrl+Shift+Enter*

​	**PowerShell** 内创建安装目录：

```powershell
New-Item -ItemType Directory -Path C:\frp -Force
```

​	下载 **frp**：

```powershell
$version = "0.69.1"

$zipPath = "$env:TEMP\frp_windows_amd64.zip"
$extractPath = "$env:TEMP\frp_extract"

Invoke-WebRequest -Uri "https://github.com/fatedier/frp/releases/download/v${version}/frp_${version}_windows_amd64.zip" -OutFile $zipPath

Remove-Item $extractPath -Recurse -Force -ErrorAction SilentlyContinue

Expand-Archive -Path $zipPath -DestinationPath $extractPath -Force

Copy-Item "$extractPath\frp_${version}_windows_amd64\frpc.exe" "C:\frp\frpc.exe" -Force
```

​	检查，正常应看到 **frpc.exe**

```powershell
Get-ChildItem C:\frp
```

---

​	运行 **Get-Service sshd** 检查 **Windows SSH** 服务是否开启，若未开启执行以下步骤：

​	查看系统版本：

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsBuildNumber
```

​	检查 **OpenSSH** 安装状态：

```powershell
Get-WindowsCapability -Online | Where-Object Name -Like "OpenSSH*"
```

​	安装 **OpenSSH Server**：

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

​	启动 **sshd** 并设置开机自启：

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
Get-Service sshd
```

---

​	查看 **TCP 22** 入站规则是否建立：

```powershell
Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue
```

​	若能看到规则再检查是否启用：

```powershell
Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" | Select-Object Name, DisplayName, Enabled, Direction, Action
```

​	若没有输出则手动创建：

```powershell
New-NetFirewallRule `
    -Name "OpenSSH-Server-In-TCP" `
    -DisplayName "OpenSSH Server (sshd)" `
    -Enabled True `
    -Direction Inbound `
    -Protocol TCP `
    -Action Allow `
    -LocalPort 22
```

​	检查 22 端口是否监听：

```powershell
netstat -ano | findstr ":22"
```

---

​	测试本机 **ssh**：

```powershell
ssh Administrator@127.0.0.1
```

​	若要输入密码但没设置密码，执行以下命令确认用户名并设置密码：

```powershell
$env:USERNAME
net user $env:USERNAME *
```

​	若用户存在问题，可新建一个 **sshuser** 用户用于服务：

```powershell
$password = Read-Host "请为 SSH 用户设置密码" -AsSecureString

New-LocalUser -Name "sshuser" -Password $password -FullName "SSH User" -Description "Local account for OpenSSH login"
```

---

​	创建 **frpc** 配置文件：

```powershell
notepad C:\frp\frpc.toml

# 公网 frps 地址
serverAddr = "PUBLIC_IP"
serverPort = 7000

user = "pc_a"

# 首次连接失败时不要直接退出
# 适合开机时网络尚未就绪的情况
loginFailExit = false

# 必须与 frps.toml 完全一致
auth.method = "token"
auth.token = "替换成服务端相同的Token"

# 显式启用 TLS
transport.tls.enable = true

# 保存客户端日志
log.to = "C:/frp/frpc.log"
log.level = "info"
log.maxDays = 7

# 本机管理界面，只允许 Windows 本机访问
webServer.addr = "127.0.0.1"
webServer.port = 7400
webServer.user = "frpcadmin"
webServer.password = "Frpguo2003"

# 将公网服务器的 6000 端口
# 映射到这台 Windows 的 22 端口
[[proxies]]
name = "company_windows_ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 6000 # 不同设备设置不同端口
```

​	完成后验证配置格式：

```powershell
cd C:\frp

.\frpc.exe verify -c .\frpc.toml
```

​	检查 **Windows** 能否连接 **frps**：

```powershell
Test-NetConnection PUBLIC_IP -Port 7000
```

​	前台启动 **frpc**：

```powershell
cd C:\frp

.\frpc.exe -c .\frpc.toml
```

​	（重启 **frpc** 任务）：

```
Stop-ScheduledTask -TaskName "frpc"; Get-Process frpc -ErrorAction SilentlyContinue | Stop-Process -Force; Start-ScheduledTask -TaskName "frpc"
```

​	在公网服务器上查看日志：

```bash
sudo journalctl -u frps -f
```

---

​	测试连接：

```powershell
ssh -p 6000 WINDOWS用户名@PUBLIC_IP
```



### 7. 配置 Windows frpc 开机自启动

​	管理员打开 **PowerShell**，创建启动任务：

```powershell
$action = New-ScheduledTaskAction `
  -Execute "C:\frp\frpc.exe" `
  -Argument "-c C:\frp\frpc.toml" `
  -WorkingDirectory "C:\frp"

$trigger = New-ScheduledTaskTrigger -AtStartup

$settings = New-ScheduledTaskSettingsSet `
  -RestartCount 999 `
  -RestartInterval (New-TimeSpan -Minutes 1) `
  -ExecutionTimeLimit ([TimeSpan]::Zero) `
  -StartWhenAvailable

Register-ScheduledTask `
  -TaskName "frpc" `
  -Description "frp client for intranet access" `
  -Action $action `
  -Trigger $trigger `
  -Settings $settings `
  -User "SYSTEM" `
  -RunLevel Highest `
  -Force
```

​	启动任务：

```powershell
Start-ScheduledTask -TaskName "frpc"
```

​	查看状态：

```powershell
Get-ScheduledTask -TaskName "frpc"
Get-ScheduledTaskInfo -TaskName "frpc"
```

​	用以下命令查看：

```powershell
Get-Process frpc # 查看进程（重启后能看到进程说明开机自启成功）
Get-Content C:\frp\frpc.log -Tail 50 # 查看日志
Get-Content C:\frp\frpc.log -Tail 50 -Wait # 持续观察日志
```



### 8. 访问 frps管理页面

​	在自己的设备上建立 **SSH** 隧道：

```powershell
ssh -L 7500:127.0.0.1:7500 服务器用户名@PUBLIC_IP
```

​	保持 **SSH** 窗口打开，浏览器进入 **http://127.0.0.1:7500**

​	本机的 **frpc** 管理页面：**http://127.0.0.1:7400**



### 9. 通过 SSH 安全访问远程桌面

​	在自己的设备上运行：

```powershell
ssh -p 6000 -L 13389:127.0.0.1:3389 WINDOWS用户名@PUBLIC_IP
```

​	保持窗口，另一窗口运行：

```powershell
mstsc /v:127.0.0.1:13389
```

