---
title: windows开启OpenSSH远程访问
abbrlink: 40508
date: 2024-02-05 16:42:21
tags: [windows,wls]
---


### 安装
[官方安装](https://learn.microsoft.com/zh-cn/windows-server/administration/openssh/openssh_install_firstuse?tabs=powershell)

使用 PowerShell 安装 OpenSSH，请先以管理员身份运行 PowerShell。 为了确保 OpenSSH 可用，请运行以下 cmdlet：
```shell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```
如果两者均尚未安装，则此命令应返回以下输出：
```shell
Name  : OpenSSH.Client~~~~0.0.1.0
State : NotPresent

Name  : OpenSSH.Server~~~~0.0.1.0
State : NotPresent
```
然后，根据需要安装服务器或客户端组件：
```shell
# Install the OpenSSH Client
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

# Install the OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```
这两个命令应会返回以下输出：
```shell
Path          :
Online        : True
RestartNeeded : False
```

但是执行第二条命令后卡住，手动回车出错,错误信息如下：
```shell
PS C:\Windows\system32> Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0                                   

Add-WindowsCapability : Add-WindowsCapability 失败。错误代码 = 0x80240438
所在位置 行:1 字符: 1
+ Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Add-WindowsCapability], COMException
    + FullyQualifiedErrorId : Microsoft.Dism.Commands.AddWindowsCapabilityCommand
```
解决上述问题，手动下载对应版本[OpenSSH安装包](https://github.com/PowerShell/Win32-OpenSSH/releases)手动创建 C:\Program Files\OpenSSH 目录;
如果已经存在了该目录，检查install-sshd.ps1是否存在，不存在将下载的安装包该文件放入
PowerShell中在C:\Program Files\OpenSSH 目录下执行
```shell
cd 'C:\Program Files\OpenSSH'

powershell.exe -ExecutionPolicy Bypass -File install-sshd.ps1
```

启动并配置 OpenSSH 服务器以供初始使用，请打开提升的 PowerShell 提示符（右键单击，以管理员身份运行），然后运行以下命令以启动 sshd service：
```shell
# Start the sshd service
Start-Service sshd

# OPTIONAL but recommended:
Set-Service -Name sshd -StartupType 'Automatic'

# Confirm the Firewall rule is configured. It should be created automatically by setup. Run the following to verify
if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue | Select-Object Name, Enabled)) {
    Write-Output "Firewall Rule 'OpenSSH-Server-In-TCP' does not exist, creating it..."
    New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
} else {
    Write-Output "Firewall rule 'OpenSSH-Server-In-TCP' has been created and exists."
}
```
### 连接到OpenSSH服务器
使用 PowerShell 输入下面命令，username是当前windows登录用户名，执行后输出用户密码即可登录
```shell
ssh username@127.0.0.1 
```

### 卸载OpenSSH
使用 PowerShell 卸载 OpenSSH 组件，请使用以下命令：
```shell
# Uninstall the OpenSSH Client
Remove-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

# Uninstall the OpenSSH Server
Remove-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```