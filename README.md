# TDL 桌面客户端 · 发布仓库

> 本仓库**仅用于分发**安装包与更新日志，由 `scripts/publish-release.mjs` 自动生成，请勿手工修改。
> 源码、构建方式与问题反馈请前往 **[t3nc/TgDL_Helper](https://github.com/t3nc/TgDL_Helper)**。

**最新版本：v0.1.0**（2026-09-16）

## 下载

| 版本 | 发布日期 | 安装包 | 大小 | SHA256 |
| --- | --- | --- | --- | --- |
| 0.1.0 | 2026-09-16 | [TgDL-0.1.0-x64-setup.exe](https://github.com/t3nc/TgDL_repo/releases/download/v0.1.0/TgDL-0.1.0-x64-setup.exe) | 11.43 MB | `16a6cfea55463362…` |

- 完整校验和见 [`SHA256SUMS.txt`](./SHA256SUMS.txt)
- 安装包**未做代码签名**，Windows SmartScreen 可能提示「未知发布者」，请核对校验和后运行

## 更新日志

见 [`UPDATELOG.md`](./UPDATELOG.md)。

---

<details>
<summary>展开查看完整项目说明（同步自源码仓库 README.md）</summary>

# TDL 桌面客户端

面向 Windows 平台的 [tdl](https://docs.iyear.me/tdl/zh/)（Telegram Downloader）图形客户端。
把命令行工具 tdl 的下载能力封装为可视化桌面应用：**内嵌 tdl 内核、支持从 GitHub 自动更新内核**、
提供二维码 / 手机验证码登录向导，以及大批量链接的批量下载任务队列与实时进度。

界面为简体中文，面向不熟悉命令行的用户；架构上已为后续扩展到 macOS 预留平台分支。

> 当前版本 **0.1.0**（内核 tdl 0.20.4），完整变更历史见 [UPDATELOG.md](./UPDATELOG.md)。

---

## 功能一览

| 模块 | 能力 |
| --- | --- |
| **批量下载** | 每行粘贴一个 Telegram 消息链接（空行忽略，自动去重与合法性预校验），或导入官方客户端导出的 JSON 文件；超长列表按命令行长度预算自动拆分为多个子任务串行执行 |
| **下载参数** | 下载目录、文件名模板（含变量/函数速查与实时预览）、扩展名白名单/黑名单（互斥）、按 MIME 重写扩展名、跳过同名同大小、相册整组下载、反序下载、Takeout 会话、断点续传策略 |
| **并发与节流** | 稳健 / 均衡 / 极速三档预设，以及并发任务数、单任务线程数、连接池、任务间隔的手工微调 |
| **登录向导** | 二维码扫码登录、手机号 + 验证码登录（含两步验证密码），多账号命名空间隔离，登录前明确提示数据覆盖风险 |
| **任务队列** | 独立页卡；按会话（DialogID）两级展开进度 —— 会话行显示汇总进度、展开后逐个文件显示各自进度与体积；状态筛选、取消（保留断点）、重试、打开目录、删除记录 |
| **重复下载检测** | 每次成功下载都记入本地历史；提交时自动识别此前下载过的链接，弹窗确认后可选「仍然全部下载」（以 `重下_` 前缀重命名，不覆盖原文件）或「跳过这些链接」；原文件已不存在时自动放行 |
| **运行日志** | 内置只读终端（xterm.js）原样呈现 tdl 的 ANSI 输出（二维码、彩色进度条），同时提供结构化日志列表 |
| **内核管理** | 显示当前内核版本与来源（内置 / 已更新 / 自定义路径），检查更新、分步更新进度、多版本回滚、指定版本安装 |
| **设置中心** | 代理、存储路径、NTP、重连超时、主题外观与配置维护，全部本地持久化 |

---

## 技术栈

- **桌面框架**：[Tauri 2](https://tauri.app/)（Rust 后端 + 系统 WebView2）
- **前端**：React 18 + TypeScript + Vite 5 + Tailwind CSS 3 + shadcn/ui 风格组件 + Zustand + xterm.js
- **后端**：Rust（`portable-pty` 提供 ConPTY、`reqwest` 负责 GitHub 更新、`zip` + `sha2` 负责校验解压）

---

## 环境要求

| 依赖 | 说明 |
| --- | --- |
| Windows 10 / 11 | 需自带 WebView2 Runtime（Win11 及新版 Win10 默认已安装） |
| Node.js ≥ 18 | 构建前端（开发使用 Node 26 验证） |
| Rust stable（MSVC） | `rustup default stable-x86_64-pc-windows-msvc` |
| **Visual Studio Build Tools** | 必须勾选「使用 C++ 的桌面开发」以提供 `link.exe`；**缺少时 `cargo` 会报 `linker link.exe not found`** |

安装缺失的构建工具：

```powershell
winget install --id Microsoft.VisualStudio.2022.BuildTools --override "--quiet --wait --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended"
```

---

## 快速开始

```bash
npm install              # 安装前端依赖
npm run kernel:fetch     # 预置 tdl 内核到 src-tauri/resources/（网络不可用时自动跳过）
npm run tauri:dev        # 开发模式：同时启动 Vite 与 Tauri 窗口
```

构建安装包：

```bash
npm run tauri:build      # 产物位于 src-tauri/target/release/bundle/nsis/
```

### 构建产物

| 产物 | 路径 | 体积 |
| --- | --- | --- |
| 免安装可执行文件 | `src-tauri/target/release/tgdl-desktop.exe` | 约 6 MB |
| 安装包（NSIS，推荐分发） | `src-tauri/target/release/bundle/nsis/TDL 桌面客户端_0.1.0_x64-setup.exe` | 约 11 MB |

> 体积小是因为内核（`tdl.exe`）以压缩形式随包分发，前端资源也经过 gzip 优化。

**注意：可执行文件必须与同级 `resources/` 目录一起分发。**

Tauri 会把 `src-tauri/resources/` 复制到可执行文件同级的 `resources/` 目录，
应用按以下顺序查找内置内核：

1. `resource_dir()/tdl.exe`、`resource_dir()/resources/tdl.exe`
2. 可执行文件同级 `tdl.exe`、`resources/tdl.exe`

若只单独拷贝 `tgdl-desktop.exe`，内置内核会丢失，此时应用仍可启动，
「内核管理」会显示内核来源为 `missing`，点击「检查更新」即可从 GitHub 重新获取。

仅预览前端界面（浏览器中运行，使用模拟后端）：

```bash
npm run dev              # 打开 http://localhost:5173
```

> 开发端口刻意避开了 Tauri 默认的 1420：在部分 Windows 环境中
> `1382-1481` 属于系统保留端口区间，监听会被拒绝（`EACCES`）。
> 可用 `netsh interface ipv4 show excludedportrange protocol=tcp` 查看本机保留区间。

> 浏览器预览模式下，所有后端调用会回落到 `src/lib/mock.ts` 的模拟实现，
> 便于在没有 Rust 工具链的机器上验收界面；桌面运行时不加载任何模拟逻辑。
> 状态栏会显示「浏览器预览模式」徽标作为提示。

---

## 内核（tdl.exe）管理

### 解析优先级

由 `src-tauri/src/kernel/resolve.rs` 统一处理，从高到低：

1. 设置中手动指定的 `customPath`
2. 应用数据目录中的托管内核（本应用安装 / 回滚的版本）
3. 安装包内置的 sidecar 内核（`src-tauri/resources/tdl.exe`）

磁盘布局（位于 `%APPDATA%\com.tgdl.desktop\`）：

```text
config.json                      应用配置
kernel/
  current.json                   当前激活的内核指针
  versions/
    0.20.4/
      tdl.exe
      meta.json
```

### 更新流程

1. 请求 `https://api.github.com/repos/iyear/tdl/releases/latest` 获取版本号（或分页列表）
2. 按架构选择资产（Windows 默认 `tdl_Windows_64bit.zip`）
3. 流式下载并实时上报进度（可走设置中的代理，或配置 GitHub 加速前缀）
4. 拉取 `tdl_checksums.txt` 并比对 SHA256
5. 解压出 `tdl.exe` 写入独立版本目录
6. 全部成功后才原子切换 `current.json` 指针，旧版本保留用于回滚

任一步失败都只清理临时产物，**不会触碰正在使用的内核**。

### 构建期预置

`scripts/fetch-kernel.mjs` 会把内核放进安装包。支持环境变量：

| 变量 | 作用 |
| --- | --- |
| `TDL_VERSION` | 指定要预置的版本（默认取最新稳定版） |
| `TDL_GH_PROXY` | 加速前缀，例如 `https://ghproxy.net/` |

脚本是「尽力而为」的：下载失败只写 `tdl.version = none` 后正常退出，
保证构建不中断，应用首次运行后仍可在「内核管理」中获取内核。

---

## 项目结构

```text
src/                          前端
  lib/ipc.ts                  唯一 IPC 出口（Tauri 命令 / 事件 / 浏览器模拟回落）
  lib/types.ts                与 Rust 侧一一对应的类型契约
  lib/validate.ts             链接解析去重、参数互斥校验、模板速查表
  lib/qr.ts / ansi.ts         二维码位图还原、ANSI 输出解析
  lib/jobItems.ts             任务条目（按会话/文件两级）聚合
  lib/jobSync.ts              任务列表合并（终态不回退、本地新任务不丢失）
  hooks/                      终端、二维码矩阵、Tauri 事件、任务同步轮询
  store/                      配置、任务、登录、内核、下载历史五类状态
  components/                 布局 / 通用 / 下载 / 登录 / 内核 / 队列 / 设置 / 终端 / ui
  pages/                      下载工作台、任务队列、账号、内核、设置五个页面

src-tauri/                    后端
  src/commands/               Tauri 命令层
  src/config/                 配置模型与持久化（原子写 + 损坏回退）
  src/kernel/                 内核解析、GitHub 客户端、安装器、版本清单
  src/tdl/                    ConPTY 运行器、参数构建、输出解析、URL 分块
  src/auth/                   登录状态机
  src/download/               下载队列与进度聚合
  src/history/                下载历史持久化与查重
  src/state.rs                全局共享状态
  src/events.rs               事件契约（字段命名与前端严格一致）
  resources/                  内置内核目录（由构建脚本填充）

scripts/fetch-kernel.mjs      构建期预置内核（失败不中断构建）
```

---

## 关键实现约束

这几条是踩坑后固化的设计决策，改动时请勿绕过：

1. **必须用伪终端（ConPTY）启动 tdl。**
   `tdl login -T code` 依赖 `survey/v2`（要求真实 TTY），下载进度条依赖
   `go-pretty/progress`（按终端宽度做 ANSI 原地重绘），二维码是半块字符绘制。
   使用普通管道会导致登录直接失败、二维码与进度条无法渲染。

2. **必须显式传 `--continue` 或 `--restart`。**
   二者互斥；若都不传且存在未完成进度，tdl 会弹出交互式确认，
   在图形界面下会导致任务卡住。界面通过「断点续传策略」固定传入其一。

3. **`--include` 与 `--exclude` 互斥**，前端禁止同时勾选，后端在构建参数时二次拦截。

4. **`-u` 与 `-f` 不能同时为空**，否则 tdl 只会抛出 `no urls or files provided`；
   后端在启动进程前拦截并给出中文提示。

5. **命令行长度限制**：Windows 单条命令行上限约 32767 字符，
   分块预算取 20000 字符（`src-tauri/src/tdl/chunker.rs`）。

6. **进度聚合 = 文件清单通道 + 目录采样通道，而不是解析进度条动画。**
   go-pretty 的进度条是 ANSI 原地重绘，逐字符解析进度条本身既脆弱又易误判。
   这里只用两类**稳定信息**：

   - **文件清单通道**：从 `来源(ID):消息ID -> 文件名` 提取文件集合，并从
     **同一行尾部**的 `已下载/总量` 取到每个文件的总大小
     （注意：消息与进度条同行，必须先 `patterns::split_metrics()` 切分，见「已修复缺陷」第 10 项）；
   - **目录采样通道**：每秒扫描一次目标目录（含下载中的临时文件）计算真实增量与滑动窗口速度。

   总长度取「tdl 报告的总量」与「实际已落盘量」的较大者，保证进度不会超过 100% 或倒退；
   文件计数优先采用 tdl 的报告值，避免把下载中的临时文件误算成已完成。

7. **隐私**：代理密码、验证码、两步验证密码不会写入任何日志；
   参数快照中的代理地址会被 `****` 脱敏（`src-tauri/src/error.rs::desensitize`）。

---

## 已验证与未验证项

| 项目 | 状态 |
| --- | --- |
| 前端类型检查（`tsc -b`） | ✅ 通过 |
| 前端生产构建（`vite build`） | ✅ 通过 |
| 前端单元测试（`npm test`） | ✅ 27 个全部通过（3 个测试文件） |
| Rust 编译（`cargo test --all-targets`） | ✅ 通过，0 warning |
| Rust 单元测试 | ✅ 102 个全部通过 |
| Rust 集成测试 | ✅ `pty_exit`（PTY 退出语义）通过；`pty_prompt` 需真实网络，默认忽略 |
| 内嵌内核可执行 | ✅ `tdl.exe version` 输出 `0.20.4` |
| `tauri dev` / 发布版启动 | ✅ 均通过（发布版已产出安装包） |
| 真实账号登录与下载 | ⏳ 需使用你自己的 Telegram 账号验证 |

**Rust 单元测试覆盖**（102 个）：**消息级下载条目与按会话分组**、**下载历史持久化与三层查重**、tdl 参数构建与互斥约束、**参数快照的链接/文件计数**、
URL 分块字符预算、ANSI 剥离与行组装、输出节流路由、
**不带换行的交互提示识别**、**标题与进度条同行时的切分与解析**、
**重绘不放大文件数 / 全局汇总行不污染单文件统计**、
**tdl 真实进度行（无 `/总量`、`#` 进度条）的体积解析与总量反推**、
**反推总量的单调性与完成时校准**、**收尾补齐（终帧缺失）与收尾总量口径**、
进度行与体积解析、版本号比较、内核目录布局、zip 提取失败分支、
GitHub 限流降级与版本号提取、**事件序列化字段命名（防契约漂移）**。

**集成测试**：
- `tests/pty_exit.rs` —— 固定「ConPTY 在子进程退出后不给 EOF，必须显式关闭主端」
  这一平台契约。它曾直接复现了「下载完成后状态永远停在下载中」的根因。
- `tests/pty_prompt.rs` —— 需真实内核与网络，默认 `#[ignore]`。

**前端测试覆盖**（27 个）：以真实捕获的 tdl 二维码输出为夹具，验证
CRLF/ANSI 归一化、二维码图块识别、字符到模块的位映射（含明暗极性）、
静默区与三个定位图形与定时图形的结构校验、自动刷新取最新帧、
**任务列表合并（终态不回退、本地新任务不丢失、后端新增被追加）**、
截断图块拒绝、以及 canvas 绘制的等比无缝与静默区不被侵入。

### 已修复的关键缺陷（排查记录）

1. **事件字段命名不匹配导致全部事件静默失效**
   `#[serde(tag = "kind", rename_all = "camelCase")]` 中的 `rename_all`
   只作用于变体名，不作用于变体内部字段。后端实际发出 `job_id`，
   而前端读取 `jobId` → 恒为 `undefined`，导致终端输出、日志、进度全部丢失。
   修复：为每个变体单独标注 `rename_all = "camelCase"`，并补充序列化回归测试。

2. **终端订阅被误删导致二维码永不显示**
   发起登录时调用了 `terminalBus.dispose()`，而该方法会连同订阅者一起删除。
   此时终端面板已挂载并完成订阅 → 后续输出永远收不到。
   修复：改用 `terminalBus.clear()`（保留订阅，仅清屏），并明确两者的语义边界。

3. **开发端口落在 Windows 保留区间**
   Tauri 默认的 1420 在部分机器上属于系统保留端口段（如 `1382-1481`），
   监听时报 `EACCES`。已改用 5173。

4. **二维码被截断**
   二维码由 23~24 行半块字符组成，且依赖「光标上移」原地刷新，
   终端可视高度不足会导致顶部滚出视口而无法扫描。

5. **验证码 / 两步验证步骤在界面上无法推进**（交互提示不带换行符）
   tdl 的验证码登录依赖 `survey` 交互库，而 `survey` 在等待输入时只写出
   `? Enter Code: ` 这样一条**没有换行符**的提示，直到用户回车才补上换行。
   输出路由原先只按 `\n` 切行，导致该提示永远不会进入语义识别，
   后端始终停在 `sendingCode` 阶段，界面也就一直停在「手机号」步骤，
   用户根本没有输入验证码的入口。
   （手机号那一步之所以正常，是因为它的阶段在启动时就已预设，不依赖提示识别。）

   修复：在输出路由中增加「未结束行」通道 —— 未结束的内容稳定 120ms 后
   即作为提示上报，并以 1.5s 间隔兜底重复上报（防事件丢失）。
   同时把提示识别**只**放在该通道：带换行的同一条提示是用户提交后内核的回显，
   若一并识别会把阶段错误地回退一步。

6. **时长格式校验过严，导致「开始下载」在默认配置下永久禁用**（阻塞性）
   校验正则 `^\d+(\.\d+)?(ns|us|µs|ms|s|m|h)$` 只允许**单段**时长，
   而 tdl 与应用的默认值都是 `5m0s`（Go 的 `time.ParseDuration` 允许多段组合）。
   于是 `validateDownload()` 对默认配置直接报错 → `canSubmit` 恒为 `false` →
   「开始下载」永远是禁用状态，用户点不动也不知道为什么。
   校验失败文案里写的示例恰好就是 `5m0s`，自相矛盾。

   修复：改为支持多段组合与裸 `0` 的正则，并对 `--delay` 同步生效。

7. **底部执行条（开始下载 / 保存设置）跑到了内容末尾**
   内容包裹层带有 `animate-fade-up`，而该动画的 `animation-fill-mode: both`
   会**永久保留** `transform: translateY(0)`。任何带 transform 的祖先
   都会成为 `position: fixed` 的包含块，执行条因此相对包裹层而非视口定位，
   落到了内容流末尾——必须滚动到底才看得见。
   且内容区的滚动容器是 AppShell 的 `<main>`（window 本身不滚动），
   `fixed` 在这个布局里本就不成立。

   修复：下载工作台与设置中心的执行条改用 `position: sticky`（`bottom-0`），
   既钉在滚动容器底部，又只占内容区宽度、不会压到侧边栏。

8. **二维码模块之间存在缝隙，手机无法识别**（最彻底的一次修复）
   终端字符单元格并非正方形（等宽字体字宽约 0.6em、行高 1em），
   再叠加字体行距，半块字符拼出的二维码会出现明显纵向缝隙，
   模块被割裂后手机无法识别；同时终端配色还会影响明暗极性。

   最终方案是**不再用终端展示二维码**：从内核输出中把半块字符当作位图编码解析出来，
   还原成模块矩阵，再用 canvas 按严格正方形模块重绘（纯黑白、关闭抗锯齿、
   在 tdl 自带的 4 模块静默区之外再补 2 个模块）。
   解析结果还会经过「静默区 + 三个定位图形 + 定时图形」的结构校验，
   既挡住了重绘中途采到的半截图块，也能反证明暗极性是否正确。

   终端视图作为排障用途保留在可折叠面板中，并将行高调整为 1 以减少缝隙。

9. **下载完成后界面永远停在「下载中」**（收尾顺序错误，阻塞性）
   Windows ConPTY 在子进程退出后**不会**让 PTY 读取端收到 EOF，
   读取线程会一直阻塞在 `read()` 上（`tests/pty_exit.rs` 固定了这一契约）。

   而下载任务的收尾顺序是「先 `join_reader()` 等读取线程 → 再落地最终进度与
   `done` 状态」，于是这一 join 永久卡住，最终进度与完成状态**永远发不出去**：
   进度停在最后一次采样，任务状态一直显示「下载中」，尽管终端里 tdl 早已完成。

   同一个坑还影响登录：`auth/service.rs` 的守卫线程同样是 `wait()` 之后
   `join_reader()`，因此**登录失败时界面永远收不到失败通知**（成功路径靠日志
   正则侥幸可用）。另外仓库里此前的 `pty_prompt` 端到端测试也是卡在这里不退出。

   修复：
   - `PtySession` 支持显式关闭主端（`close_master_async`），关闭后 ConPTY 才会
     放开输出管道、读取端才能拿到 EOF；
   - `SpawnedTdl` 增加有界等待 `join_reader_timeout`，析构时**只放弃、不 join**；
   - 下载收尾改为：确保进程结束 → 关主端 → 冲刷最后一批输出 → **立即落地
     最终进度与状态** → 最后才做有界清理。任务成败不再依赖读取线程。

10. **进度条恒为 100%、数值显示成 `X/X`**（标题与进度条同行导致解析错位）
    tdl 的 `pkg/prog` 用 go-pretty 渲染，`SetTrackerPosition(PositionRight)` 且
    消息占终端宽度 3/5、进度条占 1/5 —— **消息与进度条在同一行**：

    ```text
    我的频道(-1001234567890):42 -> video.mp4    ██████░░  62.5%  18.4 MiB/29.7 MiB  2.1 MiB/s  5s
    ```

    而 `TRACKER` 正则用 `(?P<file>\S.+?)\s*$` 锚定行尾，于是把百分比、体积、速度
    全部当成文件名；紧接着 `observe()` 又提前 `return`，把同一行的总量信息丢弃。
    后果是每次重绘都生成一个「新文件」（文件数被无限放大），并且
    `known_bytes()` 恒为 0 → `bytes_total` 退化成「已下载量」→ 进度条恒为 100%。

    修复：新增 `patterns::split_metrics()`，先按「首个进度条字符」与「首个连续空格」
    中更靠前者把行切成「消息」与「进度指标」两段，再分别解析；同一行的指标归入该文件。
    全局汇总行（消息区为空）刻意不参与单文件统计，避免污染总量与完成判定。

    同时文件计数改为优先采用 tdl 的报告值 —— 目录里躺着下载中的临时文件，
    按目录文件数统计会把「正在下载」误算成「已完成」。

11. **「生效参数快照」里的链接数统计错误**
    快照渲染无法区分「开关型参数」与「带值参数」，于是 `--continue` 把紧随其后的
    `-u` 当成自己的取值吞掉了——而 `-u` 恰好**总是**排在 `--continue` 之后，
    因此快照里链接数恒为 0（实际下载完全正常）。
    修复：维护开关型参数白名单，只对带值参数跳过其取值；同时补上 `-f` 的 JSON 文件计数。

12. **任务队列进度恒为 `0 B`，而运行日志里明明在下载**（tdl 真实进度行格式不匹配）
    tdl 的进度行由 go-pretty 渲染，实测形态是：

    ```text
    国产麻豆糖心反差 (2515320395):15569 -> E:\Download…  51.6%  [##########.........]  83.00 MB in 1m19.836s; ~ETA: 31s; 1.36 MB/s
    ```

    与既有解析假设有三处不匹配：

    1. **体积形态**：`parse_bar()` 只用 `VALUE_TOTAL`（要求 `已下载/总量`）取体积，
       而真实输出**只有已下载量**、没有 `/总量`。于是 `value`/`total` 恒为 `None`，
       条目 `bytes`/`bytes_total` 恒为 0、`known_bytes()` 恒为 0 →
       聚合 `bytes_total` 退化成已下载量 → 进度条恒空、文件计数恒为 `0/N`。
    2. **进度条字符**：真实填充字符是 `#`（形如 `[####.....]`），而 `BAR_CHARS`
       只含半块字符；当消息被 go-pretty 截断到刚好填满分配宽度、与指标之间
       只剩**单个空格**时，「首个 bar 字符」与「首个连续空格」双双落空，
       整行会被当成文件名、指标全部丢失。
    3. **采样目录**：`run_job()` 用「当前配置」的目录做目录采样，而 tdl 的 `-d`
       来自提交时的目录（`job.dir`）。前端是「先 submit、后异步保存配置」，
       用户也可能中途改过默认目录——两者不一致时目录采样恒为 0，
       `bytes`/`speedBps` 永远是 0（这正是队列概览里「总速度 0 B/s」的由来）。

    修复：
    - `parse_bar()` 改为双形态兼容：先按 `已下载/总量` 取（旧形态行为不变），
      未命中时**先剥离速度片段**再取首个可读体积作为已下载量，
      并用百分比反推总量（`percent < 5%` 时不反推，避免放大舍入误差）；
    - `split_metrics()` 增加「首个百分比」作为第三候选分界点，
      `BAR_CHARS` 追加 `#`，覆盖单空格 + `#` 进度条的渲染；
    - `TrackerState::upsert()` 的总量取**历史最大值**（反推值会随百分比舍入
      在 ±1% 内波动，取最大值可保证进度条不倒退），并在完成时用内核给出的
      真实体积校准，避免「标记完成却停在 9x%」；
    - `run_job()` 的工作/采样目录改用 `plan.job.dir`，与 `-d` 完全一致。

    另外补了一层前端兜底：存在活动任务时每秒静默同步一次任务列表
    （`useTaskSync` + `jobSync.mergeJobList`），即使某个进度事件丢失，
    队列也不会停在旧值。合并规则刻意「只增不删」，避免取消状态回跳与
    刚提交的任务因同步早于后端入队而闪没。

13. **任务已完成，子任务进度却停在 92%**（go-pretty 不渲染最后一个文件的终帧）
    tdl 的进度条由 go-pretty 渲染，生命周期由 `pkg/prog/prog.go` 的 `Wait()` 控制：

    ```go
    for pw.IsRenderInProgress() {
        if pw.LengthActive() == 0 { pw.Stop() }   // 已无活跃 tracker → 立刻停止渲染
        time.Sleep(10 * time.Millisecond)
    }
    ```

    它以 **10ms** 轮询，而 `SetUpdateFrequency` 是 **100ms**。文件下载完成的瞬间
    该 tracker 变为非活跃，`Stop()` 会在 10ms 内被调用（又因 `Visibility.Pinned = true`
    还会顺带清掉钉住区域），因此「`done!` + `100.0%`」这一帧**几乎必然来不及渲染**：
    最后渲染出的有效帧就是 9x%（真实反馈：`7.00 MB / 7.58 MB = 92.3%`，
    正是 `92.4%` 那一帧）。

    后果链条：

    1. `upsert()` 只在 `percent >= 99.5` 或 `bytes >= bytes_total` 时置 `done`，
       该条件永不触发 → 条目 `bytes` 停在最后一帧的值、`done` 恒为 false；
    2. `files_done()` 恒为 0 → 界面显示 `文件 0/1`，会话行停在 92%；
    3. 而收尾时是**无条件**把任务状态置为 `done` 的（只要未取消、未捕获 error），
       于是「任务已完成」与「条目 92%」并存；
    4. 连带影响：`completed_inputs()` 按 `done` 过滤 → **下载历史漏记**，
       下次重贴同一链接不会被查重命中；
    5. 另有 0.4% 的残余误差：收尾总量取了「百分比反推值」（7.58 MB）而非
       「实际落盘量」（7.55 MB），聚合因此停在 99.7%。

    修复：把「完成」的判据从**文本通道**换成**事实通道**：

    - `TrackerState::finalize_all()`：收尾时把仍未标记完成的条目补齐
      （`bytes` 抬到 `bytes_total`，总量未知时反向回填），并自增 revision
      以便上层补发条目快照；
    - `run_job()` 收尾在处理完最后一批输出之后调用它，条件为
      「未取消且未捕获到错误」—— 取消与失败的任务保持真实停止点，
      绝不显示成全部完成；
    - `DirSampler::finalize()` 的总量改以**实际落盘量**为准，
      仅在目录完全扫不到内容时才回落到内核报告值。

    刻意**不**调低 `DONE_PERCENT`：那会让正在下载的文件在 99.5% 就提前显示完成，
    属于用一个错误掩盖另一个错误。两条路径互补 —— 对会渲染终帧的内核版本，
    原有的 `percent >= 99.5` 判定依然生效。

---

## 开发脚本

| 命令 | 说明 |
| --- | --- |
| `npm run dev` | 仅启动前端（浏览器预览模式，后端走 `src/lib/mock.ts` 模拟实现） |
| `npm test` | 前端单元测试（vitest） |
| `npm run kernel:fetch` | 预置 tdl 内核；`--force` 强制重新下载 |
| `npm run tauri:dev` | 开发模式（Vite + Tauri 窗口） |
| `npm run tauri:build` | 打包 NSIS 安装包 |
| `npm run release` | 一键发布到发布仓库（详见下节） |
| `npm run release:dry` | 发布预演（只渲染与检查，不推送、不调接口） |
| `cargo test` | Rust 单元测试 + 集成测试（在 `src-tauri/` 下执行） |

---

## 发布流程（对外分发）

对外分发使用独立仓库 **[t3nc/TgDL_repo](https://github.com/t3nc/TgDL_repo)**：

- 发布仓库里**只有文档与校验和**（`README.md` / `UPDATELOG.md` / `SHA256SUMS.txt` / `release-index.json`），
  安装包作为该仓库的 **Release 资产**上传 —— 不进入 Git 历史，所以发布仓库体积恒定在几十 KB，
  源码仓库可视需要设为私有；
- 文档是**单向同步**的：源码仓库是唯一事实源，发布仓库的内容由脚本生成，请勿手改。

### 一键发布

```bash
# 1) 升级三处版本号（必须一致，否则脚本直接拒绝执行）
#    package.json            → version
#    src-tauri/tauri.conf.json → version
#    UPDATELOG.md            → 顶部追加 ## [x.y.z] - YYYY-MM-DD 条目

npm run release          # 完整发布
npm run release:dry      # 先预演，确认无误再正式发布
```

脚本 `scripts/publish-release.mjs` 依次完成：

1. 校验 `package.json` / `tauri.conf.json` / `UPDATELOG.md` 三处版本一致（缺失更新日志条目 → 报错退出）
2. 定位安装包；缺失时自动执行 `npm run tauri:build`
3. **校验产物文件名包含当前版本**，防止把上一次构建的旧安装包发出去
4. 计算 SHA256，渲染并提交 `README.md` / `UPDATELOG.md` / `SHA256SUMS.txt` / `release-index.json`
5. 创建（或更新）`vX.Y.Z` Release，上传 ASCII 命名的安装包与校验和文件
6. 打印 Release 地址与 SHA256 汇总

### 参数

| 参数 | 作用 |
| --- | --- |
| `--dry-run` | 预演：渲染结果写入临时目录并打印计划变更，不推送、不调接口 |
| `--version <x.y.z>` | 显式指定版本（必须与 `package.json` 一致，作为二次确认） |
| `--repo <owner/name>` | 覆盖发布仓库（默认 `t3nc/TgDL_repo`） |
| `--docs-only` | 只同步文档，不创建 Release、不上传安装包 |
| `--skip-build` | 产物缺失时不自动构建，直接报错 |
| `--keep-asset` | Release 上已有同名资产时跳过上传（默认替换） |
| `--allow-missing-changelog` | 跳过 UPDATELOG 条目校验（应急用） |
| `--tag-source` | 同时给源码仓库打 `vX.Y.Z` 标签并推送 |
| `--keep-temp` | 预演时保留临时工作区，便于检查渲染结果 |

### 令牌配置（只需一次）

创建 Release 与上传资产需要 GitHub 令牌，按以下优先级自动查找：

1. 环境变量 `TDL_RELEASE_TOKEN` / `GITHUB_TOKEN` / `GH_TOKEN`
2. 仓库根目录 `.release-token` 文件（已 gitignore，内容就是令牌本身）
3. `git credential fill`（本机 Git 凭据管理器缓存，GitHub 账号密码口令即可）
4. `gh auth token`（安装了 GitHub CLI 时）

推荐用 **fine-grained PAT**：仅勾选发布仓库 → `Contents: Read and write`，有效期自定。
把令牌写进 `.release-token` 即可长期免配：

```powershell
'github_pat_xxx' | Set-Content -NoNewline .release-token
```

> 发布仓库的本地副本保存在 `.release-workspace/`（已 gitignore）。脚本每次会把它
> 同步到 `origin/main` 再生成内容，因此下载索引能跨版本累积；删掉它也不会丢历史。

---

## 更新日志

所有版本变更记录在 [UPDATELOG.md](./UPDATELOG.md)，**每次更新后必须同步更新该文件**：
`npm run release` 会以该文件中对应版本的段落作为 Release 说明，缺少条目即拒绝发布。
格式遵循「版本号 + 日期 + 新增 / 修复 / 变更」三段式，最新条目置于文件顶部。

---

## 许可证

本项目为 [tdl](https://github.com/iyear/tdl) 的图形化封装，tdl 内核遵循其自身许可协议；
本客户端源码暂未指定开源许可证，如需分发请先联系作者。

</details>
