---
title: WSL2 + VcXsrv 实现Ubuntu图形化界面
tags:
  - Linux
  - WSL2
  - VcXsrv
categories:
  - 后端
  - Linux
abbrlink: a678c4cd
date: 2023-03-01 19:12:21
---
使用WSL2运行Linux比在Vmware上搭建Linux虚拟环境资源消耗更小，更加高效。

自己配置Ubuntu的过程踩了不少坑，在这里记录一下配置过程以及遇到的问题及有效解决方案。

<!-- more -->

# 使用 WSL 在 Windows 上安装 Linux

## 先决条件

必须运行 Windows 10 版本 2004 及更高版本（内部版本 19041 及更高版本）或 Windows 11 才能使用以下命令。 如果使用的是更早的版本，请参阅[手动安装页](https://learn.microsoft.com/zh-cn/windows/wsl/install-manual)。

## 安装WSL

在管理员模式下打开 PowerShell 或 Windows 命令提示符，输入

```powershell
wsl --install
```

此命令将启用运行 WSL 并安装 Linux 的 Ubuntu 发行版所需的功能。

之后输入 `wsl -l -v` ，查看以下已安装 Linux 信息，如下图。

![1677672851148](1677672851148.png)

此时Ubuntu已经安装完成，可以通过开始菜单打开或在命令行输入 `ubuntu` 进入Ubuntu，配置好用户信息。

## WSL命令

<details>
<summary>记录WSL常用的命令</summary>

**安装**

```powershell
wsl --install
```

安装 WSL 和 Linux 的默认 Ubuntu 发行版。还可以使用此命令通过运行 `wsl --install <Distribution Name>` 来安装其他 Linux 发行版。 若要获取发行版名称的有效列表，请运行 `wsl --list --online`。

选项包括：

* `--distribution`：指定要安装的 Linux 发行版。 可以通过运行 `wsl --list --online` 来查找可用的发行版。
* `--no-launch`：安装 Linux 发行版，但不自动启动它。
* `--web-download`：通过联机渠道安装，而不是使用 Microsoft Store 安装。

未安装 WSL 时，选项包括：

* `--inbox`：使用 Windows 组件（而不是 Microsoft Store）安装 WSL。 *（WSL 更新将通过 Windows 更新接收，而不是通过 Microsoft Store 中推送的可用更新来接收）。*
* `--enable-wsl1`：在安装 Microsoft Store 版本的 WSL 的过程中也启用“适用于 Linux 的 Windows 子系统”可选组件，从而启用 WSL 1。
* `--no-distribution`：安装 WSL 时不安装发行版。

**列出可用的 Linux 发行版**

```powershell
wsl --list --online
```

查看可通过在线商店获得的 Linux 发行版列表。 此命令也可输入为：`wsl -l -o`。

**列出已安装的 Linux 发行版**

```powershell
wsl --list --verbose
```

查看安装在 Windows 计算机上的 Linux 发行版列表，其中包括状态（发行版是正在运行还是已停止）和运行发行版的 WSL 版本（WSL 1 或 WSL 2）。此命令也可输入为：`wsl -l -v`。 可与 list 命令一起使用的其他选项包括：`--all`（列出所有发行版）、`--running`（仅列出当前正在运行的发行版）或 `--quiet`（仅显示发行版名称）。

**将 WSL 版本设置为 1 或 2**

```powershell
wsl --set-version <distribution name> <versionNumber>
```

若要指定运行 Linux 发行版的 WSL 版本（1 或 2），请将 `<distribution name>` 替换为发行版的名称，并将 `<versionNumber>` 替换为 1 或 2。

**设置默认 WSL 版本**

```powershell
wsl --set-default-version <Version>
```

若要将默认版本设置为 WSL 1 或 WSL 2，请将 `<Version>` 替换为数字 1 或 2，表示对于安装新的 Linux 发行版，你希望默认使用哪个版本的 WSL。 例如，`wsl --set-default-version 2`。

**设置默认 Linux 发行版**

```powershell
wsl --set-default <Distribution Name>
```

若要设置 WSL 命令将用于运行的默认 Linux 发行版，请将 `<Distribution Name>` 替换为你首选的 Linux 发行版的名称。

**更新 WSL**

```powershell
wsl --update
```

将 WSL 版本更新到最新版本。 选项包括：

* `--web-download`：从 GitHub 而不是 Microsoft Store 下载最新更新。

**检查 WSL 状态**

```powershell
wsl --status
```

查看有关 WSL 配置的常规信息，例如默认发行版类型、默认发行版和内核版本。

**设置默认 WSL 版本**

```powershell
wsl --set-default-version <Version>
```

检查有关 WSL 及其组件的版本信息。

**Help 命令**

```powershell
wsl --help
```

查看 WSL 中可用的选项和命令列表。

**关闭**

```powershell
wsl --shutdown
```

立即终止所有正在运行的发行版和 WSL 2 轻量级实用工具虚拟机。 在需要重启 WSL 2 虚拟机环境的情形下，例如更改内存使用限制或更改 .wslconfig 文件，可能必须使用此命令。

**将目录更改为主页**

```powershell
wsl ~
```

`~` 可与 wsl 一起使用，以在用户的主目录中启动。 若要在 WSL 命令提示符中从任何目录跳回到主目录，可使用命令 `cd ~`。

</details>

# 使用 Windows 终端

> 使用 Windows 终端支持你想要安装的任意数量的命令行，并允许你在多个标签或窗口窗格中打开它们并在多个 Linux 发行版或其他命令行（PowerShell、命令提示符、PowerShell、Azure CLI 等）之间快速切换。

不用当然没有关系，但还是比较推荐使用Windows 终端。

在微软商店中搜索 Windows Terminal ，找到并下载即可。

![1677745141962](1677745141962.png)

之后想要打开，可以 `win + R` ，调出运行窗口,输入 `wt` 命令打开。

打开终端后，可以直接从标签栏的菜单中找到Ubuntu。

![1677745472986](1677745472986.png)

点开后如果出现错误 `[出现错误 2147942402 (0x80070002) (启动“ubuntu2004.exe”时)]` ，可能是环境变量的问题，将下面环境变量添加到用户变量即可。

```
%USERPROFILE%\AppData\Local\Microsoft\WindowsApps
```

# 使用图形化界面

## 下载并安装 **VcXsrv**

下载地址：[VcXsr](https://sourceforge.net/projects/vcxsrv/)

国内下载可能比较慢，耐心等待下载完毕。

下载完毕后安装选项按自己需求选择。

![1677746649453](1677746649453.png)

完成后打开 **XLaunch**，出现窗口，本人选择 `One large window` ，可以根据自己需求选择。

![1677746863888](1677746863888.png)

直接下一页

![1677746926493](1677746926493.png)

勾选 `Disable access control` ，下一页

![1677746981129](1677746981129.png)


选择 `Save configuration` 保存配置，下次可以直接点击配置文件打开不用重新一步步设置。

![1677747032011](1677747032011.png)

点击完成，出现黑色窗口，暂时不用管他。

## 安装 **xfce4**

在终端中打开 Ubuntu ，接下来的操作都在 Ubuntu进行。

输入 `sudo apt update ` 更新一下包的信息。

更新完后输入 `sudo apt install xfce4 ` 安装xfce4

> xfce4 国内下载不了可以先换国内镜像源，网上有很多教程，这里就不赘述了。

## 配置 IP

> 修改文件 `~/.bashrc` 。

WSL2 是使用虚拟机实现的，和 Win32 系统有不同的ip了，`export DISPLAY`和 `PULSE_SERVER`中的 IP 地址需要进行修改，格式如下：

```
export DISPLAY=`cat /etc/resolv.conf | grep nameserver | awk '{print $2}'`:0
export PULSE_SERVER=`cat /etc/resolv.conf | grep nameserver | awk '{print $2}'`
```

`PULSE_SERVER`一句是关于 PulseAudio 声音支持的，不需要可以删掉。

输入 `nano ~/.bashrc` ，添加到末尾即可。

![1677748050105](1677748050105.png)

添加完后，按 `ctrl + X` ，回车保存即可完成修改。

修改完毕后，输入 `source ~/.bashrc` 重新载入。

## 启动 xfce4

在 Ubuntu 终端输入 `startxfce4` 

如果出现以下报错

![1677748525091](1677748525091.png)

说明是权限问题，将命令改为 `sudo startxfce4` 即可。

启动后，之前 VcXsrv 的黑色窗口也出现了画面。

![1677748686966](1677748686966.png)



---

<!-- Q.E.D. -->
