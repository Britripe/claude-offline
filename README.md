# 交付说明（先读我）

内网离线安装包：**Claude Code（仅本体）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.327` → `2.1.328`**
> 本包只含 Claude Code 本体；模型配置直接写 `~/.claude/settings.json`（见第五节）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.328-linux-x86_64.tar.gz` | 完整离线包（**168 MB**，解包后 365 MB） |
| `claude-offline-2.1.328-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件（**含完整更新说明**） |

```
sha256 = b516c89341c753ba1314ab70e64682b86591acc167e00c7d1c676e0e8c1c1ef2
```

---

# 第一部分：更新说明（2.1.327 → 2.1.328）

> 二进制只增大 0.1 MB，但内容不少：**约 147 条**，其中好几条正中你的配置。

## ⭐⭐ 与你（网关 + 关遥测 + 关实验性 beta）最相关 —— 5 条

| # | 修复 | 你的场景 |
|---|---|---|
| 1 | **修「设了 `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS`、网关又拒绝 beta 时，auto 模式每轮都失败」**（Bedrock / Agent Platform / Foundry / Claude apps gateway，且不在 Claude Desktop 内） | **直接命中**：之前 auto 模式在你这种配置下会整轮报错 |
| 2 | **修 `CLAUDE_STREAM_IDLE_TIMEOUT_MS` 在网关、自定义 base URL 和云厂商上没能延长「5 分钟连接上限」** | **直接命中**：内网链路慢时不再 5 分钟被掐 |
| 3 | **修「MCP 工具的 input schema 以 `anyOf` / `oneOf` / `allOf` 开头时，在第三方提供商或关闭遥测/特性开关的会话里整个工具消失」** | **直接命中**：你关了非必要流量，之前这类 MCP 工具会不见 |
| 4 | **修「MCP 工具 input schema 非法时使整个请求失败」**（现在只排除该工具并告知 Claude 原因） | 一个坏工具不再拖垮整轮 |
| 5 | **修「关闭遥测/特性开关时，保存定时任务会把项目其他已保存任务抹掉」**（读不到任务文件时） | **数据丢失**，已修 |

## ⚠️ 行为变更

| # | 变更 |
|---|---|
| 1 | **Bedrock / Agent Platform / Foundry / Claude apps gateway 上的 Opus 4.6、Sonnet 4.6 新增 auto 模式**；**未设权限模式的交互会话会直接以 auto 启动** |
| 2 | 第三方提供商上 Opus 4.6 / Sonnet 4.6 的 plan 模式与 Bash 提示改为使用并提供 auto 模式（`disableAutoMode` 可关） |
| 3 | **托管的 `PostToolUse` 钩子不再对「运行前就被拒绝」的调用触发**（不记日志、不改写、不阻断） |
| 4 | 沙箱无法启动时的报告方式变更：**下一条沙箱化 Bash 命令会被拒绝并说明本会话沙箱已关**（除非设了 `sandbox.failIfUnavailable`） |
| 5 | 被 `PermissionRequest` 钩子改写为"沙箱豁免"的命令**将在沙箱外运行** |
| 6 | `CLAUDE_CODE_FORK_SUBAGENT=0` 时子代理默认会等待自己的子代理（可用 `CLAUDE_CODE_PRINT_BG_WAIT_CEILING_MS` 放宽） |
| 7 | 第三方提供商或关遥测时：**粘贴的文本会被标记为"粘贴"**；关闭时带未回答问题结束会**结束该轮**而不是无答继续 |
| 8 | 子代理进度摘要在 Anthropic API 上改由**默认小快模型基于短笔记生成**（关遥测也生效），**token 消耗大幅下降** |

## 一、重要修复

| # | 修复 |
|---|---|
| 1 | **自动压缩复用了会话早先的 system prompt，而不是像 `/compact` 那样重建** —— 已修 |
| 2 | **Linux：分支名不是合法 UTF-8 时，Bash 沙箱每条命令后都重命名 `.git/HEAD`** —— 已修 |
| 3 | **zsh 下 Bash 命令遇到以 `=` 开头的词（如 `echo ====`、`[ a == b ]`）报 not found** —— 已修（`=cmd` 不再展开为路径） |
| 4 | 极长/畸形 markdown 导致渲染卡死 —— 已修 |
| 5 | 偶发屏幕冻结 —— 已修 |
| 6 | 权限提示/提问时紧接着输入导致 **Esc 经常丢失** —— 已修 |
| 7 | 慢连接把粘贴或终端回复截成两半时**内容被敲进提示框** —— 已修 |
| 8 | 恢复大会话时**丢失压缩摘要或最近消息**（消息保存晚时）—— 已修 |
| 9 | spinner 与子代理的 **token 计数把重试的回复算两次**、漏掉未流式回复、思考时不动 —— 已修 |
| 10 | 用量额度在回复中途耗尽时**错误提示成速率限制** —— 已修 |
| 11 | 第三方提供商上部分 Claude Desktop 会话在大量工具结果后**永久 "Prompt is too long"** —— 已修（最大的结果会被排除） |
| 12 | `2.1.327` 引入的回归：Bedrock/Agent Platform/Foundry 拒绝摘要的 effort 设置时，**该子代理模型的 effort 被整会话关掉** —— 已修 |
| 13 | `FORCE_COLOR` 设为负数时启动崩溃 —— 已修 |
| 14 | 机器内存将尽时**机密掩码处理崩溃** —— 已修 |
| 15 | Linux：装 `xclip`/`wl-copy` 时，Ctrl+Z 挂起期间**在其他应用粘贴会冻结** —— 已修 |
| 16 | Linux：Ctrl+V **拿不到剪贴板图片 / 粘到旧图 / 卡住** —— 已修 |
| 17 | `/terminal-setup` 在 Zed 里会把符号链接的 `keymap.json` 替换成普通文件 —— 已修 |

## 二、Code Mode（实验性）大量修复

约 15 条，主要有：权限提示/计划审批/提问停留 10 分钟不再中断脚本；auto/bypass 模式下新连上的 MCP 服务器可用；从 `--resume` 选择器恢复时可在后台运行；恢复会话时不再丢失已加载技能/hook 上下文/审批时输入的备注；不再对未定义变量静默用 `undefined`；不再拒绝以 `{` 开头的合法脚本；不再拒绝同时用顶层 `await` 与 `return` 的脚本。

性能：auto/bypass 模式下**去掉脚本后 0.75 秒停顿**；有 MCP 的会话提示缓存更好；工具描述更短省 token；**运行时显示脚本正在执行的命令与输出**。

## 三、新功能 / 新开关

| 项 | 说明 |
|---|---|
| `CLAUDE_CODE_DISABLE_DEFERRED_TOOL_RESUME` | 让宿主阻止 `claude -p --resume` / `--continue` 运行被钩子延后的工具调用 |
| `mcp_tool` 钩子支持 `"onFailure": "block"` | 钩子调用失败即阻断该动作（与命令/HTTP 钩子一致） |
| `/status` 与 `claude doctor` 新增提示 | OpenTelemetry 导出器没有 endpoint、数据可能丢失时告警 |
| 跨会话消息提示 | `claude -p` 文本输出会在 stderr 说明消息被扣留/丢弃的原因及如何放行 |
| 网关启动告警 | 策略的 `maxEffortLevel` 不是合法 effort 档位时告警 |
| 嵌入方支持 `allowManagedModsOnly` | 管理员托管设置仍优先 |

## 四、其他变更

- `/code-review` 在多数模型上**只报 bug 与 CLAUDE.md 违规**（复用/简化请用 `/simplify`）；每条结论标记 **Important / Pre-existing / Nit**，摘要末尾给建议修复
- `/bug` 改为**先问 Yes/No 再发送**（连按回车不再误发）
- `/list-agents` 只给你自己看列表
- 恢复大会话更快；图片多的会话恢复时内存更低
- 长时间 SDK/桌面会话（跑慢命令/工具）内存与响应改善
- 同时启动多个 in-process teammate 时 "Lock file is already being held" 大幅减少
- `claude -p` / SDK 会话会**后台预取**带自己 git 仓库的托管设置插件（首次运行可能晚 ≤30 秒退出）
- 计算型审批对话框会显示每个应用的访问级别
- 大量 Remote Control / 云会话 / VS Code / 自托管 runner 修复（与本包无关）

---

# 第二部分：安装与验证

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.328（原生二进制，355.7 MB） | 官方 CDN 产物，sha256 `a982d380…65e8` **与官方 manifest 完全一致** |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = a982d38030b97e750848add2cbb2df6d9a70d5544a22ca9b216a349ba25865e8
            size   = 373,019,152 bytes
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
sha256sum -c claude-offline-2.1.328-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/）
tar -xzf claude-offline-2.1.328-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

```bash
source ~/.bashrc
claude --version      # 期望：2.1.328 (Claude Code)
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
- 内网链路慢、常撞 5 分钟上限：可设 `"CLAUDE_STREAM_IDLE_TIMEOUT_MS": "600000"`（本版起在网关上真正生效）。

> **离线安装 ≠ 离线调用模型**。

## 六、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/versions/2.1.328   # 二进制 355.7 MB
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
5. **本版修了 3 条直接命中你配置的问题**（beta 拒绝时的 auto 失败、5 分钟连接上限、anyOf 类 MCP 工具消失），建议尽快升级。
6. 升级后留意：**未设权限模式的交互会话现在会以 auto 模式启动**（可用 `disableAutoMode` 关闭）。

## 九、分批传输

```bash
# 发送端
split -b 90M -d claude-offline-2.1.328-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.328-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.328-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、重建步骤）在归档内 `claude-offline/README.md`。
