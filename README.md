# 交付说明（先读我）

内网离线安装包：**Claude Code + ccswitch-tui（命令行版）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.296`**（上一版打包的是 2.1.282；二进制由 228 MB 增至 245 MB）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.296-linux-x86_64.tar.gz` | 完整离线包（**158 MB**，解包后 380 MB） |
| `claude-offline-2.1.296-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件 |

```
sha256 = 9e530295b19f7971520a107a3d746d096ddb7b3e946484a06ca38867a926bacd
```

**归档内部**（解包后可见）：`README.md`（完整文档：依赖审计/验证记录/故障排查/重建方法）、
`UNPACK-FIRST.md`（速查）、4 个安装脚本、各组件与来源 `.deb`。

---

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.296（原生二进制，245 MB） | `downloads.claude.ai`，sha256 `24972e3b…d0de` **与官方 manifest 一致** |
| **ccswitch-tui** | 1.0.0（命令行/TUI 版，替代 GUI 版 cc-switch） | npm，45 个依赖包，**0 个原生模块** |
| **Node** | v22.23.3 LTS（119 MB） | nodejs.org，sha256 对官方 `SHASUMS256.txt` **校验通过** |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = 24972e3bc859fab2b46ed4c1e51f7d6130f06d3bd550811a114640de3370d0de
            size   = 257,068,216 bytes
```

## 二、目标机要求

| 项 | 要求 |
|---|---|
| 架构 / 系统 | **Linux x86_64** |
| **glibc** | **无要求** —— 两个程序都跑在自带运行时上（内核 ≥ 2.6.32） |
| **图形界面** | **不需要** —— ccswitch 是纯终端 TUI，SSH 里直接用 |
| 系统 Node | **不需要**（已随包携带） |
| 必备命令 | 安装：`bash`、`tar`、`gzip`、`sha256sum`（发行版自带）；运行：任意 `/bin/sh` |
| root | **不需要** |
| 网络 | 安装**不需要**；运行时需要能访问你配置的模型端点 |

> 已实测：用 tmpfs 盖住系统 `/usr/lib/x86_64-linux-gnu` 与 `/usr/lib64`（掩蔽后两处条目数均为 0，
> 系统 ELF 全部无法启动 —— `bash: /usr/bin/head: cannot execute: required file not found`，
> `/usr/bin/true` 退出码 127）后，两个程序靠随身运行时里的 loader 仍能正常运行。
> 启动器 shebang 是 `#!/bin/sh`（实测 `dash` 可直接运行，ash/busybox 同理，目标机不必装 bash）。
> 唯一无法随包消除的是「目标机得有个能用的 shell」—— 这也是安装脚本本身的前提，不构成额外门槛。

## 三、安装（3 步）

```bash
# 1) 校验
sha256sum -c claude-offline-2.1.296-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/，不会散落）
tar -xzf claude-offline-2.1.296-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

**生效并验证**

```bash
source ~/.bashrc
claude --version      # 期望：2.1.296 (Claude Code)
ccswitch              # 期望：CC Switch CLI 终端菜单
```

## 四、配置模型

```bash
ccswitch
#   ► 管理 Providers   —— 填 API Key / Base URL / 模型（内置 50+ 预设）
#     切换 Provider     —— 写入 ~/.claude/settings.json 的 env 节点
#     查看状态
```

切换后写入的字段：`ANTHROPIC_BASE_URL` / `ANTHROPIC_AUTH_TOKEN` / `ANTHROPIC_MODEL`。

> **离线安装 ≠ 离线调用模型**：请把 `ANTHROPIC_BASE_URL` 指向**内网可达的网关或代理**。
> 直接手工编辑 `~/.claude/settings.json` 也可以，不装 ccswitch 同样能用。

## 五、常用参数

```bash
bash install-offline.sh --no-ccswitch    # 只装 Claude Code
bash install-offline.sh --link           # 省 245MB（符号链接，但离线包不能删）
bash install-offline.sh --system-glibc   # 不用随身运行时（改用目标机 glibc）
bash install-offline.sh --force          # 覆盖已安装（会先备份）
bash install-offline.sh --no-path        # 不改 shell rc
```

## 六、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/            # Claude Code 二进制 + 随身运行时
~/.local/share/ccswitch-cli/      # Node + ccswitch-tui + 随身运行时
~/.local/bin/claude               # 启动器（用自带 loader 拉起）
~/.local/bin/ccswitch             # 启动器
~/.claude.json                    # installMethod=native, autoUpdates=false
~/.claude/settings.json           # 模型端点与 Key（ccswitch 写入）
~/.ccswitch-cli/providers.json    # ccswitch 的供应商库
```

安装后实测占用：`~/.local/share/claude` **252 MB**、`~/.local/share/ccswitch-cli` **136 MB**，合计 **387 MB**。

## 七、卸载

```bash
rm -rf ~/.local/share/claude ~/.local/bin/claude ~/.claude.json
rm -rf ~/.local/share/ccswitch-cli ~/.local/bin/ccswitch
# 再删掉 ~/.bashrc 里 "# >>> claude-code >>>" 与 "# >>> ccswitch >>>" 两个标记块
```

## 八、注意事项

1. **ccswitch-tui 只支持 Claude Code 与 Codex**，不含 OpenClaw。给 OpenClaw 配模型请直接编辑
   `~/.openclaw/openclaw.json`。
2. `claude` 首次交互运行会尝试登录 Anthropic 官方账号；内网请改用 API Key + 自建 `BASE_URL`。
3. 随身运行时只解决 **glibc 版本**问题，不改变架构与内核要求（x86_64、内核 ≥ 2.6.32）。
4. 已自动写入 `DISABLE_AUTOUPDATER=1` 与 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1`，避免离网反复重试。
5. **升级已有安装**：`bash install-offline.sh --force` 会先备份旧目录再装新版；
   模型配置（`~/.claude/settings.json`）与 ccswitch 供应商库（`~/.ccswitch-cli/`）不受影响、
   会被继续沿用，无需重新配置。

## 九、分批传输（如介质/上传有大小限制）

```bash
# 发送端
split -b 90M -d claude-offline-2.1.296-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.296-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.296-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、换版本/换架构的重建步骤）在归档内
`claude-offline/README.md`。
