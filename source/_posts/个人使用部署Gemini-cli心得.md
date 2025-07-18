# 部署 Gemini CLI 的排错与解决方案

## 前言

Google Gemini 以其强大的多模态能力吸引了无数开发者，Gemini CLI 则为我们提供了在终端中与其交互的便捷方式。对于国内开发者来说，部署过程中常遇到网络和认证等问题。本文记录本人部署 Gemini CLI 的完整排错过程及最终解决方案，希望能帮你少走弯路。

## Step 1: 基础准备

请确保你已准备好以下内容：

1. **Node.js 环境**（推荐 18.x 或更高版本，请自备）
2. **网络代理工具**（如 Clash、V2RayN）
3. **Gemini API 密钥**（可在 [Google AI Studio](https://ai.google.dev/) 免费获取，可选吧）

> 注意：Gemini CLI 是基于 Node.js 开发的工具，不是 Python 工具。这是与其他一些 AI 工具的主要区别之一。

## Step 2: 安装 Gemini CLI

在终端运行：

```bash
npm install -g @google/gemini-cli
```

或者，如果你只想尝试而不想全局安装，可以使用：

```bash
npx https://github.com/google-gemini/gemini-cli
```

安装完成后，`gemini` 命令即可使用。

## Step 3: 排错全过程复盘

### 1. `Waiting for auth...` & `Login with Google` 失败

- **问题**：卡在 `Waiting for auth...`，选择 `Login with Google` 报错：需设置 `GOOGLE_CLOUD_PROJECT`。
- **原因**：Google 账号需绑定 GCP 项目，个人开发者不便。
- **解决**：放弃 Google 登录，改用 API 密钥。

### 2. `GEMINI_API_KEY not found`

- **问题**：选择 `Use Gemini API Key`，提示找不到环境变量。
- **解决**：设置环境变量（建议同时设置 `GEMINI_API_KEY` 和 `GOOGLE_API_KEY`）。

**Bash (macOS/Linux)：**

```bash
export GEMINI_API_KEY="你的API密钥"
export GOOGLE_API_KEY="你的API密钥"
```

**Windows (CMD)：**

```cmd
set GEMINI_API_KEY=你的API密钥
set GOOGLE_API_KEY=你的API密钥
```

### 3. 请求无响应 & 代理设置（端口号请自己确认是不是自己代理走的端口号）

- **问题**：认证通过但网络无响应，超时失败。
- **原因**：Gemini CLI 不会自动读取系统代理。
- **解决**：为终端会话设置 `https_proxy` 环境变量。

**Bash (macOS/Linux)：**

```bash
export https_proxy=http://127.0.0.1:7890
```

**Windows (CMD)：**

```cmd
set https_proxy=http://127.0.0.1:7890
```

设置好 API 密钥和代理后，Gemini CLI 即可正常使用。
参考文章：[国内使用 Gemini CLI 常见登录授权失败：安装与排错指南国内使用 Gemini CLI 常见登录授权失败,网络问 - 掘金](https://juejin.cn/post/7520139212605128714)

## Step 4: 解决方案——TUN 模式

每次手动设置代理环境变量较为繁琐。推荐开启代理软件（如 Clash）的 **TUN 模式**，让系统所有流量自动走代理，无需每次手动设置。

**操作方法（以 Clash 为例）：**

1. 在 Clash 设置中开启"TUN模式"或"虚拟网卡"。
2. 授权相关系统权限。
3. 启用后，所有应用（包括终端）都能自动代理，无需再设置 `https_proxy`。

## 总结：最佳实践工作流

1. **一次性配置**：将 `GEMINI_API_KEY` 和 `GOOGLE_API_KEY` 添加为系统环境变量（macOS/Linux 写入 `~/.zshrc` 或 `~/.bashrc`，Windows 通过"系统属性-环境变量"设置）。
2. **开启 TUN 模式**：日常只需打开代理软件并确保 TUN 模式开启。
3. **直接使用**：随时在终端输入 `gemini "你的问题"`，无需额外设置。

## 结语

部署 Gemini CLI 的过程虽有波折，但通过合理设置环境变量和代理，结合 TUN 模式，完全可以顺利使用。希望本指南能为你提供帮助。
后续：由于我在别的终端设置时使用Gemini CLI遇到新的问题，后续找到解决办法再解决。
