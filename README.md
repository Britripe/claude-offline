# 交付说明（先读我）

内网离线安装包：**Claude Code（仅本体）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.316` → `2.1.318`**
> 本包只含 Claude Code 本体；模型配置直接写 `~/.claude/settings.json`（见第五节）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.318-linux-x86_64.tar.gz` | 完整离线包（**154 MB**，解包后 332 MB） |
| `claude-offline-2.1.318-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件 |

```
sha256 = 4c29138560e3187986ed36f1216887b7770459a86134f5c1111801a905b0a241
```

**归档内部**：`README.md`（完整文档）、`UNPACK-FIRST.md`（速查）、2 个安装脚本、
构建脚本、官方 `manifest.json`、随身运行时及其来源 `.deb`。

---

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.318（原生二进制，322.2 MB） | 官方 CDN 产物，sha256 `708c77b2…f0bd` **与官方 manifest 完全一致** |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = 708c77b204db30086ff6f1ed34577bda03e1be74ced80557cbf7b54249ebf0bd
            size   = 337,814,032 bytes
```

> **下载方式**：国内镜像高速获取（12.1 MB/s，约 12 秒），再**按官方 manifest 逐字节校验通过**，
> 并随包附带官方 `manifest.json`（2,101 B）。

## 二、本版更新要点（2.1.318）

### 与你（内网 + 共享家目录 + headless）最相关 —— 8 条

| # | 变更 | 影响 |
|---|---|---|
| 1 | **修「空闲压缩把会话替换成"不是摘要的回复"」** | 之前压缩可能把对话搞坏；这是压缩正确性问题 |
| 2 | **修 `--continue` 在"工具调用格式错误重试耗尽"后丢失之前对话** | **2.1.314 引入的回归**，本版修复 —— 如果你在用 `--continue` 务必升 |
| 3 | **修共享网络家目录下配置文件写不进去时**，`/logout`、`claude auth logout`、`claude mcp remove` **假报成功**、信任/同意/MCP 设置被静默丢弃 | **NFS / 网络家目录场景**（内网常见）：以前会"以为改了其实没改" |
| 4 | **修 `claude mcp add` / `remove` 写配置时不加锁** | 现在会等锁或退出 1；避免并发写坏配置 |
| 5 | **修 `claude plugin` 命令在"令牌刷新慢"时提前退出**（可能导致下次运行掉登录） | 内网网关卡顿场景受益 |
| 6 | **修自定义斜杠命令在 `claude -p` / SDK 里什么都不打印** | headless 输出缺失 |
| 7 | **修 `claude -p` / SDK 在"API 错误截断回复"时仍报成功** | headless 误报成功 → 脚本误判 |
| 8 | **修版本清理误删"其他用户的 Claude 进程"的二进制**（如残留的 `sudo claude`） | 多用户共享机器上的安全隐患 |

### 其他值得知道的

**登录 / 凭据**
- `claude auth status` 新增 `loginExpiresAt`；`claude -p` JSON 结果新增 `credential_expires_at`
- 存储的 claude.ai 登录**到期前 10 天**开始向 stderr 打一行警告
- 修"同一账号在另一终端重新登录后，本终端任务报 `Not logged in`"
- 修"另一个登录存的 `allowedProviders` 让本机启动被拒"

**headless / SDK**
- 改模型后立刻发的消息，在 headless 里 Stop 不能取消 —— 已修
- 空闲非交互会话退出时，会**重建已删除的会话记录** —— 已修
- `claude remote-control` 主机 / 后台守护 / 自托管 runner 在"更新删掉了仍在运行进程的二进制"后报 `spawn ENOENT` —— 已修

**MCP / 插件**
- stdio MCP 服务在版本探测时退出会被**每次启动重复探测**；自行 3 秒内退出的服务会被固定用 legacy 握手 7 天
- `claude plugin validate` 对自定义名字的 marketplace 文件误放行 —— 已修

**交互 / 平台**
- 权限提示会被"刚发完消息的按键"误答 —— 已修
- macOS 多并发 Bash 命令导致卡顿且内存增长 —— 已修
- Claude apps gateway 的 `mantle` upstream 新增 `assume_role`（跨账号角色 + 按开发者分摊 AWS 成本）
- mod 的 `$.model.fork` 新增 `effort` 选项（旁路小任务可用 `low`）
- 云会话 / Remote Control / Claude Tag 若干修复（与本包无关）

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
sha256sum -c claude-offline-2.1.318-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/）
tar -xzf claude-offline-2.1.318-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

```bash
source ~/.bashrc
claude --version      # 期望：2.1.318 (Claude Code)
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
bash install-offline.sh --link           # 省 322MB（符号链接，但离线包不能删）
bash install-offline.sh --system-glibc   # 用目标机 glibc（需 ≥2.26）
bash install-offline.sh --force          # 覆盖已安装（会先备份）
bash install-offline.sh --no-path        # 不改 shell rc
```

## 七、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/versions/2.1.318   # 二进制 322.2 MB
~/.local/share/claude/runtime/           # 随身运行时（5.9 MB）
~/.local/bin/claude                      # 启动器（#!/bin/sh）
~/.claude.json                           # installMethod=native, autoUpdates=false
~/.claude/settings.json                  # 模型端点与 Key（需自行创建）
```

安装后实测占用：**329 MB**（含 5.9 MB 运行时）。

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
5. **本版修了 2.1.314 的 `--continue` 回归**，如果你在用 `--continue`，建议尽快升到本版。

## 十、分批传输

```bash
# 发送端
split -b 90M -d claude-offline-2.1.318-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.318-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.318-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、重建步骤）在归档内 `claude-offline/README.md`。
