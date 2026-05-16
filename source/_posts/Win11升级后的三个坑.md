---
title: Win11升级后踩的三个坑
date: 2026-05-16 23:20:41
tags: [Windows 11, SMB, Defender, 系统激活, 故障排除]
categories: [技术教程, 系统配置]
---

# Win11 升级后踩的三个坑

> 记录于 2026.05.16，重装系统到 Win11 之后，陆续遇到的问题。
> 写下来是因为每个都花了点时间才定位到真正原因，而真正原因往往不是第一直觉猜的那个。

---

## 一、共享文件夹访问不了

同事的共享文件夹，`\\192.168.1.143`，突然访问不了了。弹窗写的是"Windows 无法访问"，很笼统。

![img](https://p9-flow-imagex-sign.byteimg.com/tos-cn-i-a9rns2rl98/4708289130ae4bbaba267e7cfd4177fa.png~tplv-a9rns2rl98-image.png?lk3s=8e244e95&rcl=2026031816050793D4B22A35500D5FD07A&rrcfp=dafada99&x-expires=2090045107&x-signature=qzCwNIKsWcqANJh25mvz3MLYzoI%3D)

第一反应是防火墙。改了半天规则，没用。

后来冷静下来想了一步——先确认网络层通不通。ping 是通的，端口 445（SMB 用的端口）也是通的。那说明不是网络的问题，是协议层的问题。

查了 SMB 客户端配置，发现 `RequireSecuritySignature` 是 `True`。

这就对了。Win11 更新后默认强制要求 SMB 签名。我的机器升级了，同事的没有，两边签名能力不对等，握手就失败了。本质上是一个版本差导致的协议协商失败。

**解法：**

```powershell
# 管理员 PowerShell
Set-SmbClientConfiguration -RequireSecuritySignature $false -Force
```

或者走组策略：`gpedit.msc` → 计算机配置 → Windows 设置 → 安全设置 → 本地策略 → 安全选项 → `Microsoft 网络客户端: 数字签名通信(始终)` → 已禁用。

改完重启下服务：

```powershell
Restart-Service LanmanWorkstation -Force
```

**一句话总结：** 能 ping 通但访问不了共享，大概率不是防火墙，是 SMB 签名策略不一致。

---

## 二、Defender 拦截了正常软件

解压 MobaXterm 的时候，右下角弹了个"发现威胁"。一开始以为是防火墙拦的，其实不是——是 Microsoft Defender 防病毒的实时保护在扫描文件时把它当威胁处理了。

查了一下检测记录，被误杀的还不少：Clash、HBuilderX 的 cli.exe、MarkdownPad、Typora 安装包……全是开发者常用工具。Defender 对这类带壳或者非签名的程序确实容易误报。

关闭实时保护可以临时解决，但重启后会自动恢复。更稳妥的做法是加排除项。

**解法：**

Windows 安全中心 → 病毒和威胁防护 → 管理设置 → 拉到最底部 → 排除项 → 添加或删除排除项 → 添加文件夹。

把常放工具和代码的目录加进去，比如 `D:\软件`、桌面路径等。

如果想用命令行加，需要先关掉「篡改防护」，然后在**管理员 PowerShell** 里执行：

```powershell
Set-MpPreference -ExclusionPath "D:\软件","C:\Users\Administrator\Desktop"
```

这里有个细节容易忽略：即使关了实时保护和篡改防护，如果终端本身不是以管理员身份启动的，命令一样会报权限不足。所以要么右键"以管理员身份运行"终端，要么直接在 GUI 里加排除项，GUI 反而更省事。

**一句话总结：** 开发机上 Defender 误报是常态，别关实时保护，加排除目录就行。

---

## 三、Win11 激活失败 0x80072EFE

重装完系统，激活的时候报了 `0x80072EFE`。查了一下，这个错误码是"服务器响应超时或连接丢失"，属于网络问题。

但网络明明是通的——浏览器能上网，ping 也正常。

想了一下，我开着 Clash 代理。激活请求走了代理通道，微软的激活服务器要么拒绝了代理 IP，要么连接被中间环节丢掉了。

**第一步：关掉 Clash 的系统代理，直连激活。**

```powershell
# 管理员 PowerShell
slmgr.vbs /ato
```

如果还不行，重置一下激活服务：

```powershell
net stop sppsvc
net start sppsvc
slmgr.vbs /ato
```

我这边试了还是不行。用 `slmgr.vbs /xpr` 查了一下当前状态，发现装的是 `ProfessionalCountrySpecific`——特供版，不是标准 Pro。这个版本没法直接联网激活，得先换密钥。

**实际解法：**

```powershell
# 1. 换成标准 Pro 通用密钥
slmgr.vbs /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX

# 2. 指向 KMS 服务器
slmgr.vbs /skms kms.03k.org

# 3. 激活
slmgr.vbs /ato
```

激活成功，显示批量激活到期时间为 2026/09/14。

**一句话总结：** 激活报网络错误，先排查代理；如果关了代理还不行，大概率是系统版本不对，查一下是不是特供版。

---

## 写在后面

三个问题，表面上看分别是"网络不通"、"防火墙拦截"、"激活超时"，但实际原因分别是协议签名策略、防病毒误报、代理干扰加上系统版本错误。

排查的思路其实就一条：**先确认你以为的原因是不是真的原因，再动手改。** 多查一步状态，少走很多弯路。
