# Windows WSL2 + Ubuntu 24.04 完整安装配置笔记
## 基础环境
宿主系统：Windows11
WSL版本：WSL2
Linux发行版：Ubuntu 24.04 LTS

## 一、安装WSL
1. 以**管理员身份**打开 PowerShell
![alt text](image-1.png)
wsl --install
wsl --install
## 设置WSL默认版本为WSL2
1. 以管理员身份打开 PowerShell，执行配置命令
```powershell  
wsl --set-default-version 2
一、安装 WSL
右键开始菜单，以管理员身份打开 PowerShell
执行一键安装命令
powershell
wsl --install
命令会自动开启所需 Windows 功能、默认安装 Ubuntu；执行完毕重启电脑生效
指定安装 Ubuntu 24.04（可选）
powershell
wsl --install -d Ubuntu-24.04
二、设置 WSL 默认版本为 WSL2
管理员 PowerShell 运行：
powershell
wsl --set-default-version 2
WSL 信息查看命令
powershell
# 查看已安装发行版与版本
wsl -l -v
三、Ubuntu 首次初始化
电脑重启后启动 Ubuntu 终端：
设置自定义用户名
设置登录密码
⚠️ 输入密码时终端不会显示字符，属于正常机制，输完直接回车
该密码后续执行sudo管理员操作需要使用，请牢记
四、更换国内清华软件源（解决下载缓慢）
Ubuntu 官方源国内访问速度很差，推荐替换镜像源
备份原有源文件
bash
运行
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
打开配置文件
bash
运行
sudo nano /etc/apt/sources.list
清空文件原有全部内容，粘贴下面清华源内容：
plaintext
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ noble main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ noble-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ noble-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ noble-security main restricted universe multiverse
快捷键操作：
Ctrl+O 保存 → 回车确认 → Ctrl+X 退出编辑器
更新软件索引与系统
bash
运行
sudo apt update
sudo apt upgrade -y
五、安装基础开发工具
bash
运行
# 基础工具集
sudo apt install git curl wget vim zip unzip -y

# C/C++编译环境（可选，编程开发必备）
sudo apt install build-essential gdb
六、Git 配置（对接 GitHub/Gitee）
1. 设置账号信息
bash
运行
git config --global user.name "你的用户名"
git config --global user.email "你的GitHub注册邮箱"
2. 生成 SSH 密钥（免密推拉代码）
bash
运行
ssh-keygen -t ed25519 -C "你的GitHub注册邮箱"
一路回车使用默认路径，无需设置密码。
3. 查看公钥
bash
运行
cat ~/.ssh/id_ed25519.pub
复制终端输出的全部文本，前往 GitHub → Settings → SSH and GPG keys，新建 SSH 密钥粘贴保存。
七、Windows 与 WSL 文件互相访问
Windows 访问 WSL 文件
文件资源管理器地址栏输入：
plaintext
\\wsl$
WSL 访问 Windows 文件
Windows C 盘自动挂载在 /mnt/c/
示例：
bash
运行
cd /mnt/c/Users/Windows用户名/Documents
✅ 开发建议：项目代码放在 WSL 内部目录，不要直接在 Windows 目录编译，避免权限不足、运行卡顿问题
八、WSL 常用管理命令（PowerShell 执行）
powershell
# 关闭指定Linux发行版
wsl --terminate Ubuntu-24.04

# 完全关闭WSL虚拟机
wsl --shutdown

# 导出系统备份（迁移电脑使用）
wsl --export Ubuntu-24.04 D:\ubuntu-backup.tar

# 导入备份文件恢复系统
wsl --import Ubuntu-24.04 D:\WSL\Ubuntu D:\ubuntu-backup.tar

# 彻底卸载发行版
wsl --unregister Ubuntu-24.04
九、VSCode 连接 WSL 开发
VSCode 扩展商店安装插件：WSL（微软官方插件）
在 Ubuntu 终端内直接输入命令打开 VSCode：
bash
运行
code .
自动加载 WSL 环境，使用 Linux 的编译器、终端，实现无缝开发。
Markdown 编写辅助插件推荐（VSCode 扩展）
Markdown All in One：格式化、标题快捷键、目录生成
Markdown Preview Enhanced：美化预览界面
预览快捷键：
Ctrl+Shift+V 独立预览窗口
Ctrl+K V 侧边分屏实时预览
十、常见问题排查
wsl --install 命令报错
手动开启 Windows 功能：适用于 Linux 的 Windows 子系统、虚拟机平台，开启后重启电脑。
apt 下载超时
检查软件源文本是否完整复制，确认网络正常。
WSL 运行卡顿
PowerShell 执行 wsl --shutdown 重启 WSL。
VSCode 无法连接 WSL
更新 WSL、更新 VSCode 与 WSL 插件。
十一、可选拓展配置
安装 zsh + Oh My Zsh，美化终端
配置 Python / Conda 虚拟环境
搭建 ROS2 机器人开发环境
