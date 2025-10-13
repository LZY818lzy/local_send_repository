# 第10章 Linux 下的库

## 问题解答：

### 问题1：如何上传vscode里的文件到Linux里？

- **方法一：使用SCP命令（推荐）**

在VS Code的终端或本地系统的终端中执行：

```li
scp test.cpp username@linux_server_ip:/path/to/destination/
```

```
scp test.cpp user@192.168.1.100:/home/user/code/
```

如果需要指定端口：

```
scp -P 2222 test.cpp user@192.168.1.100:/home/user/code/
```

- **方法二：使用SFTP**

1. 安装SFTP扩展：

   - 在VS Code中搜索并安装 "SFTP" 扩展

2. 配置SFTP：

   - 按 `Ctrl+Shift+P`，输入 "SFTP: Config"

   - 在生成的 `sftp.json` 文件中配置：

     json

     ```
     {
         "host": "your_linux_server_ip",
         "username": "your_username",
         "password": "your_password",
         "protocol": "sftp",
         "port": 22,
         "remotePath": "/home/your_username/code",
         "uploadOnSave": true
     }
     ```

   - 右键文件选择"Upload" 或使用SFTP面板上传

- **方法三：使用VS Code Remote - SSH扩展**

1. 安装Remote - SSH扩展
2. 连接到Linux服务器：
   - 按 `F1`，输入 `"Remote-SSH: Connect to Host"`
   - 添加你的服务器：`user@hostname`
3. 直接在远程环境中工作，文件会自动同步

- **方法四：使用Git**

如果Linux服务器有Git仓库：

```
git add test.cpp
git commit -m "Add test.cpp"
git push origin main
```

然后在Linux服务器上拉取更新。

- **方法六：手动复制粘贴**

对于小文件：

1. 在VS Code中打开test.cpp
2. 全选复制 (`Ctrl+A`, `Ctrl+C`)
3. 通过SSH连接到Linux服务器
4. 创建文件并粘贴内容

------

### 问题2：Visual Studio 2022自动上传文件到Linux里面

#### **方法1：使用 Visual Studio 的 SSH 支持**

配置 SSH 连接

1. **打开项目** → 右键点击项目 → **属性**
2. 在 **调试** 选项卡中：
   - 选择 **远程调试** 作为调试器
   - 配置 SSH 连接信息：
     - 主机名：Linux 服务器 IP
     - 端口：22（默认）
     - 用户名：你的 Linux 用户名
     - 认证：密码或私钥文件

使用远程文件管理器

1. **视图** → **Cloud Explorer**
2. 添加 SSH 连接
3. 可以直接拖拽文件/文件夹到远程目录

#### **方法2：使用 SFTP 扩展**

安装 SFTP 扩展

1. **扩展** → **管理扩展**
2. 搜索安装：
   - "SFTP"
   - "SSH FS"

配置 SFTP 设置

在项目根目录创建 `sftp.json`：

json

```
{
    "name": "My Linux Server",
    "host": "your-linux-ip",
    "protocol": "sftp",
    "port": 22,
    "username": "your-username",
    "password": "your-password",
    "remotePath": "/home/your-username/project",
    "uploadOnSave": true,
    "ignore": ["**/.vscode/**", "**/.git/**"]
}
```

#### **方法3：使用 Git 与自动部署**

设置 Git 钩子

在 Linux 服务器上配置 Git 钩子自动同步：

bash

```
# 在服务器上创建 Git 仓库
git init --bare your-project.git
cd your-project.git/hooks
cat > post-receive << 'EOF'
#!/bin/bash
TARGET="/home/your-username/deploy"
GIT_DIR="/home/your-username/your-project.git"
BRANCH="main"

while read oldrev newrev ref
do
    if [[ $ref = refs/heads/$BRANCH ]];
    then
        echo "Ref $ref received. Deploying ${BRANCH} branch to production..."
        git --work-tree=$TARGET --git-dir=$GIT_DIR checkout -f
    else
        echo "Ref $ref received. Doing nothing: only the ${BRANCH} branch may be deployed on this server."
    fi
done
EOF

chmod +x post-receive
```

#### **方法4：使用 Rsync 脚本**

创建部署脚本

在 VS 2022 中添加生成后事件：

batch

```
# 在项目属性 → 生成事件 → 后期生成事件
rsync -avz --delete $(ProjectDir) user@linux-server:/path/to/destination/
```

或使用 PowerShell 脚本：

powershell

```
$source = "C:\your\project\path\"
$destination = "user@linux-server:/home/user/project/"
& rsync -avz --delete $source $destination
```

- **推荐方案**

对于日常开发，我推荐：

1. **开发阶段**：使用 SFTP 扩展（方法2），设置 `uploadOnSave: true`
2. **生产部署**：使用 Git 自动部署（方法3）或 Rsync 脚本