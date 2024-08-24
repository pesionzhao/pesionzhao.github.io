### windows配置ssh服务

[适用于 Windows 的 OpenSSH 入门](https://learn.microsoft.com/zh-cn/windows-server/administration/openssh/openssh_install_firstuse?tabs=powershell)

[Windows 免密码 SSH 连接指南](https://gist.github.com/wklchris/d875e7056d5536ab520a7921f85a2a7c)

以管理员身份运行powershell

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

会显示你是否安装ssh-client和ssh-server,之后按需安装

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

启动服务

```powershell
# Start the sshd service
Start-Service sshd

# 开机自启动 OPTIONAL but recommended:
Set-Service -Name sshd -StartupType 'Automatic'

# 判断防火墙是否允许 Confirm the Firewall rule is configured. It should be created automatically by setup. Run the following to verify
if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue | Select-Object Name, Enabled)) {
    Write-Output "Firewall Rule 'OpenSSH-Server-In-TCP' does not exist, creating it..."
    New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
} else {
    Write-Output "Firewall rule 'OpenSSH-Server-In-TCP' has been created and exists."
}
```

```powershell
ssh localhost
```

### 安装ZeroTier

首先在官网注册账号: [zerotier](https://www.zerotier.com/), 点击`Create A Network`, 此时会得到一个唯一的**Network ID**,点击该网络进入详情页,也就是管理员界面,

本地机和远程机都下载[客户端](https://www.zerotier.com/download/), 安装好后点击`join a new network`可以输入之前得到的**Network ID**即可完成绑定,此时需要在管理员界面完成授权,可以完成连接,就可通过获得的新ip进行ssh

[内网穿透神器ZeroTier使用教程](https://muzihuaner.github.io/2021/09/22/%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F%E7%A5%9E%E5%99%A8ZeroTier%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/#ZeroTier-%E5%8E%9F%E7%90%86)

[【教程】超详细安装和使用免费内网穿透软件Zerotier-One](https://cloud.tencent.com/developer/article/2390573)