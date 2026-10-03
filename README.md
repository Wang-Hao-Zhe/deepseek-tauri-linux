# deepSeek-tauri-linux

> 将 `https://chat.deepseek.com` 包装成 Linux 桌面应用的 Tauri 学习项目。

---

##  重要说明

**本项目仅作为 Tauri 学习示例，不建议日常使用。**

原因如下：

- **风控风险**：DeepSeek 风控系统可识别 WebKitGTK 指纹，启动后会提示“环境异常”，无法正常使用。
- **账号风险**：使用非官方客户端可能导致 DeepSeek 账号被临时封禁。社区已有用户因类似行为被封禁 1～3 天。
- **功能受限**：剪贴板图片粘贴、文件拖拽上传在 WebKitGTK 下均无法正常工作。

如果你只是想要一个 DeepSeek 桌面入口，**强烈推荐使用 Firefox 155+ 自带的 Taskbar Tabs 功能**，几秒钟即可搞定，且零风控风险。详见下文 [推荐方案](#-推荐方案firefox-原生任务栏应用)。

---

##  项目背景

这个项目起源于一个简单的想法：**用 Gecko 引擎把 DeepSeek 网页包装成桌面应用**。

最初的探索路径：

1. **Gecko 引擎直接嵌入**：没有成熟的桌面端方案。
2. **参考社区项目**：注意到 [jwangkun/DeepSeek-Desktop](https://github.com/jwangkun/DeepSeek-Desktop) 基于 Pake 将网页打包为桌面应用，但其目标平台主要是 macOS 和 Windows，未覆盖 Linux。
3. **Tauri + WebKitGTK**：在 Linux 上使用系统 WebView，理论上轻量且可行。
4. **实际构建**：编译了数百个 Rust crate，配置了图标、分类、`.desktop` 文件。
5. **撞上风控**：DeepSeek 检测到 WebKitGTK 指纹，判定“环境异常”，拒绝服务。
6. **最终发现**：Firefox 自带的 Taskbar Tabs 功能完全满足需求，且没有风控问题。

本项目记录了第 3～5 步的完整配置和构建流程，希望能为后来者提供参考，避免重复踩坑。

---

##  技术栈

- **Tauri 2**（Rust 后端 + WebKitGTK 前端）
- **Rust** 1.90+
- **Fedora Linux**（开发环境,版本44）

---

##  构建方法 (Fedora Linux)

### 1. 安装系统依赖

```bash
sudo dnf install gtk3-devel webkit2gtk4.1-devel glib2-devel \
  libappindicator-gtk3-devel librsvg2-devel libxdo-devel \
  openssl-devel patchelf gcc-c++ file
```

### 2. 安装 Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
rustup update stable
```

确保 Rust 版本 ≥ 1.85.0。

### 3. 安装 Tauri CLI

```bash
cargo install tauri-cli --version "^2"
```

### 4. 构建

```bash
cargo tauri build
```

产物位于：

```text
src-tauri/target/release/bundle/
```

包含 `.rpm`、`.deb` 和 AppImage。

### 5. 安装（RPM 示例）

```bash
sudo dnf install ./src-tauri/target/release/bundle/rpm/*.rpm
```

---

##  已知限制

| 限制 | 说明 |
|---|---|
| **风控拦截** | DeepSeek 检测到 WebKitGTK 指纹后提示“环境异常”，无法使用 |
| **剪贴板图片** | WebKitGTK 不通过标准 Web API 暴露剪贴板图片，粘贴图片无效 |
| **拖拽上传** | 即使设置 `dragDropEnabled: false`，网页端仍可能因模式限制无法上传 |
| **User-Agent** | WebKitGTK 的 UA 与主流浏览器差异明显，易被识别 |
| **Canvas/WebGL 指纹** | 与 Chrome/Firefox 不同，无法通过配置彻底伪装 |

**结论**：Tauri 在 Linux 上包装主流网站服务时，WebView 指纹是难以逾越的障碍。本项目仅作为 Tauri 打包流程的学习案例。

---

##  推荐方案：Firefox 原生“任务栏应用”

如果你的 Firefox 版本在 **155+**，Mozilla 已内置类似 PWA 的功能，官方叫 **Taskbar Tabs**。它使用真正的 Firefox 内核渲染，**UA、Canvas 指纹、WebGL 指纹全部是标准 Firefox**，DeepSeek 风控不会拦截。

### 启用步骤（Fedora）

1. 在 Firefox 地址栏输入 `about:config`，回车，接受风险提示。
2. 搜索 `browser.taskbarTabs.enabled`，将其值设为 `true`。
3. 重启 Firefox。
4. 打开 `https://chat.deepseek.com`，地址栏右侧会出现一个 **“Add tab to taskbar”** 按钮，点击它。
5. Firefox 会弹出一个精简窗口，并提示你将它固定到任务栏。确认即可。

完成后，DeepSeek 就会作为一个独立窗口出现在应用菜单和任务栏里，**没有地址栏、没有标签页**，看起来和原生应用一样。

> **注意**：Flatpak 版 Firefox 也支持此功能，但默认禁用，需要手动开启。如果 `about:config` 里搜不到 `browser.taskbarTabs.enabled`，说明 Firefox 版本低于 155，请先升级。

### 与 Tauri 方案对比

| 对比项 | Tauri + WebKitGTK | Firefox Taskbar Tabs |
|---|---|---|
| **渲染引擎** | WebKitGTK（指纹异常） | Gecko（标准 Firefox 指纹） |
| **风控风险** | 高，会被判定“环境异常” | **零风险** |
| **剪贴板图片** | 不支持 | **支持** |
| **拖拽上传** | 需要额外配置 | **原生支持** |
| **依赖** | Rust、GTK、WebKitGTK 开发库 | 只需 Firefox 155+ |
| **维护成本** | 需自行编译、打包、对抗风控 | **零维护** |

---

##  方案选择建议

| 你的情况 | 推荐方案 |
|---|---|
| Firefox 155+，想要最省事、最稳定 | **Firefox Taskbar Tabs** |
| 坚持用 Tauri 做学习项目 | 保留本项目，但请勿日常使用 |

---

##  许可证

[MIT](LICENSE)

---

##  免责声明

- 本项目为**非官方项目**，与深度求索公司无关。
- 本项目仅用于学习 Tauri 打包流程，**不鼓励、不支持**用于绕过任何服务条款。
- 使用本项目产生的任何后果（包括账号封禁）由使用者自行承担。
- DeepSeek 是其各自所有者的商标。本项目不使用其 Logo 或暗示官方关联。
