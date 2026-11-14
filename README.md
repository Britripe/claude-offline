# 交付说明（先读我）

内网离线安装包：**Claude Code（仅本体）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.323` → `2.1.325`**
> （中间 2.1.324 无独立 changelog，属短命发布）
> 本包只含 Claude Code 本体；模型配置直接写 `~/.claude/settings.json`（见第五节）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.325-linux-x86_64.tar.gz` | 完整离线包（**165 MB**，解包后 359 MB） |
| `claude-offline-2.1.325-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件（**含完整更新说明**） |

```
sha256 = c9218ef269599cfad944077549e958e0356a1f5c897700ac96455a0204541003
```

---

# 第一部分：更新说明（2.1.323 → 2.1.325）

## ⚠️ 最重要：一条会影响无人值守流程的行为变更

> **沙箱化的 Bash 命令默认改为走正常权限流程**（也就是会询问）。
>
> | 想要 | 做法 |
> |---|---|
> | 保持"沙箱内免提示直接跑" | `sandbox.autoAllowBashIfSandboxed: true` |
> | 用托管设置开沙箱 | 必须把该项写进**最高优先级的托管来源** |

**相关连带变更（都会增加"被询问"的机会）：**

1. 只读 git 命令在**不属于本项目/未信任的仓库**里改为先询问（用 `--add-dir` 点名可免提示）
2. 定时任务与 `/loop` 在运行声明了 `allowed-tools` 或 hooks 的**技能**前会先询问（无人值守就把 `Skill(name)` 加进 `permissions.allow`）
3. Desktop / IDE / SDK 的「Always allow」只在"保存后确实能免提示"时才单独提供规则

## 一、安全修复

| # | 修复 |
|---|---|
| 1 | **zstd 压缩响应的解码方式存在安全问题** —— 已修（changelog 第一条） |
| 2 | **Bash 权限检查存在缺口：部分复合命令可未经询问直接运行** → 现在会询问 |
| 3 | **Bash allow 规则可能运行与规则同名的另一个程序** → 已修 |
| 4 | **部分 deny / ask 规则未作用于某些 Bash 命令** → 已修 |
| 5 | 沙箱命令**能写入原生更新器的下载目录** → 已修 |
| 6 | 沙箱**未完全保护 worktree 的设置与 hooks** → 已修 |
| 7 | 沙箱**未完全保护 linked worktree 的共享 git 目录** → 已修 |
| 8 | 沙箱现在**同样拒绝写入 added 目录里的 `.claude`**（设置/技能/agents/commands/hooks） |
| 9 | Claude Code 自己启动的沙箱辅助程序（网络桥）**不再携带登录令牌/API Key** |
| 10 | ripgrep、本地 git、沙箱运行时助手**不再携带 mTLS 客户端私钥口令** |
| 11 | 账号邮箱检查**扩展到脚本、文件写入、WebFetch 与 MCP 调用** |
| 12 | 提交进 git 的 `.claude/settings.local.json` 在**某些子目录启动时被误当作你自己的设置** → 已修 |
| 13 | 相对路径 deny/ask 规则（如 `Read(./.env)`）**在 `cd` 之后失效** → 现在也作用于 added 目录 |
| 14 | `mcp__plugin_*_` 开头的 deny/ask 规则**大小写不一致时漏判** → 已修 |
| 15 | 插件加载会**跳过不安全的路径与符号链接**，`plugin validate` 也会报告 |

## 二、与你网关环境最相关的 3 条 ⭐

| # | 变更 | 为什么重要 |
|---|---|---|
| 1 | **修「很多第三方网关与 Foundry 上每轮都报 context length 错误而不是压缩」** | **直接对应你的自建网关** —— 之前会卡死在上下文超限 |
| 2 | **`CLAUDE_CODE_MAX_CONTEXT_TOKENS` 现在也作用于以 `claude-` 开头的未识别模型 ID**（如网关的 `claude-sonnet-latest`） | 承接上版"未识别模型按 1M 估算"，现在可精确控制 |
| 3 | 修子代理按**主会话模型**计算文件读取与 MCP 结果大小（子代理模型不同时算错） | 子代理上下文预算更准 |

## 三、Linux / 家目录 / 文件系统

| # | 修复 |
|---|---|
| 1 | **Linux：工作目录位于 `denyRead` 内的 `allowRead` 文件夹下时（例如家目录是符号链接），沙箱化 Bash 报 `bwrap: Can't create file`** → 已修 |
| 2 | 工作目录被删除后 shell 命令仍在**启动目录**执行 → 现在报错，可重新发起 |
| 3 | 没有 `HOME` 环境变量时 **AWS `credential_process` 与 GCP 默认凭据**失败 → 已修 |
| 4 | 修 `.oauth_refresh.lock.owner.tmp.*` 残留文件堆积 |
| 5 | 修 `claude auth status` 等短命令在 Console 配置上**遗留锁**，导致下个会话卡约 1 分钟 |

## 四、Windows

- 修 PATH 某条目位于**账号无法打开的文件夹**下时的启动停顿（约 1 秒）
- 修 `/resume` 在**盘符大小写或斜杠写法不同**时拒绝/隐藏会话
- 修会话内 `/resume` 因盘符大小写不同**丢掉项目的已保存信任**

## 五、headless / SDK / 认证

- 修 `-p` 在**终端**输出时放行非文本样式的控制序列（管道与文件输出不变）
- 修 `-p` 在没有缓存账号数据时**先等数据**再发首个请求（干净环境首次启动可能慢几秒）
- 修 Agent SDK `applyFlagSettings()` 里**一个非法值会静默丢弃之前所有设置** → 现在直接报错拒绝
- `claude remote-control` 退出码改进：自行停止 **4**、无法注册 **69**（便于服务管理器重启）
- 修背景会话**误用并计费**启动它的 shell 里的 API Key（本应走 claude.ai 登录）
- 更多"后台会话切换到别的登录"的情况现在**被拒绝并说明**

## 六、MCP

- 修 MCP 服务器在**一次被暂时拒绝的令牌刷新**后一直显示"需要认证"，且 `mcp_tool` hooks 被跳过
- 修 SDK 宿主在工具调用重连 MCP 后返回 "Not connected"
- 修**后台 MCP 任务**在其它调用导致连接丢失后停止更新
- 修后台 MCP 任务的问题框**打断正在输入的消息**（改为等你发送或清空草稿）
- 修 `/mcp` 在插件重载后仍列出已被移除的 MCP 服务器
- 修 `worktree.sparsePaths` 与稀疏插件克隆在**老 git（如 2.34）**上漏掉仓库根文件

## 七、其他变更要点

- 组织策略阻止请求时，错误下方新增一行提示：用 `/rewind` 改消息或用 `/clear`
- 沙箱化的 `git checkout` / `git stash` / autostash pull 无法更新 `.claude` 等受保护文件时会告知 Claude
- `/cost`、`$.session.usage()`、`--max-budget-usd` 现在也统计 mod 的 `$.model.complete` / `$.model.classify` 调用
- 插件市场下载在**只设了 `ALL_PROXY`** 时改用 CONNECT 隧道（拒绝 CONNECT 的代理会失败）
- `-p`/SDK 在挂进容器的 worktree / submodule 里，`.claude/settings.local.json` 的 allow 规则**要等目录被信任后**才生效
- `/login` 现在会**先征求同意**再改动隐私设置
- 自托管 runner 拒绝会话目录或仓库路径是**符号链接**的会话（除 `checkout` hook 的检出）
- 新增：Claude 读过/改过 Jupyter notebook 后，VS Code 等终端会显示一条"重跑单元格"提示
- 大量 VS Code / Claude Code on the web / Claude Tag 修复（与本包无关）

---

# 第二部分：安装与验证

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.325（原生二进制，348.9 MB） | 官方 CDN 产物，sha256 `0dc4f4a9…b0cf` **与官方 manifest 完全一致** |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = 0dc4f4a9a165fc60746d7ae8489712e5fc352f1e7ad4eb22e7141bd0812eb0cf
            size   = 365,842,960 bytes
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
sha256sum -c claude-offline-2.1.325-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/）
tar -xzf claude-offline-2.1.325-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

```bash
source ~/.bashrc
claude --version      # 期望：2.1.325 (Claude Code)
```

## 四、常用参数

```bash
bash install-offline.sh --link           # 省 348MB（符号链接，但离线包不能删）
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
- 自定义模型名默认按 1M 上下文估算；想回 200K 或只想调窗口，加
  `"CLAUDE_CODE_DISABLE_1M_CONTEXT": "1"` 或 `"CLAUDE_CODE_MAX_CONTEXT_TOKENS": "<数值>"`。
- 若用沙箱且想免提示：加 `"sandbox": {"autoAllowBashIfSandboxed": true}`。

> **离线安装 ≠ 离线调用模型**。

## 六、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/versions/2.1.325   # 二进制 348.9 MB
~/.local/share/claude/runtime/           # 随身运行时（5.9 MB）
~/.local/bin/claude                      # 启动器（#!/bin/sh）
~/.claude.json                           # installMethod=native, autoUpdates=false
~/.claude/settings.json                  # 模型端点与 Key（需自行创建）
```

安装后实测占用：**355 MB**（含 5.9 MB 运行时）。

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
5. **本版起沙箱 Bash 默认会询问** —— 无人值守脚本请先设 `sandbox.autoAllowBashIfSandboxed: true`。
6. 本版修了 zstd 解码安全问题与多项沙箱保护缺口，建议尽快升级。

## 九、分批传输

```bash
# 发送端
split -b 90M -d claude-offline-2.1.325-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.325-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.325-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、重建步骤）在归档内 `claude-offline/README.md`。
