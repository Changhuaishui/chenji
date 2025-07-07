---
title: 解决个人在 IntelliJ 中部署 Maven 的 POM 文件出现的错误
date: 2025-07-07 21:22:17
tags: [Maven, IntelliJ, POM, 依赖, ESET, 网络]
---
# 🛠️ Maven POM 配置与依赖解析问题诊断及解决方案（地区本人用户）

---

## 目录

1. [问题背景与初始症状](#问题背景与初始症状)
2. [关键线索：文件锁定错误](#关键线索文件锁定错误)
3. [根本原因确认：ESET 杀毒软件的干扰](#根本原因确认eset-杀毒软件的干扰)
4. [ESET 如何导致 Maven POM 配置问题](#eset-如何导致-maven-pom-配置问题)
5. [解决方案汇总](#解决方案汇总)
6. [总结](#总结)

---

## 1. 问题背景与初始症状

在使用 IntelliJ IDEA 进行 Maven 项目开发时，遇到如下错误：

```
Non-resolvable import POM: The following artifacts could not be resolved: org.springframework.ai:spring-ai-bom:pom:1.0.0-M6 (absent): org.springframework.ai:spring-ai-bom:pom:1.0.0-M6 failed to transfer from https://repo.maven.apache.org/maven2 during a previous attempt. This failure was cached in the local repository and resolution is not reattempted until the update interval of central has elapsed or updates are forced. Original error: Could not transfer artifact org.springframework.ai:spring-ai-bom:pom:1.0.0-M6 from/to central (https://repo.maven.apache.org/maven2): Connect to repo.maven.apache.org:443 [repo.maven.apache.org/146.75.112.215] failed: Connect timed out
```

**分析：**
Maven 无法从远程仓库下载依赖，核心原因是 `Connect timed out`（连接超时）。这通常与中国地区访问境外 Maven 仓库的网络限制有关。

---

## 2. 关键线索：文件锁定错误

后续又遇到如下错误：

```
Could not acquire lock(s)
```

**分析：**
该错误表明文件或资源被其他进程占用，Maven 或 IntelliJ IDEA 无法获得独占锁，导致操作失败。

---

## 3. 根本原因确认：ESET 杀毒软件的干扰

经过排查，发现是电脑上的 **ESET 杀毒软件** 导致了文件锁定问题。

---

## 4. ESET 如何导致 Maven POM 配置问题

- **文件锁定（File Locking）：**ESET 的实时文件系统保护会扫描 Maven 本地仓库（如 `~/.m2/repository`），在扫描过程中锁定文件，阻止 Maven 正常读写，导致依赖无法下载或校验。
- **网络拦截（Network Interception）：**
  ESET 可能会拦截 Maven 与远程仓库的连接，加剧 `Connect timed out` 问题。

---

## 5. 解决方案汇总

### 5.1 针对 ESET 杀毒软件的解决方案（核心）

- **添加排除项（Exclusions）：**

  - 排除 Maven 本地仓库目录（如 `C:\Users\你的用户名\.m2\repository`）
  - 排除 IntelliJ IDEA 安装及工作目录
  - 排除 Java (JVM) 进程（如 `java.exe`、`javaw.exe`）
- **操作步骤示例：**

  1. 打开 ESET 主界面
  2. 设置 → 高级设置（F5）
  3. 检测引擎 → 实时文件系统保护 → 排除项
  4. 添加需要排除的文件夹或文件
- **临时禁用 ESET 实时保护**（仅在必要时，操作完毕后务必重新启用）

### 5.2 配置 Maven 镜像仓库（解决网络连接超时）

- 编辑 `~/.m2/settings.xml`，添加阿里云镜像：

  ```xml
  <mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>Aliyun Maven</name>
    <url>https://maven.aliyun.com/repository/public</url>
  </mirror>
  ```
- 在 IntelliJ IDEA 中指定 `settings.xml` 路径。

### 5.3 强制刷新 Maven 本地仓库与项目

- 删除本地仓库中相关 `.lastUpdated` 文件或整个依赖文件夹
- 运行命令强制更新：

  ```bash
  mvn clean install -U
  ```
- 在 IntelliJ IDEA 中点击 Maven Projects → Reimport All Maven Projects，并勾选 Force Update

### 5.4 IntelliJ IDEA 相关操作

- 清除缓存并重启：
  `File → Invalidate Caches / Restart... → Invalidate and Restart`

### 5.5 其他网络排查建议

- 检查本地防火墙设置
- 确保 DNS 能正确解析 Maven 仓库地址

---

## 6. 总结

你遇到的 Maven POM 配置问题，是中国地区网络限制、Maven 缓存机制和 ESET 杀毒软件三者共同作用的结果。
通过**添加 ESET 排除项**、**配置国内镜像**、**强制刷新依赖**和**清理 IDEA 缓存**，可以彻底解决此类问题。

---

> **遇到问题不可怕，关键是科学排查与系统解决。祝你开发顺利！**

---

_如有更多问题，欢迎留言或通过 [GitHub](https://github.com/Changhuaishui) 联系我。_

```

```
