# 交付说明（先读我）

内网离线安装包：**Claude Code（仅本体）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.330` → `2.1.332`**（含 2.1.331 + 2.1.332 两版，**共 285 条改动**）
> 本包只含 Claude Code 本体；模型配置直接写 `~/.claude/settings.json`（见第五节）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.332-linux-x86_64.tar.gz` | 完整离线包（**181 MB**，解包后 393 MB） |
| `claude-offline-2.1.332-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件（**含完整更新说明**） |

```
sha256 = 71a82d90b6cf8361328a2251351b02afd6114a1666d8bed97d0ce57194444dda
```

---

# 第一部分：更新说明（2.1.330 → 2.1.332）

## 改动规模

| 指标 | 数值 |
|---|---|
| 二进制 | 381,121,040 → **402,325,736 B**（**+20.2 MB / +5.3%**） |
| changelog | **291 行 / 285 条** |
| 分类 | Fixed **99** / Changed **26** / Improved **22** / Added **16** / 其他 **122** |

## ⭐⭐⭐ 与你（网关 + 关遥测 + 内网）最相关的 8 条

| # | 变更 | 说明 |
|---|---|---|
| 1 | **新增 `CLAUDE_CODE_DISABLE_RESPONSE_COMPRESSION`** | 让模型响应不压缩，专为"代理会截断压缩响应"的场景（官方暂不建议用于自托管 runner / Remote Control） |
| 2 | **改进「代理/网关有请求体上限，导致每条消息和 `/compact` 都 413」的恢复** | 长会话不再全废 |
| 3 | **改进压缩响应被代理切坏时的报错** | 不再抛原始 `ZlibError fetching`，改为平实说明 |
| 4 | **`claude doctor` 新增「API 连通性」小节** | 报告 API 是否可达 + HTTP 状态或具体错误 —— 内网排障利器 |
| 5 | **修「重复自动压缩失败后整轮报 `Prompt is too long`」** | 现在会在上下文上限再压一次 |
| 6 | **修「摘要引用某些 XML 标签时 `/compact` 与自动压缩丢部分摘要」** | 压缩正确性 |
| 7 | **修「Claude 思考中回复静默导致整轮/子代理超时失败」** | 现在会再试一次 |
| 8 | **修「`opus` 在网关用自己的话拒绝模型时失败」** | 现在降级到该家族旧模型 |

## ⚠️ 模型别名与默认模型变更（会影响 `ANTHROPIC_MODEL`）

| 平台 | 变更 |
|---|---|
| **Google Agent Platform / Foundry** | `sonnet` → **Sonnet 5.5**；Foundry 的 `opus` → **Opus 5.5** |
| **Microsoft Foundry** | **默认模型改为 Sonnet 5.5**（需部署 `claude-sonnet-5-5`，或设 `ANTHROPIC_DEFAULT_SONNET_MODEL`） |
| **Amazon Bedrock** | `sonnet` **仍为 Sonnet 4.5** |
| **Bedrock Mantle** | `sonnet` 现在指 Sonnet 5（非 5.5）；账号无 Sonnet 5 会报错，需先启用 |
| **Foundry 退役** | Sonnet 4 / Opus 4 / Opus 4.1 计为**已退役** |
| 通用 | `--model default` 与 Default 选择在模型不可用时**降级到旧模型**而非报错 |
| 通用 | 被移到 Sonnet 5/5.5 的 Bedrock Mantle / Agent Platform / Foundry 会话，**未设权限模式时以 auto 启动**（`disableAutoMode` 可关） |

## 一、稳定性大修（很多冻结/内存问题）

- 修「权限规则或钩子条件含多个通配符、匹配长命令时冻结」
- 修「钩子无休止输出导致冻结与内存失控」
- 修「钩子阻断/失败时日志极大 → 冻结 + 上下文溢出」（超约 5 万字符会截断）
- 修「本地 MCP 服务器刷屏 stdout 时终端冻结、Ctrl+C 失灵」
- 修「提示或管道输入含 `@` + 超长不间断字符串（如内嵌字体的压缩 CSS）时冻结」
- 修「`Read`/`@` 提及/压缩在无人写入的命名管道上挂住（如被锁的 secrets manager 挂载的 `.env`）」→ 20 秒失败
- 修「SSE MCP 服务器事件流被切断后所有工具调用挂住，需手动重连」
- 修「MCP 工具调用在畸形回复上挂 30 分钟」
- 修「`~/.claude.json` 在另一会话正在保存时被误报为损坏」
- 修「`claude auth login` 保存失败却报 `Login successful.`」→ 现在退出码 1 并说明原因
- 修「恢复会话/`/stats` 在 transcript 有大量损坏行时冻结数秒」
- 修「`/stats` 读大会话时按键延迟数秒」
- 修「一个权限提示或重新读大文件后长时间冻结」

## 二、新功能 / 新开关

| 项 | 说明 |
|---|---|
| **MCP 服务器提供的技能（beta）** | 在 `/` 菜单显示为 `server:skill` |
| **`PreSkillShellCommand` 钩子事件** | 可在技能/命令加载时拒绝其内联 shell 命令 |
| **`CLAUDE_CODE_DISABLE_RESPONSE_COMPRESSION`** | 见上 |
| **`CLAUDE_GATEWAY_IDP_CONNECT_BY_HOSTNAME=1`** | 网关 + OIDC + 代理场景 |
| `claude doctor` 的 API 连通性 | 见上 |
| 插件 `optionalDependencies` | 随主插件安装、可单独禁用/卸载 |
| Code Mode 支持 Bedrock / Agent Platform / Foundry / Claude Platform on AWS | 扩展了可用范围 |
| OTel：`gen_ai.usage.*`、`gen_ai.response.model`、`gen_ai.operation.name` | 加入 `claude_code.llm_request` span |

## 三、其他重要修复

- 修「空闲压缩对远低于周额度上限的订阅者被跳过」
- 修「`--effort` 在 `--environment` 启动且 `CLAUDE_CODE_EFFORT_LEVEL` 同值时被忽略」
- 修「`@file#10-12` 附了整个文件而不是那几行」
- 修「粘贴的链接 / 以图片文件名结尾的相对路径出现较晚，吞掉紧随其后的回车」
- 修「自定义 API 端点返回无名工具调用时崩溃」→ 现在该调用失败、会话继续（**网关相关**）
- 修「web 搜索在允许/阻止域名列表为空或有重复时 API 报错」
- 修「`temp_*` 目录在插件缓存里堆积」
- 修「MCP 任务 ID 超过 128 字符报 `malformed task handle`」→ 支持到 1024
- Windows：修「部分企业 PC 上启动后无输出直接退出」
- Windows：修「注册表托管设置含非 ASCII（如日文）时拒绝启动」
- Windows：修「内存将尽时可能崩溃」
- Windows：Glob 在 6+ 核机器上并行遍历，速度提升
- 修「沙箱化 Bash 在临时盘满时启不来」
- 改进「`claude -p` 与 SDK 会话在会话开始时的内存」
- 改进「启动时 MCP 服务器列出大量工具的内存」
- 改进「恢复长会话的内存」
- 改进「`/usage` 在近期会话很多时的峰值内存与速度」
- Linux：Ctrl+V 读不到剪贴板图片时，提示里会告知安装 `wl-clipboard` 或 `xclip`
- 改进「`/permissions` 会拒绝下次启动会被跳过的规则（如无工具名的 `rm -rf`）并说明原因」

## 四、即将到来的变更（提前准备）

- **2026-11-30 或之后**：托管 `extraKnownMarketplaces` 里的插件市场将**默认自动更新**；不想更新就设 `"autoUpdate": false`
- **Windows 注册表托管设置**：`HKLM` 策略优先级**高于** `managed-settings.json`，值写坏会导致启动失败 —— 升级前先检查
- **macOS computer use** 更新了安全规则：部分动作会被拒绝、输入可能变慢
- OTel：桌面 Code tab 会话默认 `service.name` 改为 `claude-code-desktop`（如果用旧名做过滤，需更新或设 `OTEL_SERVICE_NAME`）

---

# 第二部分：安装与验证

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.332（原生二进制，383.7 MB） | 官方 CDN 产物，sha256 `a2df1f41…d026` **与官方 manifest 完全一致** |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = a2df1f41bb00250992926420d0063f89b127cb81abdd284dbefb80c50ff1d026
            size   = 402,325,736 bytes
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
sha256sum -c claude-offline-2.1.332-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/）
tar -xzf claude-offline-2.1.332-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

```bash
source ~/.bashrc
claude --version      # 期望：2.1.332 (Claude Code)
```

## 四、常用参数

```bash
bash install-offline.sh --link           # 省 383MB（符号链接，但离线包不能删）
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
- **代理会截断压缩响应时**：加 `"CLAUDE_CODE_DISABLE_RESPONSE_COMPRESSION": "1"`（本版新增）。
- 若用沙箱且想免提示：加 `"sandbox": {"autoAllowBashIfSandboxed": true}`。
- 内网链路慢：可设 `"CLAUDE_STREAM_IDLE_TIMEOUT_MS": "600000"`。

> **离线安装 ≠ 离线调用模型**。

## 六、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/versions/2.1.332   # 二进制 383.7 MB
~/.local/share/claude/runtime/           # 随身运行时（5.9 MB）
~/.local/bin/claude                      # 启动器（#!/bin/sh）
~/.claude.json                           # installMethod=native, autoUpdates=false
~/.claude/settings.json                  # 模型端点与 Key（需自行创建）
```

安装后实测占用：**390 MB**（含 5.9 MB 运行时）。

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
5. **本版压缩与网关相关修复较多**（413 恢复、摘要丢失、静默超时重试），建议尽快升级。
6. 升级后请核对第一节的**模型别名变更**，确认 `ANTHROPIC_MODEL` 仍指向你要的模型。

## 九、分批传输

```bash
# 发送端
split -b 90M -d claude-offline-2.1.332-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.332-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.332-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、重建步骤）在归档内 `claude-offline/README.md`。
