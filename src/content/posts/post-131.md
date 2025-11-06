---
title: "WSL2相关操作"
published: 2025-09-23
updated: 2025-09-23
description: ""
category: "VPS相关"
draft: false
---

# 常用的 WSL 管理命令

以下是在 Windows PowerShell 或 CMD 中管理 WSL (Windows Subsystem for Linux) 的一些常用命令。

### 1. 查看已安装的 WSL 发行版

这个命令会列出所有你已经安装的 Linux 发行版，并用 `(默认)` 标记出默认启动的那个。

```powershell
wsl -l
```


### 2. 查看 WSL 及内核版本

获取 WSL 核心组件的详细版本信息，这在排查问题时非常有用。

```powershell
wsl --version
```

### 3. 查看正在运行的发行版

只显示当前处于“正在运行”状态的发行版。

```powershell
wsl -l --running
```

### 4. 立即关闭所有 WSL 实例

这个命令会强制关闭所有正在运行的 Linux 发行版和 WSL 2 轻量级实用工具虚拟机。当你觉得 WSL 行为异常或需要释放资源时可以使用。

```powershell
wsl --shutdown
```

### 5. 备份 (导出) 发行版

将指定的发行版完整地打包成一个 `.tar` 文件，用于备份或迁移。在执行此操作前，最好先使用 `wsl --shutdown` 确保该发行版已停止运行。

```powershell
wsl --export <DistributionName> <FilePath.tar>
```

**示例:**
```powershell
wsl --export Ubuntu-22.04 D:\\temp\\Ubuntu-22.04.tar
```

### 6. 注销 (卸载) 发行版

**注意：这是一个危险操作！** 它会从系统中彻底删除指定的 Linux 发行版及其包含的所有数据，并且**无法恢复**。操作后，所有文件、安装的软件和配置都会丢失。

```powershell
wsl --unregister <DistributionName>
```

**示例:**
```powershell
wsl --unregister Ubuntu-22.04
```

<hr style=\"height:4px; border:none; background-image:linear-gradient(to right, red, orange, yellow, green, blue, indigo, violet);\"> </hr>

### 配置代理

你现在使用的 `export` 命令只在当前终端窗口有效。关闭窗口后设置就没了。为了让你每次打开 WSL2 都能自动配置好代理，你需要把设置写入到 shell 的配置文件中。

但还有一个问题：`172.29.224.1` 这个 IP 在你重启电脑或重启 WSL 服务后**有可能会改变**。

所以，我们需要一个能**自动获取**这个真实 IP 并设置代理的脚本。下面是更稳健的方法：

1. 打开你的 shell 配置文件。如果你用的是默认的 bash，就运行：
 ```bash
 nano ~/.bashrc
 ```
 (如果你用的是 zsh，就是 `nano ~/.zshrc`)

2. 在文件的最底部，添加下面这几行代码：
 ```bash
 # WSL2 Proxy configuration
 export hostip=$(ip route | grep default | awk \'{print $3}\')
 export ALL_PROXY=\"http://$hostip:7890\"
 export http_proxy=\"http://$hostip:7890\"
 export https_proxy=\"http://$hostip:7890\"
 ```
 *说明：`ip route | grep default | awk \'{print $3}\'` 这条命令比之前从 `/etc/resolv.conf` 读取要可靠得多，它会直接从路由表里找到 Windows 主机的 IP。*

3. 按 `Ctrl+O` 保存，回车确认，然后按 `Ctrl+X` 退出编辑器。

4. 让配置立即生效，运行：
 ```bash
 source ~/.bashrc
 ```

现在，每次你打开一个新的 WSL2 终端，它都会自动找到正确的 Windows IP 并为你设置好代理。你可以随时通过 `curl -I https://www.google.com` 来验证。

<hr style=\"height:4px; border:none; background-image:linear-gradient(to right, red, orange, yellow, green, blue, indigo, violet);\"></hr>

### 步骤 1：在你的 Debian (WSL2) 中安装并启动 SSH 服务

默认情况下，WSL2 系统为了安全和轻量，通常不会自带或自启动 SSH 服务。你需要手动安装和开启它。

1. **更新软件列表并安装 SSH Server**
 在你的 Debian 终端里运行：
 ```bash
 sudo apt update
 sudo apt install openssh-server
 ```

2. **启动 SSH 服务**
 安装完成后，SSH 服务不会自动启动，需要手动开启。
 ```bash
 sudo service ssh start
 ```
 你可以用下面的命令来检查 SSH 服务的状态，如果看到 `active (running)` 就表示成功了。
 ```bash
 sudo service ssh status
 ```

3. **(可选但推荐) 配置 SSH 服务**
 默认配置通常就能用，但如果你想修改端口（默认为22）或者禁止密码登录等，可以编辑配置文件：
 ```bash
 sudo nano /etc/ssh/sshd_config
 ```
 修改后记得重启服务：`sudo service ssh restart`。

### 步骤 2：在 FinalShell 中配置连接

现在你的 WSL2 已经准备好了，打开 FinalShell 进行设置。

1. 新建一个连接（选择 SSH 连接）。
2. 填写连接信息：
 * **名称：** 随便填，方便你自己识别，比如 `My WSL Debian`。
 * **主机 / IP 地址：** 填写 `localhost` 或者 `127.0.0.1`。
 > **为什么是 localhost?**
 > WSL2 的一个很棒的特性是，它内部运行的服务端口会自动映射到你的 Windows 主机上。所以你可以直接通过 Windows 的 `localhost` 来访问 WSL2 里的服务。你不需要去查找 WSL2 的内部动态 IP 地址。
 * **端口：** `22` (除非你在上一步修改了 SSH 服务的默认端口)。
 * **用户名：** 你在 Debian (WSL2) 中的用户名 (就是在终端 `@` 前面的那个，比如 `heiyu`)。
 * **密码：** 你在 Debian (WSL2) 中为该用户设置的密码。

3. 点击“连接”，如果一切顺利，FinalShell 就能成功连接到你的 WSL2 系统了。

---

**一个重要提醒：**

每次你的 WSL2 完全关闭并重新启动后（比如你重启了电脑，或者手动执行了 `wsl --shutdown`），SSH 服务**不会**自动启动。你需要再次进入 Debian 终端手动运行 `sudo service ssh start`。

如果你希望它每次都自动启动，可以将启动命令加入到你的 shell 配置文件中，例如 `~/.bashrc`：

1. 编辑文件: `nano ~/.bashrc`
2. 在文件末尾添加:
 ```bash
 # Start SSH server if not running
 if ! pgrep -x \"sshd\" > /dev/null
 then
 sudo service ssh start
 fi
 ```
3. 保存文件并退出。这样每次打开新的终端时，它都会检查并启动 SSH 服务。

<hr style=\"height:4px; border:none; background-image:linear-gradient(to right, red, orange, yellow, green, blue, indigo, violet);\"> </hr>

### **目标**

将一个基于普通用户创建的 Linux 系统（如默认的 WSL 或云主机），改造为允许并默认使用 `root` 用户通过 SSH 登录，并删除原有的普通用户。

### **操作流程**

#### 步骤 1: 从普通用户切换到 Root 权限

首先，你需要临时的 Root 权限来执行系统级别的修改。

```bash
# 使用 sudo su 命令切换到 root 用户环境。
# 这需要你输入当前用户的密码。
sudo su
```
执行成功后，你的命令提示符会从 `$` 变为 `#`，表示你现在是 Root 用户。

#### 步骤 2: 设置 Root 用户的密码

默认情况下，`root` 用户可能没有设置密码或被锁定，无法直接登录。你需要为它创建一个密码。

```bash
# 在 root 环境下 (#)，执行 passwd 命令
passwd
```
系统会提示你输入新的 `root` 密码并确认。**请务必设置一个强密码并牢记它**，因为这是你以后登录系统的唯一凭证。

#### 步骤 3: 修改 SSH 配置以允许 Root 登录

接下来，修改 SSH 服务的配置文件，允许 `root` 用户通过密码远程登录。

1. **编辑配置文件**：
 ```bash
 # 使用 nano 或 vim 编辑器打开 sshd_config 文件
 nano /etc/ssh/sshd_config
 ```

2. **找到并修改 `PermitRootLogin`**：
 在文件中找到 `#PermitRootLogin prohibit-password` 这一行（或者类似的一行）。
 * 去掉行首的 `#` 注释符。
 * 将 `prohibit-password` 修改为 `yes`。

 修改后的最终结果应该是：
 ```ini
 PermitRootLogin yes
 ```

3. **保存并退出**：
 在 `nano` 中，按 `Ctrl + X`，然后按 `Y` 确认保存，最后按 `Enter` 退出。

4. **重启 SSH 服务**：
 为了让刚才的修改生效，必须重启 SSH 服务。
 ```bash
 # 这个命令适用于大多数现代 Linux 发行版
 service ssh restart
 ```

#### 步骤 4: 删除原有的普通用户

现在 `root` 已经可以登录了，你可以安全地删除原来的用户（例如 `heiyu`）。

**重要提示**：建议先**新开一个终端窗口**，用 `root` 用户和新设置的密码测试一下 SSH 登录是否成功。确认成功后再执行删除操作，以防万一。

1. **确保新终端已用 `root` 登录成功**。

2. **在 `root` 会话中，终止原用户的所有进程**：
 如果原用户还存在登录会话或后台进程，删除时会报错。用 `pkill` 命令可以干净地终止它们。
 ```bash
 # 将 <username> 替换为你要删除的用户名，例如 heiyu
 pkill -u <username>
 ```

3. **删除用户及其家目录**：
 使用 `userdel -r` 命令来彻底删除用户和其相关文件。
 ```bash
 # -r 参数会同时删除用户的家目录 (/home/<username>)
 userdel -r <username>
 ```
 执行过程中可能会出现 `mail spool not found` 的提示，这是一个正常的警告，表示用户的邮件文件本就不存在，可以忽略。

---

### **总结**

至此，你已经成功地：
1. 为 `root` 用户设置了密码。
2. 配置了 SSH 服务，允许 `root` 直接登录。
3. 彻底删除了原来的普通用户。

你的系统现在是一个只能由 `root` 用户登录的“纯净”环境了。

**安全提醒**：在生产环境或任何暴露在公网的服务器上，**强烈不推荐**允许 `root` 用户通过密码直接登录，这会带来安全风险。更安全的做法是使用密钥登录，并禁用密码登录。但在 WSL 或个人实验环境中，这种配置是方便快捷的。
