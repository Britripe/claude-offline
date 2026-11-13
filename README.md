# 交付说明（先读我）

内网离线安装包：**Claude Code（仅本体）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.322` → `2.1.323`**（**大版本**：新增 15+ 条、修复 100+ 条、变更 50+ 条）
> 本包只含 Claude Code 本体；模型配置直接写 `~/.claude/settings.json`（见第五节）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.323-linux-x86_64.tar.gz` | 完整离线包（**164 MB**，解包后 355 MB） |
| `claude-offline-2.1.323-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件（**含完整更新说明**） |

```
sha256 = 43d491b296472d83aa8f0e0478b6cbcc73d36ea4ec3cf8193d04b1a21d1e8c8b
```

---

# 第一部分：更新说明（2.1.322 → 2.1.323）

## ⚠️ 对你影响最大的 3 条（务必先看）

| # | 变更 | 你要做什么 |
|---|---|---|
| 1 | **不认识的模型 ID 的上下文窗口假设从 200K 改为 1M** | 你网关上的自定义模型名会被按 **1M** 估算 → 自动压缩触发更晚。想保持 200K：`CLAUDE_CODE_DISABLE_1M_CONTEXT=1`；想早点压缩：`/autocompact` |
| 2 | **其他厂商模型窗口较小时，填满后可能每轮失败直到 `/clear`** | 若网关后面是窗口较小的模型，**务必在填满前**设 `CLAUDE_CODE_DISABLE_1M_CONTEXT=1` |
| 3 | **组织设置与策略请求不再使用仅由 `ALL_PROXY` 设置的代理** | 内网若只靠 `ALL_PROXY` 走代理，需改设 **`HTTPS_PROXY`** |

## 一、安全修复：17 项沙箱/权限绕过（建议尽快升级）

| # | 修复的问题 |
|---|---|
| 1 | Bash 沙箱可能在**会话中途被关闭** |
| 2 | `sandbox.excludedCommands` 会让**未点名的命令跑到沙箱外** |
| 3 | **Write / Edit 可写入被 `Read` deny 规则覆盖的路径** |
| 4 | `rm` 因**判断目录错误**而跳过"危险删除"提示（bypass 或 allow 规则下） |
| 5 | `Bash(ls*)` 这类**带通配的 allow 规则**会跳过提示；`Bash(NODE_ENV*)` 这类含变量规则改为要授权 |
| 6 | deny / ask 规则**漏掉经 `sudo`、`env` 等包装器执行的命令** |
| 7 | **沙箱内命令能读到 Claude Code 的 file checkpoint 副本** |
| 8 | 旧配置文件的安装下，沙箱**未完全保护 Claude Code 自身配置** |
| 9 | **zsh 某些非默认 shell 选项**下命令不弹提示 |
| 10 | 部分 **`lsof` 命令**不弹权限提示 |
| 11 | Bash 权限检查**漏判命令的一部分** |
| 12 | **项目设置经符号链接**被读到另一个目录 |
| 13 | 沙箱现在**忽略"可写目录里的符号链接"**指向的外部写权限（需授权真实路径） |
| 14 | **`git worktree add/move/remove`** 在沙箱里改为只读失败 |
| 15 | **`--permission-mode auto`** 在 auto 不可用时会退到更宽松模式 → 现在退到 **Manual**（先问） |
| 16 | **`-p`/SDK** 在 `--settings` 声明开沙箱但校验失败时曾**不沙箱运行** → 现在启动即停并指出配置项 |
| 17 | Claude Code 自己的 **ripgrep 与多数内部 git 命令不再携带登录令牌/API Key** |

**另外两条策略收紧**（会改变实际行为）：

- 「don't ask again」若规则看起来含密钥、或会被写入 **git 跟踪的** `settings.local.json`，则**只在本次会话有效**
- Bash 沙箱的写入权限不再认可"可写目录里的符号链接"（要授权真实路径）

## 二、内网 / NFS / 文件系统

| 项 | 说明 |
|---|---|
| **autofs 家目录** | Linux 上 `/home` 是 autofs map 时启动更快（不再在 `/home` 里查文件）；map 根更深时设 `CLAUDE_CODE_AUTOMOUNT_ROOTS` |
| **NFS/FUSE 丢消息** | 修"丢弃回复时可能丢失已保存会话的一部分" |
| **`/rewind` 网络路径** | 修网络路径与挂死目录下 `/rewind` 把所有被跟踪文件都列为已修改、标签慢、读取不返回时列表为空 |
| **`/proc`、设备、命名管道** | 修 Edit/Bash 在部分 `/proc` 文件上 OOM；修 Edit 在设备与命名管道上挂起/OOM |
| **超大文件** | 修 Claude 改完超大文件后下一次提示的**内存尖峰** |
| **临时文件** | 修会话改过目录后、按 `cleanupPeriodDays` 过期时临时文件残留 |

## 三、headless / `-p` / SDK

- **新增 30 秒进度心跳**：hooks 长时间运行时 `-p`/SDK 定期上报，宿主不再判定"卡死"而结束回合
- **`-p` 被设置阻止指定模型却静默换模型** → 现在向 stderr 告警
- **`-p` 每次运行都打印 `deniedModels`/`availableModels` 提示** → 现在只在 `/status`、`claude doctor` 里列
- 新增 **`CLAUDE_CODE_DISABLE_AGENT_PROGRESS_SUMMARIES`**（2.1.318 起可用）：关掉子代理进度摘要及其请求
- 新增 **`CLAUDE_CODE_RESUME_HINT_COMMAND`**：让启动方自定义退出提示里的 resume 命令
- 修 text 输出模式下 `--plugin-dir` 的 mod hook 挂掉时 **stderr 什么都不打**
- 修 SDK 宿主在 MCP 工具列表里塞 null 时 headless 会话直接结束
- 修 `--no-session-persistence` 下长工具结果留在磁盘（现在正常退出/Ctrl+C/SIGTERM 都会清理）

## 四、网关 / 认证

| 项 | 说明 |
|---|---|
| **auto 模式 + 网关模型名含 "haiku"** | 除 `claude-haiku-5-5` 外，这类模型现在**被拒绝**（与 Haiku 4.5 同等对待） |
| **网关登录过期后会话反复失败** | 你在另一终端 `/login` 成功后，这些会话下一条消息即可继续 |
| **`policyHelper`** | 不再从用户/`--settings` 文件取环境变量；要用就写进托管 `env` 或启动 shell |
| **JSON 日志双重转义** | 修网关 JSON 日志把换行与控制字符转义两次 |
| **`/v1/models`** | 对每个上游窗口都确定的模型，现在给出 `max_input_tokens` |
| **凭据隔离** | 背景会话切换登录/配置文件时不再残留上一账号凭据；被拒绝的登录不会留下任何登录 |

## 五、新功能

| 项 | 说明 |
|---|---|
| **Code Mode（实验性，需手动开启）** | Claude 用**写 JavaScript 调用工具**的新模式。见 https://code.claude.com/docs/en/code-mode ，可按组织关闭 |
| **插件版本约束** | `plugin.json` 新增 `engines["claude-code"]`：超出范围的插件不安装/不加载，安装时选最合适版本 |
| **`claude plugin validate`** | 新增 TypeScript hooks 模块类型检查（`--strict` 时失败）；新增 mod `engine.create` 的 `$` 名词校验 |
| **Artifact 上传** | 支持 `.docx` |
| **启动告警** | 变量值里带 `*` 的 Bash allow 规则（如 `Bash(PORT=* npm run dev)`）启动时告警；插件语言服务器起不来也会告警并指名 |
| **`/diff` 关闭提示** | diff 面板新增 `/diff to close` 提示，`?` 快捷指南加入聚焦/关闭面板按键 |
| **用量选项菜单** | 模型需要用量额度但额度关闭/用尽时，给出（停止 / 切到用量额度 / 升级）菜单 |

## 六、其他重要变更

- **同机会话间互发消息上限从约 20 改为 100,000**（Desktop Code tab 同理）
- **`CLAUDE_CODE_OVERLOADED_RETRY_MAX_DELAY_MS` 上限 64s → 300s**
- **移除 `subagentStatusLine` 设置**（子代理行统一默认外观；旧配置文件仍可正常加载）
- **弃用插件的 `agent` 设置**（仍生效，但启动时、`plugin list`、`plugin validate` 都会告警）
- `MCP_SDK_GENERATION=v1` 改为在 v2 客户端上跑兼容模式（v1 客户端已移除）
- `/security-review` 改为运行前先询问；技能内联 git 命令在 bare 仓库等目录默认停止等待授权
- 项目 `.mcp.json` 的 MCP 服务器审批改为**精确匹配服务器名**（换个写法写会重新询问）
- `--trust-workspace false` 与 VS Code 未信任目录下，仓库设置**不能再关掉你的 hooks/开关插件**
- 修 `/compact` 再次运行或紧接 `/context` 后会把 Claude 尚未回复的消息压缩掉
- 修设置文件在你手改后又被运行中会话覆盖；会话中删除的设置文件现在保持删除
- 修 `claude plugin` 与 `claude mcp list` 在令牌续期结束时把你从 Console 配置登出
- 修 MCP 工具调用以 `TOKEN_EXPIRED` 等登录服务器错误结束（而非提示重新登录）
- 修 Bedrock 拒绝账号某模型的错误提示（不再说"刷新 AWS 凭据"，改为"启用该模型"）
- 修 TLS 检查代理自身用 `https://` 时证书错误持续存在
- 修证书 name-constraint 错误被重试数分钟（现在立即失败并说明怎么修）
- 大量 Remote Control / 云会话 / VS Code / Claude Tag / Code Review 修复（与本包无关）

---

# 第二部分：安装与验证

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.323（原生二进制，345.6 MB） | 官方 CDN 产物，sha256 `604d29a6…348e` **与官方 manifest 完全一致** |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = 604d29a6869409106b6a1110bdadcc8b72e8654a5a865d23258f938a696e348e
            size   = 362,435,088 bytes
```

> **下载方式**：国内镜像获取（第 3 次尝试成功，首次遇 DNS 解析失败），
> 再**按官方 manifest 逐字节校验通过**，并随包附带官方 `manifest.json`（2,101 B）。

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
sha256sum -c claude-offline-2.1.323-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/）
tar -xzf claude-offline-2.1.323-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

```bash
source ~/.bashrc
claude --version      # 期望：2.1.323 (Claude Code)
```

## 四、常用参数

```bash
bash install-offline.sh --link           # 省 345MB（符号链接，但离线包不能删）
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
- **本版起**：自定义模型名默认按 1M 上下文估算，想回 200K 就在 `env` 里加
  `"CLAUDE_CODE_DISABLE_1M_CONTEXT": "1"`。

> **离线安装 ≠ 离线调用模型**。

## 六、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/versions/2.1.323   # 二进制 345.6 MB
~/.local/share/claude/runtime/           # 随身运行时（5.9 MB）
~/.local/bin/claude                      # 启动器（#!/bin/sh）
~/.claude.json                           # installMethod=native, autoUpdates=false
~/.claude/settings.json                  # 模型端点与 Key（需自行创建）
```

安装后实测占用：**352 MB**（含 5.9 MB 运行时）。

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
5. **本版修了 17 项沙箱/权限绕过**，加上上下文窗口假设变化 —— 建议内网机器尽快升级并核对第一节。

## 九、分批传输

```bash
# 发送端
split -b 90M -d claude-offline-2.1.323-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.323-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.323-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、重建步骤）在归档内 `claude-offline/README.md`。
