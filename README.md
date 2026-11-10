# 交付说明（先读我）

内网离线安装包：**Claude Code（仅本体）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.319` → `2.1.322`**
> （中间跳过了 2.1.320；**2.1.321 是个大版本**，本包已包含其全部改动）
> 本包只含 Claude Code 本体；模型配置直接写 `~/.claude/settings.json`（见第五节）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.322-linux-x86_64.tar.gz` | 完整离线包（**156 MB**，解包后 336 MB） |
| `claude-offline-2.1.322-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件 |

```
sha256 = 6de2354a1194bc4277d9bda46aeb3808a0e4ffed9da245a3c1f77a8edd42d20b
```

---

## ⚠️ 本版有 2 处行为变更，升级前请确认

### 1) 网关的 Bedrock / Vertex / Foundry 上游不再读 `ANTHROPIC_*_BASE_URL`

> 如果你**在同一台机器上跑 Claude apps gateway**，且靠环境变量
> `ANTHROPIC_BEDROCK_BASE_URL` 之类指向自定义端点，现在**必须在 upstream 里用 `base_url` 显式设置**。
> 单纯跑 Claude Code 客户端（不经 gateway）**不受影响**。

### 2) `policy_mode: layered` 的托管 MCP 服务器改为"叠加"

> 之前只取匹配到的**第一条**策略，现在**所有匹配策略的托管 MCP 服务器都会累加**（含 `match: {}`）。
> 已经在用 layered 的网关升级后会**多出一些 MCP 服务器** —— 升级后建议核对一次 `claude mcp list`。

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.322（原生二进制，326.1 MB） | 官方 CDN 产物，sha256 `bfb3703a…c478` **与官方 manifest 完全一致** |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = bfb3703ad5380242328459bc62ada4bde8ae7ba57c6e1eeadb9c00e08874c478
            size   = 341,914,128 bytes
```

> **下载方式**：国内镜像高速获取（21.6 MB/s，7 秒），再**按官方 manifest 逐字节校验通过**，
> 并随包附带官方 `manifest.json`（2,101 B）。

## 二、本版更新要点

### 2.1.322（独立条目只有 1 条）

- **Vertex 在 global / us / eu 区域支持 fast mode**，Vertex 拒绝时自动回落到标准速度

### 2.1.321 —— 与你最相关的 10 条

| # | 变更 | 影响 |
|---|---|---|
| 1 | **修 MCP OAuth 在服务器重定向时把 refresh token 等机密发给另一个 host** | **安全**：内网多 MCP 服务器时要升 |
| 2 | **修"只读 Bash 命令跑到沙箱外且不弹权限提示"** | **安全**：沙箱逃逸 |
| 3 | **修 Bash 权限检查按错目录判断写操作、导致没弹提示** | **安全**：写保护失效 |
| 4 | **修 deny 规则与受保护文件提示没覆盖 `cp`/`mv` 写进目录的那个文件** | **安全**：`cp x 受保护目录/` 绕过 |
| 5 | **`cp`/`mv` 用路径写法（如 `/bin/cp`）现在也走同样的路径检查** | **安全**：绕过封堵 |
| 6 | **修 `-p` 在 Linux 上"满 NFS 或配额用尽丢 transcript 行"时静默退出 0** —— 现在报错并退出 1 | **headless + NFS**：脚本不再误判成功 |
| 7 | **修 `NotebookEdit` 在网络路径（UNC / WSL / `/net`）上删或覆盖 notebook 第一个单元格** | NFS 场景数据损坏 |
| 8 | **修 `Edit` 在网络路径上改错匹配项、空 `old_string` 替换整个文件、把 notebook 当文本编辑** | NFS 场景数据损坏 |
| 9 | **修 `API Error: EPROTO: TLS handshake failed` 在回复中途断连时整轮失败** —— 恢复为保留部分回复或重试（**2.1.302 前的行为**） | 内网链路不稳时很关键 |
| 10 | **修"网络对端接受连接但永不应答 TLS 握手"时登录/刷新令牌/登出**无限等待 | 内网防火墙黑洞场景 |

### 其他值得知道的

**稳定性 / 内存**
- 修长会话反复读取大文件单行导致的内存高占用
- 修 transcript 读取时内存耗尽导致新消息不保存、`-p`/SDK/VS Code 挂起
- 修权限提示与后台任务相关的偶发崩溃
- 修同时在跑的多个 Bash 命令"前一个改了目录"导致后续命令跑错目录
- 修 zsh 用户 `.zshrc` 设了 `KSH_OPTION_PRINT` 时 Bash 命令返回空

**Bash / 权限体验**
- `grep -r foo "$PWD/src"` 这类写法不再被要求授权（与写全路径一致）
- `pkill`/`pgrep`/`pidwait` 在云会话里不再匹配会话自身父进程

**MCP**
- 修重连或切目录后 MCP 服务器"显示已连接但没有工具"
- 修存储的 MCP 登录记录为空时启动白屏 / `claude mcp list` 输出空
- 修 `PreToolUse` 的 `mcp_tool` 钩子在返回 >16 MiB 时被跳过（改为直接拒绝该调用）

**网关**
- spend limit 的重置时间改为本地时间显示；错误信息不再泄露用户名/密码
- 修 `desktop` 设置含比网关更新的字段时网关拒绝启动（改为警告并服务）
- 修 busy replica 上的 spend limit 检查（更少跳过、更早记账）

**headless / 安装**
- headless 会话中 `.claude/skills` 的 mod 文件变化会重载；`CLAUDE_CODE_PLUGIN_DIR_WATCH=0` 可关
- 超过 1000 字符的后台命令通知只展示首尾，省 context
- `claude install` 与安装脚本快 2–4 秒；启动时 TLS 证书加载更快
- Windows 启动与首命令更快（找 PATH 不再起辅助进程；标准 Git for Windows 不再每次检查 git.exe）

**模型**
- 新增 **Claude Fable 5.2**（`claude-fable-5-2`）作为默认 Fable 模型：1M 上下文，$8/$40 每 Mtok
- 已保存的 Fable 5.1 选择会迁移到 `fable`（`/model claude-fable-5-1` 可切回）
- `/model` 选择器改进：「More models」改为末尾不带编号的行

**其他**
- `claude remote-control` 新增 `--settings <file>`，对其启动的每个会话生效
- 修 WebFetch 在模型没读到内容时把 "No response from model" 当作页面正文传给 Claude（改为明确报错）
- 大量 Remote Control / 云会话 / Claude Desktop / VS Code 修复（与本包无关）

## 三、目标机要求

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

## 四、安装（3 步）

```bash
# 1) 校验
sha256sum -c claude-offline-2.1.322-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/）
tar -xzf claude-offline-2.1.322-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

```bash
source ~/.bashrc
claude --version      # 期望：2.1.322 (Claude Code)
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

> **离线安装 ≠ 离线调用模型**。

## 六、常用参数

```bash
bash install-offline.sh --link           # 省 326MB（符号链接，但离线包不能删）
bash install-offline.sh --system-glibc   # 用目标机 glibc（需 ≥2.26）
bash install-offline.sh --force          # 覆盖已安装（会先备份）
bash install-offline.sh --no-path        # 不改 shell rc
```

## 七、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/versions/2.1.322   # 二进制 326.1 MB
~/.local/share/claude/runtime/           # 随身运行时（5.9 MB）
~/.local/bin/claude                      # 启动器（#!/bin/sh）
~/.claude.json                           # installMethod=native, autoUpdates=false
~/.claude/settings.json                  # 模型端点与 Key（需自行创建）
```

安装后实测占用：**332 MB**（含 5.9 MB 运行时）。

## 八、卸载

```bash
rm -rf ~/.local/share/claude ~/.local/bin/claude ~/.claude.json
# 想连模型配置一起清：rm -rf ~/.claude
# 再删掉 ~/.bashrc 里 "# >>> claude-code >>>" ~ "# <<< claude-code <<<" 之间的行
```

## 九、注意事项

1. `claude` 首次交互运行会尝试登录 Anthropic 官方账号；内网请改用 API Key + 自建 `BASE_URL`。
2. 随身运行时只解决 **glibc 版本**问题，不改变架构与内核要求（x86_64、内核 ≥ 2.6.32）。
3. 已自动写入 `DISABLE_AUTOUPDATER=1` 与 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1`。
4. **升级**：`bash install-offline.sh --force`（先备份旧目录），`~/.claude/settings.json` 会沿用。
5. **本版修了 5 个权限/沙箱绕过问题**，建议内网机器尽快升级。
6. 若在同一台机器跑 Claude apps gateway，注意第一节的第 1 条变更。

## 十、分批传输

```bash
# 发送端
split -b 90M -d claude-offline-2.1.322-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.322-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.322-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、重建步骤）在归档内 `claude-offline/README.md`。
