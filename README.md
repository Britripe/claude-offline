# 交付说明（先读我）

内网离线安装包：**Claude Code（仅本体）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.314` → `2.1.316`**
> 本包只含 Claude Code 本体；模型配置直接写 `~/.claude/settings.json`（见第五节）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.316-linux-x86_64.tar.gz` | 完整离线包（**149 MB**，解包后 323 MB） |
| `claude-offline-2.1.316-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件 |

```
sha256 = e6b930d1506820d2420e715d1b6e06f673b93922a68ecc473de0ed6bd1951d2a
```

**归档内部**：`README.md`（完整文档）、`UNPACK-FIRST.md`（速查）、2 个安装脚本、
构建脚本、官方 `manifest.json`、随身运行时及其来源 `.deb`。

---

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.316（原生二进制，313.1 MB） | 官方 CDN 产物，sha256 `fd13c700…d824` **与官方 manifest 完全一致** |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = fd13c70000fec3f2116dfbcbb6121210b4a29328bf0e46210a59dd594a67d824
            size   = 328,262,000 bytes
```

> **下载方式**：国内镜像高速获取（23.8 MB/s，6 秒），再**按官方 manifest 逐字节校验通过**，
> 并随包附带官方 `manifest.json`（1,965 B）。
> 注意：本版二进制比 2.1.314 **增大 30 MB**（298 → 328 MB），属正常体积增长。

## 二、本版更新要点（2.1.316）

> 官方 changelog 无独立的 2.1.315 段，版本从 2.1.314 直接到 2.1.316。

### 与你（内网 + 自建网关 + 私有 CA）最相关 —— 9 条

| # | 变更 | 影响 |
|---|---|---|
| 1 | **新增 `CLAUDE_CODE_DISABLE_LOOPED_CROSS_CERT_DROP`** | 保留"互相交叉签名"的系统根证书，**专为私有 CA 需要它的服务器**。内网自建 CA 证书链里存在环形交叉签名时，默认会被丢弃导致握手失败 |
| 2 | **SSL 证书报错更具体** | 现在会**指出可能原因**（如服务器漏发中间证书）及修法，内网 TLS 排查更省时间 |
| 3 | **修 `sonnet`/`opus`/`haiku` 在 Vertex/Bedrock 上中途报错** | 账号/区域拿不到最新模型时，改为**回退该家族旧模型**，不再中断会话 |
| 4 | **修 Claude apps gateway 未校验请求里"每一个"模型名** | 现在逐个对照 `availableModels` / `deniedModels` / `models`，被拦的模型名返回 400 并指明字段 |
| 5 | **修 `/model <id>` 在模型不被识别时不警告** | 之前会**静默按 200K 上下文**做自动压缩；现在会提示 |
| 6 | **修 `CLAUDE_CODE_AUTO_MODE_SERVER=1` 被忽略** | 出现在"经代理/LLM 网关且关闭实验性 beta"的会话里 |
| 7 | **新增 `CLAUDE_CODE_SUBAGENT_EFFORT`** | 单独设定子代理 effort 档位，不再继承主会话 |
| 8 | **新增 `CLAUDE_CODE_DISABLE_ULTRATHINK`** | 关闭 `ultrathink` 关键字 |
| 9 | **修 Claude Code 自行退出时遗留 shell 子进程**（Linux/macOS） | 凭证被拒、云会话归档等场景不再留挂起进程 |

### 稳定性 / 兼容

- 压缩响应被截断或损坏时网络请求挂死 —— 已修
- `--restricted` 会话在 Read allow 规则覆盖时仍会外传工作目录外文件 —— 已修（安全）
- 含不可见字符的提示词需要按两次回车 —— 已修
- 子代理继承"父级子代理模型"后被唤醒会跳到主模型 —— 已修
- Bash 工具：别名值含换行、函数名以 `-` 开头被破坏 —— 已修
- Bash 工具：导出 `SHELLOPTS` 时脚本"只跑第一条命令就停" —— 已修
- Windows：Git Bash 设 `errexit`+`pipefail` 时丢别名/函数 —— 已修
- Windows：Grep/Glob 在搜索进程被超时杀掉但未退出时永不返回 —— 已修
- 恢复会话时偶发未还原 plan —— 已修

### 企业 / 策略

- Team/Enterprise 的 server-managed settings：从 Anthropic API 拉取的设置**不再需要批准提示**；已存副本校验失败则拒绝使用
- `autoMode.allow` 未写 `"$defaults"` 时不再被塞回内置例外规则
- Claude apps gateway：`managed.policy_mode: layered`（多层策略叠加）、分组 `pooled_budget`（团队共享月度预算）、`guardrail.input_tagging`
- `claude auth login` 在托管设置要求走 gateway 时不再拒绝运行（终端里显示 gateway 登录）

### 性能 / 体验

- `/diff` 在超大仓库：每次刷新少跑一次全仓 git 命令；git 慢时自动拉长刷新间隔
- `/code-review` 在云/remote-control 会话显示 findings 卡片
- 大量 VSCode / 云会话 / Slack / Code Review 修复（与本包无关）

### ⚠️ 弃用提醒

**Opus 4.8 的 fast mode 自 2026-10-30 起弃用，2026-11-30 移除**；移除后 Opus 4.8 的 fast 请求
直接报错且**不回落**标准模式。需要加速请迁移到 **Opus 5.5**。

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
sha256sum -c claude-offline-2.1.316-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/）
tar -xzf claude-offline-2.1.316-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

```bash
source ~/.bashrc
claude --version      # 期望：2.1.316 (Claude Code)
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
bash install-offline.sh --link           # 省 313MB（符号链接，但离线包不能删）
bash install-offline.sh --system-glibc   # 用目标机 glibc（需 ≥2.26）
bash install-offline.sh --force          # 覆盖已安装（会先备份）
bash install-offline.sh --no-path        # 不改 shell rc
```

## 七、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/versions/2.1.316   # 二进制 313.1 MB
~/.local/share/claude/runtime/           # 随身运行时（5.9 MB）
~/.local/bin/claude                      # 启动器（#!/bin/sh）
~/.claude.json                           # installMethod=native, autoUpdates=false
~/.claude/settings.json                  # 模型端点与 Key（需自行创建）
```

安装后实测占用：**319 MB**（含 5.9 MB 运行时）。

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
5. 私有 CA 环境下若遇证书链问题，试 `export CLAUDE_CODE_DISABLE_LOOPED_CROSS_CERT_DROP=1`（本版新增）。

## 十、分批传输

```bash
# 发送端
split -b 90M -d claude-offline-2.1.316-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.316-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.316-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、重建步骤）在归档内 `claude-offline/README.md`。
