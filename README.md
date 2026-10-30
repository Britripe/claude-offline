# 交付说明（先读我）

内网离线安装包：**Claude Code（仅本体）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.312` → `2.1.314`**
> 本包只含 Claude Code 本体；模型配置直接写 `~/.claude/settings.json`（见第五节）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.314-linux-x86_64.tar.gz` | 完整离线包（**135 MB**，解包后 294 MB） |
| `claude-offline-2.1.314-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件 |

```
sha256 = ef3def3d46fe83f48c880374782545d132540808d4fdb8a44813fd2c932d46ff
```

**归档内部**（解包后可见）：`README.md`（完整文档）、`UNPACK-FIRST.md`（速查）、
2 个安装脚本、构建脚本、官方 `manifest.json`、随身运行时及其来源 `.deb`。

---

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.314（原生二进制，284.4 MB） | 官方 CDN 产物，sha256 `4487042b…4594` **与官方 manifest 完全一致** |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = 4487042b4c4f5d88d4568f68ae98aae0b1ffae58864cf57fb868641bc5f84594
            size   = 298,229,464 bytes
```

> **下载方式**：国内镜像高速获取（15 MB/s，约 9 秒），下载后**按官方 manifest 逐字节校验通过**，
> 并随包附带官方 `manifest.json`。快路下载 + 官方校验，两者互证。

## 二、本版更新要点（2.1.314）

### 与你（内网 + 自建网关 + 关遥测）最相关 —— 8 条

| # | 变更 | 影响 |
|---|---|---|
| 1 | **网关拒绝 beta header 时自动降级重试** | 网关/云厂商按名字拒绝 Claude Code 的 beta header 时，会**去掉该 header 重新发一次**，不再整轮失败 |
| 2 | **网关不支持 tool search 的 `tool_reference` 时自动重试并记住** | 之前会**反复被拒**；现在只重试一次（不带 tool search），并在会话内记住 |
| 3 | **网关模型名带点号（如 `claude-opus-4.8`）不再被误判为旧模型** | 之前会算错上下文窗口、输出上限、effort、思考模式和成本 —— 自建网关命名常带点号，**这条很关键** |
| 4 | **空闲压缩扩展到「关闭遥测」和网关场景** | 之前只给开了遥测的订阅用户；现在：遥测关闭时也跑，且在 Bedrock/Vertex/Foundry/**LLM 网关**上一小时 prompt cache 生效后也跑 |
| 5 | **新增 `permissions.blockWritesOutsideWorkingDirectories`** | 开启后 Write/Edit/NotebookEdit **在所有权限模式下**都拒绝工作目录外的路径（内网收紧权限很好用） |
| 6 | **`claude -p` 启动失败时不再"静默成功"** | 之前会**退出码 0 且无输出**；现在打印错误并退出 1（headless/脚本场景必修） |
| 7 | **TLS 报 `UNABLE_TO_GET_ISSUER_CERT` 的修复** | 系统证书库里有"续期根证书与旧根交叉签名"时会连不上；内网自建 CA 场景受益 |
| 8 | **通过 SSH 粘贴图片到 Linux 不再静默失败** | 现在会提示"把文件传过去再粘路径" |

### 压缩相关（承接你上次的问题）

- **PreCompact hook 输出过大曾导致压缩失败/反复** —— 现在每个 hook 输出截断到 10,000 字符
- **空闲压缩**（见上条第 4 点）覆盖范围扩大
- Bedrock 多图会话每轮失败、`/compact` 报 `400 Input is too long` —— 已修（会先丢旧图再压缩）

### 其他值得知道的

- 长 headless/SDK 多子代理会话的内存占用降低（父会话不再把每个子代理的工具结果留到轮末）
- `~/.claude/skills/.trash`、`plugins/.trash` 不再无限增长（各保留约 1024 项）
- 后台 diff 刷新短暂占用 `.git/index.lock` 导致 git 命令报错 —— 已修
- `--setting-sources` 排除的设置文件仍会漏出部分设置（如 auto 模式规则）—— 已修
- 沙箱保护对 `claude -p --resume` / `--continue` 进入的 worktree 未生效 —— 已修
- 组织限制模型 effort 时报 `Invalid effort level` —— 现在会自动降级并说明所用档位
- `/resume` 列表把 CLAUDE.md 正文当成会话标题 —— 已修
- 大量 vim 模式、UI、云会话、Slack、Code Review 修复（与本包无关）

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
sha256sum -c claude-offline-2.1.314-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/）
tar -xzf claude-offline-2.1.314-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

```bash
source ~/.bashrc
claude --version      # 期望：2.1.314 (Claude Code)
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
- 若网关模型名带点号（如 `claude-opus-4.8`），本版已修复其上下文/输出上限的误判。

> **离线安装 ≠ 离线调用模型**。

## 六、常用参数

```bash
bash install-offline.sh --link           # 省 284MB（符号链接，但离线包不能删）
bash install-offline.sh --system-glibc   # 用目标机 glibc（需 ≥2.26）
bash install-offline.sh --force          # 覆盖已安装（会先备份）
bash install-offline.sh --no-path        # 不改 shell rc
```

## 七、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/versions/2.1.314   # 二进制 284.4 MB
~/.local/share/claude/runtime/           # 随身运行时（5.9 MB）
~/.local/bin/claude                      # 启动器（#!/bin/sh）
~/.claude.json                           # installMethod=native, autoUpdates=false
~/.claude/settings.json                  # 模型端点与 Key（需自行创建）
```

安装后实测占用：**291 MB**（含 5.9 MB 运行时）。

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
5. 想禁止写工作目录以外的文件，可在 `settings.json` 加
   `"permissions": { "blockWritesOutsideWorkingDirectories": true }`（本版新增）。

## 十、分批传输

```bash
# 发送端
split -b 90M -d claude-offline-2.1.314-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.314-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.314-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、重建步骤）在归档内 `claude-offline/README.md`。
