# 交付说明（先读我）

内网离线安装包：**Claude Code（仅本体）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.328` → `2.1.330`**（含 2.1.329 + 2.1.330 两版）
> 本包只含 Claude Code 本体；模型配置直接写 `~/.claude/settings.json`（见第五节）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.330-linux-x86_64.tar.gz` | 完整离线包（**171 MB**，解包后 373 MB） |
| `claude-offline-2.1.330-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件（**含完整更新说明**） |

```
sha256 = 0feec743c6c26325e921694c0c65681cdb47f6604e39a89529d373a236a066ce
```

---

# 第一部分：更新说明（2.1.328 → 2.1.330）

## ⭐⭐⭐ 与你（网关 + 关遥测 + WSL）最相关的 8 条

| # | 变更 | 说明 |
|---|---|---|
| 1 | **`max_tokens` 超模型/网关上限时的自动重试：2.1.329 移除 → 2.1.330 恢复** | 净结果是有重试。但若网关后面模型输出上限更小，**建议设 `CLAUDE_CODE_MAX_OUTPUT_TOKENS` = 该上限** |
| 2 | **未识别模型名里含 "sonnet" 或 "haiku" 的改用精简 system prompt**；`CLAUDE_CODE_SIMPLE_SYSTEM_PROMPT=0` 让所有模型用完整版 | **直接命中你的网关模型名** |
| 3 | **修「Linux/WSL 上睡眠 ≥2 分钟后，首条提示在已死连接上等 3 分钟」** | 你的网络 + WSL |
| 4 | **Code Mode 现在在"关遥测 / 关非必要流量 / 关特性开关"时也能启动**（管理员仍可禁用） | 你正好是这个配置 |
| 5 | **修「关遥测、或经网关访问 Anthropic API、或在 Foundry 上时，auto 模式对 'yes' 这类短回复处理不当」** | 直接命中 |
| 6 | **修「`~/.claude.json` 在网络卡顿/文件锁导致读不出并在保存时被清空，从而重置为默认值」** | **数据丢失** |
| 7 | **修「大量 MCP 工具的 input schema 被 API 拒绝时，整轮报 `JSON schema is invalid` 且会话文件涨到几 MB」** | MCP + schema |
| 8 | **修「WebFetch 摘要忽略 `/effort` 档位（网关拒绝小快模型时）」** | 网关 |

## ⚠️ 行为变更

| # | 变更 |
|---|---|
| 1 | **已保存的定时任务：新登录会话中，早期版本保存的大多数需要重新排期**（`--resume` 原会话仍能跑） |
| 2 | **更多针对根目录 / 家目录 / 顶层目录 / 当前工作目录的 `rm` 改为需审批**；无人可答时直接拒绝 |
| 3 | 反复定时任务与 `/loop` 现在可跑 **30 天**（原 7 天），结束时告知 |
| 4 | 网关：`access_control.allow_cidrs` 为空时**每次启动打带框提示：绝不要把网关暴露到公网** |
| 5 | 部分 teammate 面板改由 `/bin/sh` 启动，**可能不再读取 shell 启动文件** |
| 6 | Code Mode 里 PreToolUse 钩子报错**不再拒绝脚本的工具调用**；要拒绝请设 `"onFailure": "block"` |

## 一、压缩相关

- **修「压缩完超长会话后冻结可达数分钟」**
- **修「从预热进程启动的会话忽略项目 `.claude/settings.json` 的 `autoCompactWindow`」**
- **修「`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` 从 settings.json 删除后运行中会话仍生效」**

## 二、headless / 脚本

- **修「`claude -p` 输出到满盘文件时退出码 0 且无任何消息」** → 现在报错退出 1（满 NFS 导出仍可能不报）
- 修「SessionStart 钩子写入的环境变量在 `claude -p --continue` 时丢失」
- 修「多个进程同时保存用户设置时改动丢失」
- 修「磁盘满时保留不完整 `~/.claude.json` 备份、把好备份挤掉」
- 新增 `CLAUDE_CODE_FULL_SIGN_IN_LINK=1`（短链接出错时打印完整链接）

## 三、性能

- 文件 diff 更快
- **修「Linux 慢仓库（`node_modules` 很大）里 Bash 命令等数秒，且之后整个会话不显示文件 diff」**
- Windows：Grep/Glob 不再为每次内置 ripgrep 起 console host；启动与首次搜索更快
- 修「Code Mode 读数百文件后（有路径范围/Read deny 规则时）长时间停顿」
- **auto 模式减少 `git status` 调用**，大仓库里等待更短
- 长会话（编辑大量文件）与含大量 MCP 结果的 Desktop 会话内存改善

## 四、新功能

| 项 | 说明 |
|---|---|
| **easel** | 内置图表 mod：被要求画图时、MCP 指标结果下自动出图（Pro/Max，macOS/Linux；`/config` 可关） |
| Enterprise Admin Console 登录 | `claude auth login` 后自动应用组织设置并使用其云厂商/网关 |
| `/tasks` 扩展 | 含本会话定时任务、`/loop` 唤醒、goal 与 fork |
| `/mcp` 新增「Server message」行 | 显示远程 HTTP MCP 服务器拒绝连接的 JSON-RPC 错误 |
| `/status` 新增「Marketplace auto-update」行 | 说明哪些托管市场不自动更新及原因 |
| `claude plugin install --json` | 新增 `commandPrinted` / `commandFailed` |

## 五、其他重要修复

- **修「状态栏脚本无限输出会冻结提示框并吃掉数 GB 内存」**
- **修「Code Mode 失控循环把内存吃满」**（现在有上限）
- 修「MCP 服务器在慢重连后停滞」——多数 HTTP MCP 服务器现在会自动重连（停留旧协议版本时）
- 修「PDF/图片/`.docx`/`.xlsx`/`.pptx` 以 `application/octet-stream` 发送时被存成 `.bin`，导致读不了」
- 修「`/code-review <PR号>` 有时审检出分支而非该 PR 且不说明」→ 改用 `gh pr diff`，读不到会说明
- 修「`forceLoginOrgUUID` 下一次临时服务器错误或连接中断后启动被拒」
- 修「`/stats` 按 UTC 而非本地时区统计」
- 修「Jupyter notebook 带 BOM 时被 Read/NotebookEdit 判为非法 JSON」
- 修「CLAUDE.md / 记忆文件含 HTML 注释时链接引用定义被丢弃」
- 修「Grep 模式含换行且未开 multiline 时报无匹配」→ 现在报告模式被拒
- 修「两个用户/项目 agent 文件同名时选哪个取决于搜索顺序」
- **登录链接大幅缩短**（`/login`、`claude auth login`、`claude setup-token`）
- 大量 VSCode / Remote Control / 自托管 runner 修复（与本包无关）

---

# 第二部分：安装与验证

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.330（原生二进制，363.5 MB） | 官方 CDN 产物，sha256 `f390eaa0…8ebb` **与官方 manifest 完全一致** |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = f390eaa0814e6a1bff74cbb47acb74cb9c89d30ced809fe0af5af00c7bfe8ebb
            size   = 381,121,040 bytes
```

> **下载方式**：国内镜像获取（首次成功），再**按官方 manifest 逐字节校验通过**，
> 并随包附带官方 `manifest.json`（2,101 B）。

## 二、目标机要求

| 项 | 要求 |
|---|---|
| 架构 / 系统 | **Linux x86_64** |
| **glibc** | **无要求** —— 跑在自带运行时上（内核 ≥ 2.6.32） |
| **图形界面** | **不需要** —— 纯 CLI |
| 系统 Node | **不需要** |
| 必备命令 | 安装：`bash`、`tar`、`gzip`、`sha256sum`；运行：任意 `/bin/sh` |
| root | **不需要** |
| 网络 | 安装**不需要**；运行时需要能访问你配置的模型端点 |

> 已实测：用 tmpfs 一次性盖住系统 `/usr/lib/x86_64-linux-gnu` 与 `/usr/lib64`
> （掩蔽后两处条目数均为 0，系统 ELF 全部无法启动 —— `/usr/bin/true` 退出码 127）后，
> Claude Code 靠随身运行时里的 loader 仍能正常运行。
> 启动器 shebang 是 `#!/bin/sh`（实测 `dash` 可跑，目标机不必装 bash）。

## 三、安装（3 步）

```bash
# 1) 校验
sha256sum -c claude-offline-2.1.330-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/）
tar -xzf claude-offline-2.1.330-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

```bash
source ~/.bashrc
claude --version      # 期望：2.1.330 (Claude Code)
```

## 四、常用参数

```bash
bash install-offline.sh --link           # 省 363MB（符号链接，但离线包不能删）
bash install-offline.sh --system-glibc   # 用目标机 glibc（需 ≥2.26）
bash install-offline.sh --force          # 覆盖已安装（会先备份）
bash install-offline.sh --no-path        # 不改 shell rc
```

## 五、配置模型端点

```bash
mkdir -p ~/.claude
cat > ~/.claude/settings.json <<'EOF'
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://内网网关地址:端口",
    "ANTHROPIC_AUTH_TOKEN": "你的密钥",
    "ANTHROPIC_MODEL": "模型名"
  }
}
EOF
chmod 600 ~/.claude/settings.json      # 内含密钥，收紧权限
```

- `ANTHROPIC_BASE_URL` 必须**内网可达**。
- 若网关用 API Key，键名换成 `ANTHROPIC_API_KEY`。
- 自定义模型名默认按 1M 上下文估算；想回 200K 加 `"CLAUDE_CODE_DISABLE_1M_CONTEXT": "1"`，
  或精确设定 `"CLAUDE_CODE_MAX_CONTEXT_TOKENS": "<数值>"`。
- 网关输出上限较小：加 `"CLAUDE_CODE_MAX_OUTPUT_TOKENS": "<数值>"`。
- 若用沙箱且想免提示：加 `"sandbox": {"autoAllowBashIfSandboxed": true}`。
- 内网链路慢：可设 `"CLAUDE_STREAM_IDLE_TIMEOUT_MS": "600000"`（2.1.328 起在网关上真正生效）。

> **离线安装 ≠ 离线调用模型**。

## 六、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/versions/2.1.330   # 二进制 363.5 MB
~/.local/share/claude/runtime/           # 随身运行时（5.9 MB）
~/.local/bin/claude                      # 启动器（#!/bin/sh）
~/.claude.json                           # installMethod=native, autoUpdates=false
~/.claude/settings.json                  # 模型端点与 Key（需自行创建）
```

安装后实测占用：**370 MB**（含 5.9 MB 运行时）。

## 七、卸载

```bash
rm -rf ~/.local/share/claude ~/.local/bin/claude ~/.claude.json
# 想连模型配置一起清：rm -rf ~/.claude
# 再删掉 ~/.bashrc 里 "# >>> claude-code >>>" ~ "# <<< claude-code <<<" 之间的行
```

## 八、注意事项

1. `claude` 首次交互运行会尝试登录 Anthropic 官方账号；内网请改用 API Key + 自建 `BASE_URL`。
2. 随身运行时只解决 **glibc 版本**问题，不改变架构与内核要求（x86_64、内核 ≥ 2.6.32）。
3. 已自动写入 `DISABLE_AUTOUPDATER=1` 与 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1`。
4. **升级**：`bash install-offline.sh --force`（先备份旧目录），`~/.claude/settings.json` 会沿用。
5. **本版修了 `~/.claude.json` 被重置为默认值的数据丢失问题**，建议尽快升级。
6. 升级后留意：**旧定时任务可能需要重新排期**。

## 九、分批传输

```bash
# 发送端
split -b 90M -d claude-offline-2.1.330-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.330-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.330-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、重建步骤）在归档内 `claude-offline/README.md`。
