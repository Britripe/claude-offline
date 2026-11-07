# 交付说明（先读我）

内网离线安装包：**Claude Code（仅本体）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.318` → `2.1.319`**
> 本包只含 Claude Code 本体；模型配置直接写 `~/.claude/settings.json`（见第五节）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.319-linux-x86_64.tar.gz` | 完整离线包（**155 MB**，解包后 334 MB） |
| `claude-offline-2.1.319-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件 |

```
sha256 = 1ad3077860a04085933c9d2e9bef21d727d7163e99430991ffa10dcbd4c8c343
```

---

## ⚠️ 本版有一个会影响你的行为变更（请先看这条）

**auto mode 在「关闭遥测 / 关闭特性开关 / 经 LLM 网关」时行为变了：**

> 服务端**无法检查**的工具调用，现在会**转为询问或直接拒绝**，
> **不再回退到内置分类器**。

- 想保持旧行为：`export CLAUDE_CODE_AUTO_MODE_SERVER=0`
- 反过来，`CLAUDE_CODE_AUTO_MODE_SERVER=1` 现在**即使设了 `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` 也能生效**；
  若托管设置里设了后者，必须同时设 `1`
- **Bedrock / Vertex / Foundry 不受影响**

> 你的包默认写了 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1`，属于「遥测关闭」情形 ——
> 如果你在用 auto 模式，升级后可能会感觉"更多调用被询问/拒绝"。按上面两个变量二选一即可。

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.319（原生二进制，324.5 MB） | 官方 CDN 产物，sha256 `dabafec8…8eed` **与官方 manifest 完全一致** |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = dabafec82ca1319fa43d8be36a10fc61b1d11821cb98ad5df04ad95e199b8eed
            size   = 340,226,576 bytes
```

> **下载方式**：国内镜像高速获取（15.6 MB/s，约 10 秒），再**按官方 manifest 逐字节校验通过**，
> 并随包附带官方 `manifest.json`（2,101 B）。

## 二、本版更新要点（2.1.319）

### 与你（内网 + 网关 + 关遥测 + 无人值守）最相关 —— 6 条

| # | 变更 | 影响 |
|---|---|---|
| 1 | **auto mode 行为变更**（见上方红框） | 关遥测/网关场景下不再回退内置分类器；用 `CLAUDE_CODE_AUTO_MODE_SERVER=0` 可回旧行为 |
| 2 | **新增 `CLAUDE_CODE_DISABLE_FEATURE_FLAGS`** | **不关闭遥测的前提下**停止拉取特性开关，全部用内置默认值 —— 内网/断网环境友好 |
| 3 | **新增 `permissions.denyInsteadOfPrompting`** | 本该弹权限提示的调用**直接拒绝并继续本轮**（终端、子代理、teammates 都适用）—— 无人值守/CI 场景很好用 |
| 4 | **修 MCP 老握手服务连不上 / 返回空结果** | 原因是 Claude Code 声明了只有新服务才用的扩展；**内网老版本 MCP 服务**受益 |
| 5 | **修 401 重试时反复刷新登录、连带把其他会话也搞成 401** | 多会话共用凭据时的稳定性 |
| 6 | **修开头带空格 / 制表符 / 换行的斜杠命令被当普通文本发出去** | 从文档复制命令时常见 |

### 其他值得知道的

**MCP / 工具**
- 终端新增 **MCP 任务**支持：服务把工具调用当任务跑时，Claude Code 立即移到后台，完成后再告知
- 新增 `CLAUDE_CODE_DISABLE_MCP_TASK_BACKGROUND`：改为在工具调用内**等待**而非转后台
- 修 `--resume` 后 claude.ai connector 上的长时 MCP 调用有时永不回报结果

**会话 / 界面**
- 修 `/diff` 在仓库顶层存在名为 `HEAD` 的文件/目录时不可用
- 修 `/diff` 面板在 ≥64 轮编辑过文件的会话里空白
- `/tasks` 新增「Earlier in this session」行：可重新打开已完成的子代理并继续对话
- `/loop` 在云会话里的确认框新增提示（定时器随容器销毁，可改用可存活循环）

**登录 / 计费**
- 新增警告：claude.ai 登录到期后改用 `ant auth login` 的登录会**按 Claude Console 组织计费**
- 启动时新增一行，说明当前计费的是哪个 Claude Console 组织

**网关（Claude apps gateway）**
- `admin.pooled_conflict_rule` / `admin.pooled_priority_order`：多个分组预算时谁先付费
- 可选 `login_notice`：在网关浏览器验证页与 Claude Code 登录页显示运营方自定义提示
- `CLAUDE_GATEWAY_LOG_FORMAT=json`：运维日志输出为 JSON（`severity`/`time`/`message`）
- 启动警告：某策略的 `availableModels` 也放行了另一策略按名字列出的自定义模型
- 启动警告：无 `desktopHome` 却依赖 `coworkTabEnabled:false` 或 `chatSessionRetentionDays`（11-10 变更前）
- `/usage` 为无花费上限的网关开发者显示今日/本周/本月花费

**其他**
- `/claude-api latency-optimize`：找出 Claude API 代码里让人等待的地方，并用你的 eval 试更快设置
- 云会话 / Code Review / Claude Tag 若干修复（与本包无关）

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
sha256sum -c claude-offline-2.1.319-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/）
tar -xzf claude-offline-2.1.319-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

```bash
source ~/.bashrc
claude --version      # 期望：2.1.319 (Claude Code)
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
- 想把特性开关彻底关掉（不关遥测）：加 `"CLAUDE_CODE_DISABLE_FEATURE_FLAGS": "1"` 到 `env`。

> **离线安装 ≠ 离线调用模型**。

## 六、常用参数

```bash
bash install-offline.sh --link           # 省 324MB（符号链接，但离线包不能删）
bash install-offline.sh --system-glibc   # 用目标机 glibc（需 ≥2.26）
bash install-offline.sh --force          # 覆盖已安装（会先备份）
bash install-offline.sh --no-path        # 不改 shell rc
```

## 七、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/versions/2.1.319   # 二进制 324.5 MB
~/.local/share/claude/runtime/           # 随身运行时（5.9 MB）
~/.local/bin/claude                      # 启动器（#!/bin/sh）
~/.claude.json                           # installMethod=native, autoUpdates=false
~/.claude/settings.json                  # 模型端点与 Key（需自行创建）
```

安装后实测占用：**331 MB**（含 5.9 MB 运行时）。

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
5. **若升级后发现 auto 模式变严**，按第一节说明设 `CLAUDE_CODE_AUTO_MODE_SERVER=0`。

## 十、分批传输

```bash
# 发送端
split -b 90M -d claude-offline-2.1.319-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.319-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.319-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、重建步骤）在归档内 `claude-offline/README.md`。
