# 💬 宇宙公司聊天室（chatroom）

一个用 Go 写的**轻量级网页聊天室**，单个程序文件就能运行，不需要安装数据库，也不需要配置网页服务器。小内存 VPS、Windows 电脑、Pterodactyl 类面板（比如 Altare）上都可以部署。

---

## 📑 目录

- [功能特点](#-功能特点)
- [项目文件说明](#-项目文件说明)
- [第一步：获取程序文件（编译）](#-第一步获取程序文件编译)
- [第二步：准备邀请码](#-第二步准备邀请码)
- [第三步：部署运行（三选一）](#-第三步部署运行三选一)
  - [方式 A：Linux 服务器 / VPS](#方式-alinux-服务器--vps)
  - [方式 B：Windows 电脑](#方式-bwindows-电脑)
  - [方式 C：Altare 等面板（Pterodactyl）](#方式-caltare-等面板pterodactyl)
- [第四步：登录使用](#-第四步登录使用)
- [日常管理](#-日常管理)
- [配置项（环境变量）](#-配置项环境变量)
- [常见问题](#-常见问题)

---

## ✨ 功能特点

- **两层登录**
  - 第一层：**邀请码**。没有邀请码的人进不了聊天室。
  - 第二层：**昵称和个人密码**。新昵称第一次登录时自动注册，之后再登录要输入同一个密码。
- **公开频道和私聊**：在左侧通讯录点任意一个人就能私聊，私聊记录只有你们两个人看得到。
- **@提醒**：在公开频道 @某人，对方会收到提醒，点铃铛可以跳到那条消息。
- **文件和图片**：单个文件最大 100MB。图片在聊天里直接预览，其他文件点击下载。
- **未读定位（类似微信）**
  - 进入一个会话后，页面自动停在“上次读到的位置”，第一条未读消息上方会显示“—— 以下为新消息 ——”。
  - 你往上翻看旧消息时有新消息进来，底部会出现 **“↓ N 条新消息”**，点一下回到最新。
  - 第一条未读滑出屏幕后，顶部会出现 **“↑ 回到第一条未读”**。
  - 一次最多加载 300 条未读，超出的部分点“加载更多”往上补。
- **邀请码热更新**：编辑 `invite_codes.txt` 后大约 10 秒自动生效，**不用重启**。
- **断线自动重连**，手机浏览器上也能正常使用。

---

## 📁 项目文件说明

```
chatroom/
├── main.go                  ← 后端程序源码
├── static/
│   └── index.html           ← 网页界面（编译时会打包进程序）
├── go.mod                   ← Go 依赖说明
├── invite_codes.txt         ← 邀请码列表（一行一个）
└── .github/workflows/
    └── build.yml            ← GitHub 自动编译脚本（需要手动触发）
```

程序第一次运行后，会在**运行目录**下自动生成下面这些文件和文件夹：

| 名称 | 作用 |
|---|---|
| `chat.db` | 数据库，保存账号、聊天记录和邀请码状态 |
| `uploads/` | 用户上传的文件 |

> 💡 网页界面已经打包进程序，**运行时不需要 `static` 文件夹**。不过改了 `index.html` 以后要**重新编译**才能生效。

---

## 🔨 第一步：获取程序文件（编译）

> 最推荐**方法一**：电脑上什么都不用装，在 GitHub 网页上点几下就能拿到程序。

### 方法一：用 GitHub Actions 在线编译（推荐新手）

1. 打开本仓库页面，点右上角的 **Fork**，复制一份到你自己的账号下。如果仓库就是你自己的，跳过这一步。
2. 进入仓库，点上方的 **Actions** 标签。
   - 第一次打开时如果看到一个绿色按钮 **“I understand my workflows, go ahead and enable them”**，点它启用 Actions。
3. 在左侧点 **Build Chatroom**。
4. 点右侧的 **Run workflow**，在弹出的框里再点绿色的 **Run workflow**。
5. 等 2～5 分钟，这次运行的记录前面出现绿色 ✅ 就是编译好了。
6. 点进这条运行记录，拉到页面最底部的 **Artifacts**，下载你需要的版本：

| 文件名 | 适用系统 | 解压后得到 |
|---|---|---|
| `chatroom-linux-amd64` | Linux（Ubuntu、Debian、CentOS、Alpine 等） | `chatroom` |
| `chatroom-windows-amd64` | Windows 10 / 11 | `chatroom.exe` |

> 📌 Actions **不会**在你每次修改代码后自动编译，只有你手动点 Run workflow 才会编译。
> 📌 编译出来的程序是**纯静态程序**，不依赖 gcc 或其他系统库，下载下来就能直接运行。

### 方法二：自己用 Go 编译

先安装 **Go 1.24 或更新的版本**（下载地址：https://go.dev/dl/ ），然后在项目目录里执行：

```bash
go mod tidy                      # 下载依赖（只需要执行一次）
go build -o chatroom .           # Linux / macOS
go build -o chatroom.exe .       # Windows
```

> 如果在国内下载依赖很慢，先执行这一句：`go env -w GOPROXY=https://goproxy.cn,direct`

---

## 🎟️ 第二步：准备邀请码

编辑 `invite_codes.txt`，**一行写一个邀请码**。以 `#` 开头的行是注释，空行会被忽略：

```text
# 这是注释
Kx9mPq2vLt
Hd83nZwq7R
# Old-Code-123    ← 前面加了 #，这个邀请码就停用了
```

⚠️ **注意事项**

- 邀请码就是聊天室的“门禁卡”，**请用长一点的随机字符串**（建议 10 位以上，字母和数字混合）。
- **每个邀请码第一次登录成功后，会永久绑定到那个昵称**：别人拿这个码不能用其他昵称登录。一个昵称也只能绑定一个邀请码。所以**每个人发一个邀请码**。
- `invite_codes.txt` 要和程序文件放在**同一个目录**里。

---

## 🚀 第三步：部署运行（三选一）

下面三种方式里，**`PORT` 都是指网站的端口号**，可以换成你自己想用的端口，比如 `25266`。如果不设置，默认是 `8080`。

---

### 方式 A：Linux 服务器 / VPS

**1. 上传文件**

在服务器上新建一个目录，比如 `/root/chatroom`，把下面两个文件放进去：

```
/root/chatroom/
├── chatroom            ← 编译好的程序
└── invite_codes.txt    ← 邀请码
```

可以用 FinalShell、WinSCP 之类的工具上传，也可以在本地电脑上执行：

```bash
scp ./chatroom ./invite_codes.txt root@你的服务器IP:/root/chatroom/
```

**2. 先试运行一次**

```bash
cd /root/chatroom
chmod +x chatroom          # 给程序运行权限（只需要执行一次）
PORT=25266 ./chatroom      # 启动
```

看到 `Running on :25266` 就说明启动成功了。用浏览器打开 `http://你的服务器IP:25266` 试一下，确认没问题后按 `Ctrl + C` 停止，再做下一步。

**3. 让它在后台一直运行（二选一）**

**▶ 简单方式：nohup**

```bash
cd /root/chatroom
PORT=25266 nohup ./chatroom > chat.log 2>&1 &
```

常用命令：

```bash
tail -f chat.log            # 查看运行日志（按 Ctrl + C 退出查看）
pkill chatroom              # 停止程序
ps aux | grep chatroom      # 查看是否还在运行
```

**▶ 推荐方式：systemd（开机自动启动，崩溃后自动重启）**

新建一个服务配置文件：

```bash
nano /etc/systemd/system/chatroom.service
```

把下面的内容粘贴进去，保存并退出（`Ctrl + O` 保存，回车确认，再按 `Ctrl + X` 退出）：

```ini
[Unit]
Description=Chatroom
After=network.target

[Service]
WorkingDirectory=/root/chatroom
ExecStart=/root/chatroom/chatroom
Environment=PORT=25266
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

启用并启动服务：

```bash
systemctl daemon-reload
systemctl enable --now chatroom     # 设置开机自启，并马上启动
systemctl status chatroom           # 查看运行状态
journalctl -u chatroom -f           # 查看运行日志
systemctl restart chatroom          # 重启
systemctl stop chatroom             # 停止
```

**4. 放行防火墙端口**

如果打不开网页，通常是端口没放行。根据你的系统执行对应的命令：

```bash
ufw allow 25266/tcp                                     # Ubuntu / Debian
firewall-cmd --add-port=25266/tcp --permanent && firewall-cmd --reload   # CentOS
```

另外，云服务器（阿里云、腾讯云等）还要在网页控制台的 **安全组** 里放行这个端口。

---

### 方式 B：Windows 电脑

**1. 准备文件夹**

新建一个文件夹，比如 `D:\chatroom`，把下面两个文件放进去：

```
D:\chatroom\
├── chatroom.exe
└── invite_codes.txt
```

**2. 创建启动脚本**

在这个文件夹里新建一个文本文件，粘贴下面的内容，然后**另存为 `start.bat`**。保存时“保存类型”选“所有文件”，编码选 **ANSI**。

```bat
@echo off
cd /d %~dp0
set PORT=25266
chatroom.exe
pause
```

**3. 双击 `start.bat` 启动**

- 看到 `Running on :25266` 就说明启动成功了。**这个黑色窗口不能关**，关掉聊天室就停了。
- 第一次运行时如果 Windows 防火墙弹窗询问，选 **允许访问**。

**4. 访问地址**

| 从哪里访问 | 地址 |
|---|---|
| 本机 | `http://localhost:25266` |
| 同一个 Wi-Fi 下的手机或其他电脑 | `http://本机IP:25266`（本机 IP 的查法：打开 CMD 输入 `ipconfig`，看“IPv4 地址”那一行） |

如果局域网里的其他设备打不开，**右键“以管理员身份运行”CMD**，执行下面这条命令放行端口：

```bat
netsh advfirewall firewall add rule name="chatroom" dir=in action=allow protocol=TCP localport=25266
```

**5.（可选）开机自动启动**

按 `Win + R`，输入 `shell:startup` 回车，会打开“启动”文件夹。把 `start.bat` 的**快捷方式**放进去，以后登录 Windows 就会自动启动聊天室。

---

### 方式 C：Altare 等面板（Pterodactyl）

适用于 Altare 这类基于 Pterodactyl 的面板，服务器镜像要选 **Go**（比如 `go_1.25`）。

> 这类面板有几个特点要先知道：
> - 只有 `/home/container` 这个目录可以写入文件。
> - 启动命令**不能直接写 `export`**，而且服务器上**没有 gcc**。
> - 下面的脚本已经把这些情况都处理好了，照着做就行。

**1. 上传文件**

在面板的“文件管理”里上传下面这些文件，目录结构要保持一致：

```
main.go
go.mod
invite_codes.txt
static/index.html
start.sh            ← 第 2 步新建
```

**2. 新建 `start.sh`**

在文件管理里新建文件 `start.sh`，粘贴下面的内容，**把 `PORT` 改成面板分配给你的端口**：

```sh
#!/bin/sh
export GOPATH=/home/container/.go
export GOMODCACHE=/home/container/.go/pkg/mod
export GOCACHE=/home/container/.cache/go-build
export GOPROXY=https://goproxy.cn,direct
export GOSUMDB=sum.golang.google.cn
export CGO_ENABLED=0
export PORT=25266

if [ ! -f ./chatroom ]; then
  go mod tidy || exit 1
  go build -o chatroom . || exit 1
fi
exec ./chatroom
```

**3. 修改启动命令**

进入面板的 **Startup（启动）** 页面，把启动命令改成：

```
sh start.sh
```

**4. 点“启动”**

- 第一次启动会自动下载依赖并编译，需要等几分钟。日志里出现 `Running on :你的端口` 就说明成功了。
- 以后再启动时，因为 `chatroom` 程序文件已经存在，会直接运行，不再编译。
- 💡 **修改了 `main.go` 或 `index.html` 之后**：先在文件管理里**删掉 `chatroom` 这个文件**，再重启，脚本会自动重新编译。
- 💡 如果实例内存太小，编译时程序被系统杀掉，可以改用“第一步 方法一”编译好的 `chatroom-linux-amd64`：把解压出来的 `chatroom` 直接上传，`start.sh` 发现文件已存在会跳过编译。如果提示权限不足，在 `exec ./chatroom` 前面加一行 `chmod +x ./chatroom`。

**5.（可选）绑定自己的域名**

在面板的“域名”页面把你的域名连接到这个服务器的端口就行，面板会自动配置 HTTPS。
页面上显示的 `ns1.altare.global` 这类 NS 地址，只有在你想**把整个域名的解析交给 Altare 托管**时才需要设置。**只是连接一个子域名的话，不用改 NS。**

---

## 🙋 第四步：登录使用

1. 浏览器打开 `http://你的IP:端口`，绑定了域名的话打开 `https://你的域名`。
2. 输入**邀请码**，点“进入聊天室”。浏览器会记住邀请码 7 天。
3. 输入**昵称**和**个人密码**：
   - 昵称：2～15 位，可以用中文、字母、数字、下划线 `_`、中划线 `-`。
   - 新昵称会**自动注册**，请记住密码，以后用同一个昵称登录必须输入这个密码。
4. 进去以后：
   - 左侧（手机上点左上角 ☰）是**通讯录**，点“公开频道”或任意一个人切换会话。
   - 输入框左边的 **+** 用来发送文件。
   - 在公开频道输入 **@** 可以选人提醒。
   - 点击消息上的头像或昵称，可以直接和这个人私聊。

---

## 🛠️ 日常管理

### 增加、停用、恢复邀请码（不用重启）

直接编辑 `invite_codes.txt` 并保存，**大约 10 秒后自动生效**：

| 想做什么 | 怎么改 |
|---|---|
| 增加邀请码 | 新加一行 |
| 停用邀请码 | 删掉那一行，或者在行首加 `#` |
| 恢复邀请码 | 把那一行加回来，或者去掉行首的 `#` |

- 邀请码停用后，用这个码的人**刷新页面或重新连接时**会被挡在门外。
- 为了防止误操作，如果文件被清空，程序**不会**停用任何邀请码。

### 备份数据

**先停止程序**，再复制 `chat.db` 文件和 `uploads/` 文件夹。恢复的时候放回原位置就行。

### 清空所有数据或重置

停止程序后删除 `chat.db`，所有账号、聊天记录和邀请码绑定关系都会被清空。下次启动时会重新从 `invite_codes.txt` 导入邀请码。
如果还想删掉已上传的文件，再删除 `uploads/` 文件夹。

> ⚠️ 删除后数据无法恢复，请先备份。

### 更新到新版本

1. 停止程序。
2. 用新编译的程序替换旧的程序文件。`chat.db` **不用删**，新版本会自动兼容旧数据。
3. 启动程序。

---

## ⚙️ 配置项（环境变量）

| 变量名 | 作用 | 默认值 | 示例 |
|---|---|---|---|
| `PORT` | 网站端口 | `8080` | `25266` |
| `DATABASE_PATH` | 数据库文件的位置 | `./chat.db` | `/data/chat.db` |

设置方法：
- Linux：`PORT=25266 ./chatroom`
- Windows：在 `start.bat` 里写 `set PORT=25266`

> 程序会从**运行目录**读取 `invite_codes.txt`，`uploads/` 文件夹也会建在运行目录下。所以请在程序所在的目录里启动它（上面的脚本都已经处理好了）。

---

## ❓ 常见问题

**Q：打开网页显示“无法访问此网站”？**
A：按顺序排查：① 程序日志里有没有 `Running on :端口`；② 防火墙或云服务器安全组有没有放行这个端口；③ 地址里的端口号有没有写对。

**Q：输入邀请码后提示“邀请码无效或已被禁用”？**
A：检查三点：① `invite_codes.txt` 里有没有这个码，前面有没有 `#`；② 文件是不是和程序在同一个目录；③ 修改后有没有等 10 秒。

**Q：提示“当前邀请码已绑定其他昵称”？**
A：这个码已经被别人用过了，邀请码是一人一个的。请给这个人发一个新的邀请码。

**Q：提示“密码错误或昵称被占用”？**
A：这个昵称已经有人注册过了。如果是你自己的账号，请输入当初设置的密码；否则换一个昵称。

**Q：提示“此账号已在别处登录”？**
A：同一个昵称同一时间只能在一个地方在线。先关掉另一个设备上的页面，再重新登录。

**Q：Altare 面板报 `gcc not found`？**
A：说明你用的是旧版代码，或者 `start.sh` 里写的还是 `CGO_ENABLED=1`。请使用本仓库的最新代码，并按上面的 `start.sh` 设置为 `CGO_ENABLED=0`。

**Q：Altare 面板报 `missing go.sum entry` 或 `/go/pkg/sumdb ... no such file`？**
A：这是因为没有用 `start.sh` 启动，或者启动命令里直接写了 `export`。请按“方式 C”设置，把启动命令改成 `sh start.sh`。

**Q：改了 `index.html` 但网页没变化？**
A：网页是编译时打包进程序的，所以要**重新编译**。Altare 上删掉 `chatroom` 文件后重启就会重新编译；其他环境需要重新编译程序，再替换旧文件。

**Q：上传的 `.html`、`.svg` 文件为什么点开是下载，而不是直接打开？**
A：这是安全设计，防止有人上传带恶意脚本的文件。只有 png、jpg、gif、webp、bmp 图片可以在浏览器里直接预览。

**Q：想让外网用户安全访问？**
A：建议在聊天室前面加一层 HTTPS。可以用面板自带的域名功能，或者 Cloudflare 的代理或 Tunnel、Nginx 反向代理加证书。程序检测到 HTTPS 后，会自动给登录凭证（Cookie）加上安全标记。
