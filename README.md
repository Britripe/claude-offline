# 交付说明（先读我）

内网离线安装包：**Claude Code（仅本体）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.325` → `2.1.327`**（含 2.1.326 + 2.1.327 两版内容）
> 本包只含 Claude Code 本体；模型配置直接写 `~/.claude/settings.json`（见第五节）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.327-linux-x86_64.tar.gz` | 完整离线包（**168 MB**，解包后 365 MB） |
| `claude-offline-2.1.327-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件（**含完整更新说明**） |

```
sha256 = d8b42e5f5dda79d1df4fe23798e4fa3d76af4afc7a51eae71343b311dae203da
```

---

# 第一部分：更新说明（2.1.325 → 2.1.327）

> 两版合计条目极多（300+），以下是**按你的使用场景筛过**的要点。

## ⭐ 与你（内网网关 / 关遥测 / 无头）最相关 —— 6 条

| # | 变更 | 影响 |
|---|---|---|
| 1 | **修「关遥测、以及 Bedrock / Vertex / Foundry 上的压缩，有时不是摘要而是回复了你某条消息，或漏掉 Claude 最后一条回复」** | **直接命中你的配置**（关遥测 + 网关）：之前压缩可能把会话搞坏 |
| 2 | **修后台会话丢失 shell 导出的 LLM 网关配置**（崩溃后、从别的目录重开、后台服务没有该变量）—— 现在会在任何模型请求前就停下 | 后台/常驻会话不再悄悄走错端点 |
| 3 | **修项目 `.claude/settings.json` 把 `ANTHROPIC_BASE_URL` 指向网关时 `/model` 列出错误的模型** | 网关 + 项目级配置组合 |
| 4 | **修「家目录或配置目录经符号链接指向 `/net` 且沙箱设置拒绝读取时，沙箱命令无法启动」** | NFS / autofs / 符号链接家目录 |
| 5 | **未识别模型的长回复：被截在 32,000 输出 token 时会以 128,000 上限继续** | 自定义模型名的长输出不再半截 |
| 6 | **auto 模式的提示改为预告：下个版本会移除「回退到计费分类器」** —— 之后每个本该检查的动作都会被询问或拒绝 | **提前准备**：要么接受询问，要么配好允许规则 |

## ⚠️ 行为变更（可能影响现有脚本）

| # | 变更 |
|---|---|
| 1 | **沙箱命令默认不再写入若干敏感家目录**；需要时用 `sandbox.filesystem.allowWrite` 放行 |
| 2 | **沙箱开启时会先询问"设置环境变量的命令行"**（auto-allow 关闭时）；无人值守场景会被拒绝 → 需要放行规则 |
| 3 | **未设置模式时会话可能以 auto 模式启动**（原本是 Manual），当组织策略尚未载入时 |
| 4 | 路径检查**开始拒绝部分符号链接**；"什么算工作目录之内"重新界定，**部分路径现在会要求授权** |
| 5 | `permissions.blockReadsOutsideWorkingDirectories` 在 bypass/auto 模式下**对更多 Bash 命令改为询问或拒绝** |
| 6 | Bash 命令里**变量展开成超长文本会被拒绝**（提示改用短值或文件） |
| 7 | Auto 模式**忽略更多"能跑任意包"的允许规则**（如 `Bash(pnpm dlx *)`、`/usr/bin/npm run:*`） |
| 8 | `ExitWorktree` 在 worktree 可能还有未保存工作时**会先询问** |
| 9 | 子代理被同级消息唤醒后**遵循会话的权限设置**（可能询问，或无人可答时被拒） |
| 10 | PowerShell 工具**对更多命令改为询问**（即使有指名规则；无人可答时拒绝） |
| 11 | `--add-dir` / `/add-dir` 对**未信任目录会弹信任确认**；`claude --bg` 直接拒绝此类目录 |
| 12 | Claude 的文件工具**拒绝写入 `.credentials.json` 等登录文件**；Bash 权限检查也会拦更多触及登录文件的命令 |

## 一、安全 / 沙箱加固

| # | 变更 |
|---|---|
| 1 | 沙箱在会话中**更倾向于保持状态**（直到你运行 `/sandbox` 或 `/cd`） |
| 2 | 命令后**会被清出仓库 `.git` 里异常条目** |
| 3 | Linux/WSL 上**每条沙箱命令前会创建缺失的 `--add-dir` 目录**（依赖"目录不存在"的命令可能失败或落进该目录） |
| 4 | 权限钩子/SDK 宿主重写过的命令**若原本按沙箱检查，则仍留在沙箱**（可用 `dangerouslyDisableSandbox: true` 例外） |
| 5 | `claude mcp serve` **忽略** `sandbox.excludedCommands` |
| 6 | `/rewind` 在开启沙箱时会**跳过部分文件并报告** |
| 7 | 更多 `cp` / `mv` 命令改为先询问 |
| 8 | bypass 模式下**部分工具调用改为询问或拒绝** |
| 9 | 部分文件工具**拒绝设备文件与进程文件** |
| 10 | 隔离 worktree 的子代理**拒绝部分会重写共享分支的 git 命令** |
| 11 | 改进 Code Mode 的权限检查（更多沙箱命令会询问/被拒） |

## 二、修复（与你环境相关）

| # | 修复 |
|---|---|
| 1 | `grep` 在 Bash 工具里**静默跳过部分文本文件** |
| 2 | `max_tokens` 超过模型上限时报错 → 现在**用 Anthropic API 报告的最大值重试** |
| 3 | Vertex 上图片多的会话**卡在 400（请求超 30 MB）** 而不压缩重试 |
| 4 | 设置文件嵌套数千层**导致崩溃或只有时生效**（超过 64 层的值会丢弃并告警） |
| 5 | 设置文件保存出错被**静默跳过**（现在告警并指名文件） |
| 6 | Linux 上沙箱命令中途被停时**留下空占位文件（含 `.git/config.lock`）** |
| 7 | `cp` 一个文件时**只因源被 `Edit` deny 覆盖就被拒** |
| 8 | 被拒的 MCP `headersHelper` **让其他会话也起不来该服务器** |
| 9 | 恢复会话时**丢失最近若干轮**（上次是空回复结束时） |
| 10 | 恢复会话时**低内存导致丢失更早历史** |
| 11 | `claude purge` 在 stdout 被重定向时**隐藏提示**（一次看不见的回车可能删掉项目数据）→ 现在显示在终端 |
| 12 | `claude auth logout` **残留凭据**、令牌刷新失败（仅 Console profile 凭据位于拒绝目录时间戳的盘上时） |
| 13 | `forceLoginOrgUUID` / `forceRemoteSettingsRefresh` 启动错误在**网络拦截页（HTTP 403）** 时误报"请重新登录" |
| 14 | 崩溃的 Node/Python hook **只显示栈第一行** → 现在显示错误，Ctrl+O 可看 1 万字符 |
| 15 | `/rewind` 各种问题（Esc 取消摘要、列表显示、长提示遮挡警告） |
| 16 | `/diff`、`/status`、`/hooks`、`/mcp`、`/plugin` 等大量 UI 修复 |

## 三、性能改进

- **首条 Bash 命令更快**（shell 启动文件可能在批准前就运行；`CLAUDE_CODE_DISABLE_SHELL_HEAD_START=1` 可关）
- Windows：**PowerShell 每条命令快约半秒**；有/无 Git Bash 的启动都更快
- 输入、流式输出、resize 响应更快（长会话更明显）
- 读/写文件后的响应更快（Windows 最明显）
- **空闲 CPU 更低**（空闲会话唤醒机器更少）
- 20+ 子代理时的 CPU 降低（后台代理 token 每 5 秒更新）
- `/compact` 后 git 仓库里首条消息更快
- 超长会话恢复更可靠（transcript 读到提前结束会重试一次）
- `/update-config` 终端内上下文从 93,000 token 降到约 13,000

## 四、新功能 / 新开关

| 项 | 说明 |
|---|---|
| `CLAUDE_CODE_DISABLE_SHELL_HEAD_START=1` | 关闭"首条 Bash 命令提前启动 shell" |
| `OTEL_RESOURCE_INCLUDE_HOST` | 在 OTel 资源属性里加入 `host.name` / `host.id`（默认关） |
| Code Mode | 现在**可与 agent teams 同时开启**；已在用 Code Mode 的团队会话直接以 Code Mode 启动 |
| `--allowedTools` / `--disallowedTools` / `--tools` | 值不像完整规则时**启动告警** |
| `/status` 新增一行 | 说明**提供商或 `DISABLE_TELEMETRY` 等环境变量关掉了哪些功能** |
| Ctrl+O | 显示**被隐藏的启动提示** |
| `/plugin` | 显示被忽略的畸形 `plugin.json` 字段（如 `author`、`homepage`） |

## 五、其他值得知道的

- 新模型上提交时，Claude 被告知**不要提交可能含机密的文件**
- 沙箱豁免的命令**若因依赖沙箱而仍留在沙箱内，Claude 现在能看到**
- 没有 `rm` 但被旧检查判为"危险 rm"的命令，**改进提示**让 Claude 改用普通语法重发
- **子代理进度摘要**在 Bedrock / Vertex / Foundry 上改用短笔记生成（省大量 token）
- 非交互（`-p`）模式**首个请求前执行的 git 命令更少**
- Remote Control：通过代理时改用 **CONNECT 隧道**（代理拒绝 CONNECT 会报错停止）
- 团队共享预算用尽时**告知重置时间**
- `CERT_SIGNATURE_FAILURE` 连接错误**给出 CA 续期后的可能原因与 IT 可改项**
- 大量终端渲染 / 全屏 / 窄终端 / 主题可读性 / 键位提示 / 对话框一致性改进
- `claude plugin`、`mcp`、`update`、`doctor`、`auth logout` 终端输出**统一成功/失败/警告标记**
- VSCode：新增「Add to Chat」右键菜单；启动更快；多项修复

---

# 第二部分：安装与验证

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.327（原生二进制，355.6 MB） | 官方 CDN 产物，sha256 `343c744b…6ac9` **与官方 manifest 完全一致** |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = 343c744baa14bfd38286a7cb90871051f04379956da6c5b803a9e66019a76ac9
            size   = 372,904,464 bytes
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
sha256sum -c claude-offline-2.1.327-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/）
tar -xzf claude-offline-2.1.327-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

```bash
source ~/.bashrc
claude --version      # 期望：2.1.327 (Claude Code)
```

## 四、常用参数

```bash
bash install-offline.sh --link           # 省 355MB（符号链接，但离线包不能删）
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
- 若用沙箱且想免提示：加 `"sandbox": {"autoAllowBashIfSandboxed": true}`。

> **离线安装 ≠ 离线调用模型**。

## 六、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/versions/2.1.327   # 二进制 355.6 MB
~/.local/share/claude/runtime/           # 随身运行时（5.9 MB）
~/.local/bin/claude                      # 启动器（#!/bin/sh）
~/.claude.json                           # installMethod=native, autoUpdates=false
~/.claude/settings.json                  # 模型端点与 Key（需自行创建）
```

安装后实测占用：**362 MB**（含 5.9 MB 运行时）。

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
5. **本版压缩 bug 修复对你尤其重要**（关遥测场景下之前可能把会话压坏），建议尽快升级。
6. 升级后留意"行为变更"章节：沙箱默认写入范围收窄、环境变量命令行会询问等。

## 九、分批传输

```bash
# 发送端
split -b 90M -d claude-offline-2.1.327-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.327-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.327-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、重建步骤）在归档内 `claude-offline/README.md`。
