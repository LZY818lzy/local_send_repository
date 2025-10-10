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