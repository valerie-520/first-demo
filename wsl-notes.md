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