# VS Code在Linux上的自动化部署

## 一、准备工作

### 1、确保本地已安装`rsync`（Windows 需通过 WSL安装，Linux 自带）

#### （1）**核心前提：确保「本地 Windows 环境」能运行 rsync 命令**

1. ##### **启用 WSL** 功能：

   安装相关问题见后续。

2. ##### 安装 rsync：

- 打开 Windows 的 “开始” 菜单，找到 Ubuntu
- 在 WSL 的终端窗口中，输入 `rsync --version` 命令并回车。如果 rsync 已经成功安装，你会看到类似下面的输出，显示 rsync 的版本号等信息：

```bash
rsync  version 3.2.3  protocol version 31
Copyright (C) 1996-2022 by Andrew Tridgell, Wayne Davison, and others.
Web site: https://rsync.samba.org/
```

- 打开 VS Code，在菜单栏选择 **终端 > 新建终端**，确保终端类型是 WSL（终端左上角会显示 `Ubuntu` 或对应的发行版名称）。

- 在终端中输入命令：

  ```bash
  rsync --version
  ```

- 成功安装的表现：终端会输出 rsync 的版本信息（类似下方内容）：

  ```plaintext
  rsync  version 3.2.7  protocol version 31
  Copyright (C) 1996-2022 by Andrew Tridgell, Wayne Davison, and others.
  Web site: https://rsync.samba.org/
  ```

- 未安装的解决：在 WSL 终端中执行以下命令安装：

  ```bash
  sudo apt update && sudo apt install rsync -y
  ```

#### （2）**确保「Linux 服务器」上已安装 rsync**

```bash
rsync --version
```

### 2、确保本地可以通过 SSH 无密码登录服务器（避免每次同步输入密码）

#### （1）**打开 WSL 终端**

- 在 Windows 搜索栏中输入 “Ubuntu”然后点击打开，进入 WSL 终端。
- 也可以在 VS Code 中打开 WSL 终端，按下`Ctrl+Shift+P`（Windows/Linux）或`Cmd+Shift+P`（Mac），在命令面板中输入 “WSL: New Terminal” 并选择，即可打开 WSL 终端 。

#### （2）**生成 SSH 密钥**

在 WSL 终端中，执行以下命令来生成`ed25519`类型的 SSH 密钥对（你也可以使用默认的`rsa`类型，命令为`ssh-keygen -t rsa`）：

```bash
ssh-keygen -t ed25519
```

- 系统会询问你将密钥文件保存到哪里，默认路径是`~/.ssh/id_ed25519` ，<u>直接回车使用默认路径即可</u>。如果你想指定其他路径，可以输入对应的路径。
- 接下来会提示你输入密钥的密码，<u>建议直接回车，设置为空密码</u>，这样在免密登录时就无需输入密码。如果设置了密码，在每次登录时还是需要输入这个密码，就无法实现真正的免密登录了。

#### （3）**查看生成的密钥**

密钥生成完成后，会在你指定的路径下生成两个文件，以默认路径为例：

- `~/.ssh/id_ed25519`：这是私钥文件，**一定要妥善保管，不能泄露**。
- `~/.ssh/id_ed25519.pub`：这是公钥文件，后续需要将这个文件的内容上传到服务器。你可以使用以下命令查看公钥内容：

```bash
cat ~/.ssh/id_ed25519.pub
```

执行命令后，会显示一长串字符，这就是公钥的内容。

#### （4）**将公钥上传到服务器**

在 WSL 终端中，执行以下命令将公钥上传到服务器：

```bash
ssh-copy-id 用户名@服务器IP
```

其中，`用户名`是你在服务器上的登录用户名，`服务器IP`是服务器的实际 IP 地址。

执行命令后，系统会提示你输入服务器的密码（这是首次连接时需要输入，配置成功后就不需要了），输入密码并回车，公钥就会被上传到服务器，并自动添加到服务器的`~/.ssh/authorized_keys`文件中 。

#### （5）**测试免密登录**

在 WSL 终端中，执行以下命令尝试登录服务器：

```bash
ssh 用户名@服务器IP
```

如果配置成功，你将无需输入密码直接登录到服务器 。



## 二、创建 VS Code 任务配置文件

#### 1、在C++ 项目根目录下，创建`.vscode`文件夹

#### 2、创建同步任务（`tasks.json`）

- 在项目根目录的 `.vscode` 文件夹下，创建 `tasks.json` 文件，配置**同步到 Linux 服务器**的任务：
- `user@192.168.1.100`：替换为你的服务器用户名和 IP 地址
- `/home/user/projects/my_cpp_project/`：替换为服务器上的项目存放路径
- 要将整个 `Projects` 目录自动同步到 Linux 服务器，只需修改 VS Code 任务配置，指定同步整个目录即可

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Sync to Server", // 任务名称（自定义，比如“同步到服务器”）
      "type": "shell", // 执行 shell 命令
      "command": "rsync", // 核心命令：rsync
      "args": [
        "-avz", // 归档模式 + 显示详细日志 + 压缩传输（加快速度）
        "--delete", // 删除服务器上本地没有的文件（保持完全同步）
        "--exclude", ".git/", // 排除 Git 目录（本地管理，不同步到服务器）
        "--exclude", "build/", // 排除编译生成目录（本地编译，服务器无需）
        // 👇 本地项目路径（替换为你的本地路径，Windows 用 WSL/Cygwin 格式）
        "/mnt/e/Projects/", 
        // 👇 服务器路径（替换为“用户名@IP:服务器项目路径”）
        "lzy@192.168.126.128:/home/lzy/projects/" 
      ],
      "presentation": {
        "echo": true, // 显示命令执行过程
        "reveal": "always", // 同步时自动显示终端
        "focus": false, // 不自动聚焦到终端（保持代码编辑焦点）
        "panel": "shared" // 所有同步任务共用一个终端面板
      },
      "problemMatcher": [] // 不启用“问题匹配”（因为不是编译任务）
    }
  ]
}
```



## 三、使用同步功能

1. 在 VS Code 中编辑你的 C++ 代码

2. 完成修改后，按`Ctrl+Shift+P`选择

3. 此时会自动执行 rsync 命令，将修改同步到服务器

4. 在 VS Code 的终端面板中可以查看同步进度和结果

   

------



## ❓ **过程中产生的问题：**

### 1、**启用 WSL** 

- Windows 系统原生不支持 `rsync` 命令，需要通过 WSL 来安装使用`rsync`


- 首先：以管理员身份打开 “命令提示符” ，执行以下命令来启用 WSL 功能

```bash
C:\Windows\system32>dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

部署映像服务和管理工具
版本: 10.0.19041.3636

映像版本: 10.0.19044.6332

启用一个或多个功能
[==========================100.0%==========================]
操作成功完成。
```

执行完该命令后，重启计算机。

- 然后：查看可用的 Linux 发行版，在命令提示符中执行以下命令，查看 Microsoft Store 中可用的 Linux 发行版：

```bash
wsl --list --online
```

- 安装指定的 Linux 发行版：例如，要安装 `Ubuntu`，执行以下命令：

  ```bash
  wsl --install -d Ubuntu
  ```

（2）**安装 rsync**：重启后，打开安装好的 Linux 发行版（比如 Ubuntu ），在其终端中执行以下命令安装 `rsync`：

```bash
sudo apt update
sudo apt install rsync
```

- 在 VS Code 中使用 WSL 终端：

  打开 VS Code，按下Ctrl + `` （键盘左上角波浪线按键）打开终端，在终端下拉菜单中选择你安装的WSL发行版（如Ubuntu ），此时就可以在终端中使用 rsync` 命令了

### 2、**针对安装指定的 Linux 发行版 `0x80070422` 错误**

- #### 步骤 1：检查并修复 WSL 所需的 Windows 功能

以管理员身份打开 **PowerShell**（不是命令提示符，PowerShell 对 WSL 支持更友好），依次执行以下命令：

```powershell
# 启用 WSL 功能
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 启用虚拟机平台（WSL 2 需要，若用 WSL 1 可跳过，但建议都启用）
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

执行完成后，**重启电脑**，让功能生效。

- #### 步骤 2：检查 `Windows Update` 和 `BITS` 服务

1. 按 `Win + R`，输入 `services.msc` 打开 “服务” 窗口。
2. 找到Windows Update服务：
   - 若 “启动类型” 是 “禁用”，改为 “自动”，然后点击 “启动”。
   - 若已经是 “自动” 但无法启动，右键 → “属性” → “登录” 选项卡，选择 “本地系统账户”，再尝试启动。
3. 找到Background Intelligent Transfer Service (BITS)服务：
   - 同样确保 “启动类型” 为 “自动”，且服务处于 “正在运行” 状态。

- #### 步骤 3：检查网络和代理（重点！）

1. `0x80070422` 很可能是**网络连接不稳定**或**代理配置错误**导致 WSL 安装包下载失败。
2. 关闭 VPN、代理工具（如 Clash、Shadowsocks 等），确保电脑直接连接网络。
3. 尝试切换网络（比如从 Wi-Fi 换有线，或手机热点），再执行 `wsl --install -d Ubuntu`。

- #### 步骤 4：重置 Windows Update 组件（终极手段）

如果上述步骤都无效，以管理员身份打开 PowerShell，执行以下命令重置 Windows Update 组件：

```powershell
# 停止相关服务
net stop wuauserv
net stop cryptSvc
net stop bits
net stop msiserver

# 重命名缓存文件夹
ren C:\Windows\SoftwareDistribution SoftwareDistribution.old
ren C:\Windows\System32\catroot2 catroot2.old

# 启动服务
net start wuauserv
net start cryptSvc
net start bits
net start msiserver
```

执行完成后，**重启电脑**，再尝试安装 WSL。

### **3、手动下载 WSL 发行版离线包安装**

​	如果问题2中所有方法都尝试过，仍无法解决 `0x80070422` 错误安装 WSL，可能是系统组件或注册表存在深层问题，此时可以尝试以下 **更彻底的替代方案**，绕开常规安装流程：

1. **下载 Ubuntu 离线包**

   ​	访问微软官方 WSL 发行版列表：https://learn.microsoft.com/zh-cn/windows/wsl/install-manual#downloading-distributions找到 **Ubuntu 22.04 LTS** 对应的下载链接（通常是一个 `.appx` 或 `.zip` 文件），例如：https://aka.ms/wslubuntu2204

2. **安装离线包**

   - 下载后，将文件扩展名从 `.appx` 改为 `.zip`（如果是 `.zip` 则直接解压）。
   - 解压到任意目录（例如 `C:\Ubuntu`），双击运行解压目录中的 `ubuntu2204.exe`。
   - 首次运行会初始化系统，按提示设置用户名和密码，完成后即可使用 WSL。

3. **设置为 WSL 2（可选）**以管理员身份打开 PowerShell，执行：

   ```powershell
   wsl --set-version Ubuntu-22.04 2  # 将 Ubuntu 切换到 WSL 2
   wsl --set-default-version 2       # 后续安装默认使用 WSL 2
   ```

- ### 如果这里安装以后，打开ubuntu显示`0x800701bc` 错误

  1. **下载WSL2 Linux内核更新包**
     你需要从微软官方下载一个必要的更新包。
     - **官方下载地址**：https://aka.ms/wsl2kernel （此链接在多个搜索结果中被提及）或直接访问：https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi 。
     - 这个更新包是专门用于解决 `0x800701bc` 错误的。
  2. **安装更新包**
     - 找到你刚下载的 `wsl_update_x64.msi` 文件。
     - **双击运行**它，然后按照安装向导的提示完成安装。
  3. **完成安装并重启Ubuntu**
     - 内核更新包安装完成后，**重新启动你的Ubuntu应用**。
     - 现在，Ubuntu应该可以正常启动并完成最后的安装设置了。

### 4、VS Code新建终端时出现找不到WSL终端的问题？

- #### 步骤一：安装扩展WSL

- #### 步骤二：手动配置 WSL 终端

  - 按下 `Ctrl+,`（逗号）打开 VS Code 设置界面。

  - 在搜索框中输入 `terminal.integrated.profiles.windows`

  - 点击 `在 settings.json 中编辑`，添加以下配置（以 Ubuntu 为例）：

    ```json
    {
            "[plaintext]": {
                "editor.unicodeHighlight.ambiguousCharacters": false,
                "editor.unicodeHighlight.invisibleCharacters": false
            },
            "remote.SSH.remotePlatform": {
                "192.168.126.128": "linux"
            },
            "terminal.integrated.profiles.windows": {
                // 保留原有的PowerShell配置
                "PowerShell": {
                    "source": "PowerShell",
                    "icon": "terminal-powershell"
                },
                // 保留原有的Command Prompt配置
                "Command Prompt": {
                    "path": [
                        "${env:windir}\\Sysnative\\cmd.exe",
                        "${env:windir}\\System32\\cmd.exe"
                    ],
                    "args": [],
                    "icon": "terminal-cmd"
                },
                // 保留原有的Git Bash配置
                "Git Bash": {
                    "source": "Git Bash"
                },
                // 添加WSL终端配置（核心新增部分）
                "WSL: Ubuntu": {
                    "path": "wsl.exe",  // 调用WSL执行程序
                    //"path": "C:\\Windows\\System32\\wsl.exe",  // 强制指定完整路径
                    "args": ["-d", "Ubuntu"],  // 指定WSL发行版为Ubuntu
                    "icon": "terminal-ubuntu"  // 显示Ubuntu图标（可选）
                }
            }, 
            // 可选：设置WSL为默认终端（打开终端时自动使用WSL）
           "terminal.integrated.defaultProfile.windows": "WSL: Ubuntu"
        }
    ```

    PS：这里有个问题暂时没解决：

    ​	必须设置WSL为默认终端，才可以在当前本地文件夹基础上直接打开WSL终端，如果删除 `"terminal.integrated.defaultProfile.windows": "WSL: Ubuntu"`，就会自动打开`powershell`类型终端。

    ​	没办法找到deepssek说的终端可以下拉界面选择终端类型。