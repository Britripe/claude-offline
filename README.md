# 交付说明（先读我）

内网离线安装包：**Claude Code（仅本体）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.310` → `2.1.311`**
> 本包只含 Claude Code 本体；模型配置直接写 `~/.claude/settings.json`（见第四节）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.311-linux-x86_64.tar.gz` | 完整离线包（**131 MB**，解包后 290 MB） |
| `claude-offline-2.1.311-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件 |

```
sha256 = 5ea6466f3ffc2df1384093362bd61592da25cb89fea5f40603d67c98157e05a6
```

**归档内部**（解包后可见）：`README.md`（完整文档：依赖审计/验证记录/故障排查/重建方法）、
`UNPACK-FIRST.md`（速查）、2 个安装脚本、构建脚本、随身运行时及其来源 `.deb`。

---

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.311（原生二进制，280.2 MB） | 见下方「关于本版来源」 |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = 86dc1fcc69498159eaf15f446613e847010f9036bfbbdffd4c49350be8489ada
            size   = 293,776,752 bytes
```

### ⚠️ 关于本版来源（与以往版本不同，请知悉）

构建时 **Anthropic 官方 CDN（`downloads.claude.ai`）不可达**（本机网络到该域名持续 SSL/连接失败），
因此二进制取自 **npm 官方平台包**：

```
@anthropic-ai/claude-code-linux-x64@2.1.311
https://registry.npmmirror.com/@anthropic-ai/claude-code-linux-x64/-/claude-code-linux-x64-2.1.311.tgz
```

**为什么可信**：该包内的二进制与 CDN 产物**逐字节相同** —— 已用上一版做对照验证：
下载 `...@2.1.310` 的 npm 平台包，其内二进制 sha256 = `b7367643…df56`，
与 2.1.310 官方 manifest 记录的校验和**完全一致**。

**差异**：本版**未随包附带 `manifest.json`**（取不到 2.1.311 的官方清单）。
来源与推导过程完整记录在 `claude-code/BUILD-INFO.txt`。

> 如果你要求必须与官方 manifest 逐条核对，请等 CDN 恢复后重跑一次构建（见归档内「如何重建」章节）。

## 二、目标机要求

| 项 | 要求 |
|---|---|
| 架构 / 系统 | **Linux x86_64** |
| **glibc** | **无要求** —— 跑在自带运行时上（内核 ≥ 2.6.32） |
| **图形界面** | **不需要** —— 纯 CLI |
| 系统 Node | **不需要** |
| 必备命令 | 安装：`bash`、`tar`、`gzip`、`sha256sum`（发行版自带）；运行：任意 `/bin/sh` |
| root | **不需要** |
| 网络 | 安装**不需要**；运行时需要能访问你配置的模型端点 |

> 已实测：用 tmpfs 一次性盖住系统 `/usr/lib/x86_64-linux-gnu` 与 `/usr/lib64`
> （掩蔽后两处条目数均为 0，系统 ELF 全部无法启动 —— `/usr/bin/true` 退出码 127）后，
> Claude Code 靠随身运行时里的 loader 仍能正常运行。
> 启动器 shebang 是 `#!/bin/sh`（实测 `dash` 可直接运行，ash/busybox 同理，目标机不必装 bash）。

## 三、安装（3 步）

```bash
# 1) 校验
sha256sum -c claude-offline-2.1.311-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/，不会散落）
tar -xzf claude-offline-2.1.311-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

**生效并验证**

```bash
source ~/.bashrc
claude --version      # 期望：2.1.311 (Claude Code)
```

## 四、配置模型端点

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

- `ANTHROPIC_BASE_URL` 必须**内网可达**，否则 agent 无法工作。
- 若网关用 API Key，把键名换成 `ANTHROPIC_API_KEY`。
- 也可用环境变量临时覆盖（优先级更高）：`export ANTHROPIC_BASE_URL=...`

> **离线安装 ≠ 离线调用模型**。安装全程零联网，但运行时要能连上你配置的端点。

## 五、常用参数

```bash
bash install-offline.sh --link           # 省 280MB（符号链接，但离线包不能删）
bash install-offline.sh --system-glibc   # 不用随身运行时（改用目标机 glibc，需 ≥2.26）
bash install-offline.sh --force          # 覆盖已安装（会先备份）
bash install-offline.sh --no-path        # 不改 shell rc
```

## 六、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/versions/2.1.311   # 二进制 280.2 MB
~/.local/share/claude/runtime/           # 随身运行时（5.9 MB）
~/.local/bin/claude                      # 启动器（#!/bin/sh）
~/.claude.json                           # installMethod=native, autoUpdates=false
~/.claude/settings.json                  # 模型端点与 Key（需自行创建）
```

安装后实测占用：**287 MB**（含 5.9 MB 运行时）。

## 七、卸载

```bash
rm -rf ~/.local/share/claude ~/.local/bin/claude ~/.claude.json
# 若还想清掉模型配置（含 API Key）：
# rm -rf ~/.claude
# 再删掉 ~/.bashrc 里 "# >>> claude-code >>>" ~ "# <<< claude-code <<<" 之间的行
```

## 八、注意事项

1. `claude` 首次交互运行会尝试登录 Anthropic 官方账号；内网请改用 API Key + 自建 `BASE_URL`。
2. 随身运行时只解决 **glibc 版本**问题，不改变架构与内核要求（x86_64、内核 ≥ 2.6.32）。
3. 已自动写入 `DISABLE_AUTOUPDATER=1` 与 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1`，避免离网反复重试。
4. **升级已有安装**：`bash install-offline.sh --force` 会先备份旧目录再装新版；
   模型配置（`~/.claude/settings.json`）不受影响、会被继续沿用，无需重新配置。
5. **本版来源为 npm 官方平台包**（CDN 不可达），等价性已验证，详见第一节。

## 九、分批传输（如介质/上传有大小限制）

```bash
# 发送端
split -b 90M -d claude-offline-2.1.311-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.311-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.311-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、换版本/换架构的重建步骤）在归档内
`claude-offline/README.md`。
