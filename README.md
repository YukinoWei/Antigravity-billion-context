# Billion Context Studio (v3.0)

面向 **Google Antigravity** 的原生上下文压缩与配额管理插件。采用纯 **Python 3.12 标准库**实现（零外部 `pip` / `npm` 依赖），帮你解决长对话上下文爆窗、KV-Cache 失效、大文件/日志重复消耗 Token 与会话卡死问题。

---

## 核心功能

- **三级 LSM 主动上下文压缩**：支持将已完成的历史步骤折叠为 `T1 记忆块 → T2 里程碑 → T3 纪元`（压缩率最高 99%+），硬保护近期工作尾区；执行 `/compact` 后自动回注 LSM 记忆索引与 Pins，并支持通过 `search_context` / `decompress` 跨会话无损还原原文。
- **动态模型窗口 & 1.5s 实时同步**：自动识别当前模型上下文上限（如 `256K` / `512K` / `200K` / `1M`），控制台每 **1.5 秒**自动实时刷新水位与缓存状态，无需手动刷新；新开空白对话自动重置，不同会话数据严格隔离。
- **KV-Cache 热力监控 & 自动续温**：实时统计 15 轮缓存命中趋势与 TTL 倒计时，支持 `关闭 / 自适应 / 4m / 9m / 15m` 自动保活续温，避免百万级前缀冷启动重算。
- **全链路省 Token 护栏**：
  - 四级读文件配额护栏（超限大文件首读返回符号大纲，二连读放行）与重复读去重；
  - 可选 **增量 Diff 读取守卫 (`diff_read`)**，修改后重读直接返回 `unified_diff` 差异块；
  - 终端输出自动降噪与大 JSON 粉碎（内联严格 `≤ 8.8 KB`），自动拦截 `.env`/私钥并脱敏密钥。
- **原生 Watchdog 看门狗 & 一键崩溃抢救**：后台守护检测卡死会话并自动生成 `acp_rescue.md`，一键复制脱敏抢救简报粘贴到新对话继续。
- **原创 Studio 控制台 & 悬浮窗 HUD**：内置暗色下拉菜单式 Studio 控制台（适配侧边栏与全局栏），Windows 另附 DWM 硬件圆角 + 抗锯齿底部对话栏悬浮窗。

---

## 安装方法

### 环境要求
- **Python 3.12+**（无需安装任何第三方依赖）
- **Google Antigravity**（Windows / macOS / Linux）

### 方式一：源码一键安装（推荐）

```powershell
# 1. 克隆仓库或解压源码发布包
git clone https://github.com/<YukinoWei>/billion-context-studio.git
cd billion-context-studio

# 2. 一键安装到 ~/.gemini/config/plugins/billion-context 并热重载 MCP
python bili.py install
# Windows 也可以直接运行：
.\bili.cmd install
```

### 方式二：插件包手动解压安装

1. 从 Release 下载 `billion-context-plugin-v3.0.0.zip`。
2. 解压至 Antigravity 插件目录（确保目录内直接包含 `plugin.json`）：
   - **Windows**：`C:\Users\<你的用户名>\.gemini\config\plugins\billion-context\`
   - **macOS / Linux**：`~/.gemini/config/plugins/billion-context/`
3. 重启一次 Antigravity（或在 **MCP Servers** 中重启 `billion-context_acp`）即可生效。

### 卸载

```powershell
python bili.py uninstall
```

---

## 使用方法

### 1. 打开插件控制台 UI
在 Antigravity 右上角 **「插件」** 菜单或左侧栏点击 **「Billion Context 控制台」**：
- 顶栏显示 **`● 实时`** 状态、当前模型动态上下文水位（`256K` / `512K` 等）与 KV-Cache 热力倒计时；
- 通过**下拉菜单栏**切换 4 个视图：`概览与自动续温`、`看门狗与崩溃抢救`、`记忆树与知识库`、`配额护栏与设置`；
- 在卡片上可一键点击 **`⚡ 生成续接点`**、**`📦 折叠推荐区间`**、**`⚗️ 蒸馏 T1→T2`** 或 **`🛟 抢救`**。

### 2. 对话中压缩与恢复上下文
插件注册了 `billion-context_acp` MCP 工具，AI 会自动或在你的指示下调用：
- **折叠历史步骤**：`compress(operation="capture", range_ref="s00000..s00050", ...)`
- **蒸馏/凝炼记忆块**：`compress(operation="distill", block_ids=[...])` / `operation="condense"`
- **跨会话检索与还原**：`search_context(query="关键词")`、`decompress(block_id="b00001")`
- **固定不变量与无损 `/compact`**：`pin_context(...)`、`handoff_checkpoint(action="create")`，随后执行 `/compact` 释放物理窗口，插件会自动回注核心记忆索引。

### 3. 常用命令行 (`python bili.py` / `.\bili.cmd`)

| 命令 | 说明 |
| :--- | :--- |
| `python bili.py status` | 查看当前会话上下文水位、缓存命中率、节省账本与 LSM 记忆树 |
| `python bili.py doctor` | 检查安装状态、SQLite FTS5、Watchdog 心跳与 `agentapi` 健康度 |
| `python bili.py audit` | 体检 MCP Turn-0 前缀 Token 开销与本地 Skills 开销 |
| `python bili.py mode <preset>` | 切换读文件护栏：`host-45k` / `fast-32k` / `balanced-16k` / `strict-8k` / `off` |
| `python bili.py keepalive <mode>` | 切换自动续温：`off` / `auto` / `4m` / `9m` / `15m` |
| `python bili.py rescue` | 生成当前会话的脱敏崩溃抢救简报 |
| `python bili.py widget` | 启动底部对话栏悬浮窗 HUD（Windows） |

---

## 测试

```powershell
python -m unittest discover -s tests -v
```

---

## 致谢与协议

- 核心理念借鉴并融合自 [`ranxianglei/billion-context`](https://github.com/ranxianglei/billion-context)（`acp-kernel`）与 [`zou172866/Antigravity-agy-dart`](https://github.com/zou172866/Antigravity-agy-dart)。
- 采用 [MIT License](LICENSE) 开源。
