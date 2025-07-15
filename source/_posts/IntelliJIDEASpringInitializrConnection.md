---
title: "IntelliJ IDEA Spring Initializr 'Connection refused: getsockopt' 解决方案"
date: 2025-07-09 12:00:00
categories:
  - 开发环境
  - Spring
  - IDEA
  - 网络问题
tags:
  - IntelliJ IDEA
  - Spring Initializr
  - 代理
  - 网络
  - 错误分析
---
# IntelliJ IDEA 初始化Spring项目失败：Connection refused: getsockopt

## 事件描述

用户尝试在IntelliJ IDEA中使用Spring Initializr创建项目时，遇到错误提示：

> 'https://start.spring.io' 的初始化失败
> Cannot download 'https://start.spring.io': Connection refused: getsockopt

IDEA提示用户检查URL、网络和代理设置。

---

## 问题分析

- **错误类型分析**：

  - "Connection refused: getsockopt" 表示客户端（IntelliJ IDEA）尝试连接服务器（start.spring.io）时，服务器主动拒绝了连接。
  - 这通常不是服务器宕机，而是中间存在阻碍，如防火墙、代理问题，或目标端口未开放。
  - [!image][https://github.com/Changhuaishui/chenji/blob/main/source/_posts/image/IntelliJ-IDEA-Spring-Initializr-Connection-Refused/1752501025386.png?raw=true]
- **常见原因**：

  1. 本地防火墙或杀毒软件阻止了IDEA的出站连接。
  2. VPN网络配置导致连接受阻。
  3. 代理设置不正确或缺失。
  4. SSL/TLS证书或代理对HTTPS连接的处理问题。

---

## 解决方案

### 1. 检查IDEA代理配置

1. 打开 `File -> Settings -> Appearance and Behavior -> System Settings -> HTTP Proxy`
2. 选择“Auto-detect proxy settings”（自动检测代理设置）
3. 或设置“Automatic proxy configuration URL”（自动代理配置URL）
4. 应用并重启IDEA
5. 如图：![https://github.com/Changhuaishui/chenji/blob/main/source/_posts/image/IntelliJ-IDEA-Spring-Initializr-Connection-Refused/1752501066694.png?raw=true)

### 2. 协议更改

- 尝试将 `https://start.spring.io` 改为 `http://start.spring.io` 访问。
- 某些网络环境下，HTTP协议可绕过SSL/代理相关问题。

### 3. 检查本地网络环境

- 关闭本地防火墙/杀毒软件后重试。
- 断开VPN后重试。

### 4. 其他可能性

- 历史上 initializr 项目曾有相关 [issue](https://github.com/spring-io/initializr/issues/267)，但目前服务稳定。
- 若上述方法无效，可尝试命令行或浏览器直接访问 start.spring.io，排查本地网络问题。

---

## 结论

本次“Connection refused”问题主要由IntelliJ IDEA的HTTP代理配置不正确导致。通过在IDEA中自动检测并配置代理设置，问题得以解决。同时，在特定网络环境下，尝试HTTP协议而非HTTPS或检查本地防火墙/VPN也是有效的排查方向。
