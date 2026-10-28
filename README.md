# 交付说明（先读我）

内网离线安装包：**Claude Code（仅本体）**，无 root、无网络、零系统库依赖。

> **本次版本：Claude Code `2.1.311` → `2.1.312`**
> 本包只含 Claude Code 本体；模型配置直接写 `~/.claude/settings.json`（见第五节）

| 文件 | 说明 |
|---|---|
| `claude-offline-2.1.312-linux-x86_64.tar.gz` | 完整离线包（**131 MB**，解包后 290 MB） |
| `claude-offline-2.1.312-linux-x86_64.tar.gz.sha256` | 校验和 |
| `README.md` | 本文件 |

```
sha256 = 1c3de6dacdf8045437929d3df5d6d50018a443a43d6863ca09d3433c634b9cbe
```

**归档内部**（解包后可见）：`README.md`（完整文档：依赖审计/验证记录/故障排查/重建方法）、
`UNPACK-FIRST.md`（速查）、2 个安装脚本、构建脚本、官方 `manifest.json`、
随身运行时及其来源 `.deb`。

---

## 一、装了什么

| 组件 | 版本 | 来源与校验 |
|---|---|---|
| **Claude Code** | 2.1.312（原生二进制，280.0 MB） | 官方 CDN 产物，sha256 `4f4178ce…d01d` **与官方 manifest 完全一致** |
| **随身运行时** | glibc 2.31 + libstdc++ 10.2.1 + libgcc_s（5.9 MB） | Debian 11 同源三件套 |

```
Claude Code sha256 = 4f4178ce41d83e23341ebfbaecdbcf8ea6bf7baea0690025abe49eeec3a5d01d
            size   = 293,592,432 bytes
```

> **关于下载方式**：本次用国内镜像（npm 官方平台包）高速获取二进制（21 MB/s，6 秒完成），
> 下载后**按官方 manifest 逐字节校验通过**，并已随包附带官方 `manifest.json`。
> 也就是说：**下载走快路，校验走官方标准**，两者互为印证。

## 二、本版更新要点（2.1.312）

### 与你（内网 + 自建网关 + 关遥测）最相关

| 变更 | 说明 |
|---|---|
| **自定义模型名走"精简系统提示"** | Bedrock/Vertex/Foundry/Mantle 上，Claude Code 不认识的模型名（如自定义部署名、推理配置文件 ARN）现在默认使用**更短的 system prompt**；想保留完整提示设 `CLAUDE_CODE_SIMPLE_SYSTEM_PROMPT=0` |
| **拒答后的自动模型切换扩展到自定义模型名** | 之前只对已知模型生效；可用 `CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK=1` 关闭 |
| **停止发送 `claude_code.safety_event` 遥测事件** | 与「关闭非必要流量」的用法一致 |
| **子代理 `autoCompactWindow` 现在能生效** | 它可以**调低** `CLAUDE_CODE_AUTO_COMPACT_WINDOW` 设的窗口（仍不能调高）——对压缩行为有直接影响 |
| **超大 CLAUDE.md / rules 文件不再被误覆盖** | 只读到开头就写回会破坏文件；现在 Write 会先要求 Claude 读全或改为原地编辑 |
| **命令名非法不再导致所有命令加载失败** | 之前可能让 `claude -p` 完全没有输出（headless 场景致命） |
| **`cd` 进子目录后按目录/`paths:` 匹配的技能恢复可用** | 之前会找不到 |

### 其他修复与改进

- 恢复会话后首个回复偶尔丢失的问题
- PDF 读取偶发 `pdftoppm failed: Could not write image`（与清理旧会话文件并发时）
- 全屏模式「点击立即发送」、vim 模式 `d/c/y` 配 `gj/gk` 计数、`/verify` 描述过长
- `/model`、`/effort` 缓存提示更准确；effort 指示器在任务中途变化时说明会延后生效
- hooks 并发上限（每会话 24 个，含子代理；超时阈值 <1s 按 1s 计）
- 云会话 / VSCode / Slack / Code Review 若干修复（与本离线包无关）

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
> 启动器 shebang 是 `#!/bin/sh`（实测 `dash` 可直接运行，目标机不必装 bash）。

## 四、安装（3 步）

```bash
# 1) 校验
sha256sum -c claude-offline-2.1.312-linux-x86_64.tar.gz.sha256

# 2) 解包（顶层单目录 claude-offline/，不会散落）
tar -xzf claude-offline-2.1.312-linux-x86_64.tar.gz
cd claude-offline

# 3) 安装
bash install-offline.sh
```

**生效并验证**

```bash
source ~/.bashrc
claude --version      # 期望：2.1.312 (Claude Code)
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

- `ANTHROPIC_BASE_URL` 必须**内网可达**，否则 agent 无法工作。
- 若网关用 API Key 而非 Auth Token，把键名换成 `ANTHROPIC_API_KEY`。
- 用自定义模型名时，可配合第二节的 `CLAUDE_CODE_SIMPLE_SYSTEM_PROMPT` /
  `CLAUDE_CODE_DISABLE_REFUSAL_FALLBACK` 调整行为。

> **离线安装 ≠ 离线调用模型**。安装全程零联网，但运行时要能连上你配置的端点。

## 六、常用参数

```bash
bash install-offline.sh --link           # 省 280MB（符号链接，但离线包不能删）
bash install-offline.sh --system-glibc   # 不用随身运行时（改用目标机 glibc，需 ≥2.26）
bash install-offline.sh --force          # 覆盖已安装（会先备份）
bash install-offline.sh --no-path        # 不改 shell rc
```

## 七、安装位置（全部在 $HOME 下）

```
~/.local/share/claude/versions/2.1.312   # 二进制 280.0 MB
~/.local/share/claude/runtime/           # 随身运行时（5.9 MB）
~/.local/bin/claude                      # 启动器（#!/bin/sh）
~/.claude.json                           # installMethod=native, autoUpdates=false
~/.claude/settings.json                  # 模型端点与 Key（需自行创建）
```

安装后实测占用：**286 MB**（含 5.9 MB 运行时）。

## 八、卸载

```bash
rm -rf ~/.local/share/claude ~/.local/bin/claude ~/.claude.json
# 若还想清掉模型配置（含 API Key）：
# rm -rf ~/.claude
# 再删掉 ~/.bashrc 里 "# >>> claude-code >>>" ~ "# <<< claude-code <<<" 之间的行
```

## 九、注意事项

1. `claude` 首次交互运行会尝试登录 Anthropic 官方账号；内网请改用 API Key + 自建 `BASE_URL`。
2. 随身运行时只解决 **glibc 版本**问题，不改变架构与内核要求（x86_64、内核 ≥ 2.6.32）。
3. 已自动写入 `DISABLE_AUTOUPDATER=1` 与 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1`。
4. **升级已有安装**：`bash install-offline.sh --force` 会先备份旧目录再装新版；
   模型配置（`~/.claude/settings.json`）不受影响、会被继续沿用，无需重新配置。

## 十、分批传输（如介质/上传有大小限制）

```bash
# 发送端
split -b 90M -d claude-offline-2.1.312-linux-x86_64.tar.gz cc.part.
# 接收端
cat cc.part.* > claude-offline-2.1.312-linux-x86_64.tar.gz
sha256sum -c claude-offline-2.1.312-linux-x86_64.tar.gz.sha256
```

---

**详细文档**（依赖审计表、逐项验证记录、常见问题、换版本/换架构的重建步骤）在归档内
`claude-offline/README.md`。
