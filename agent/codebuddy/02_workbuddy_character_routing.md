# WorkBuddy 专用规则：角色卡路由

> **部署与引入说明**：本文件是分散式规则模块，不直接作为 WorkBuddy 的最终全局规则文件。WorkBuddy 桌面端的全局规则目录为 `~/.workbuddy/rules/`，角色卡目录为 `~/.workbuddy/character/`。部署时**必须同时将下方路由所需的角色卡文件（含 `default.md`）从仓库 `character/` 目录复制到 `~/.workbuddy/character/`**，并把规则中的读取路径指向复制后的实际位置。下文结论已于 2026-11-26 在本机（Windows，WorkBuddy 版本对应 `app.asar` 构建 2026-11-13）实测通过。
>
> 与同目录 `01_character_routing.md` 的分工：那份面向 CodeBuddy（IDE / Code CLI）场景，本份面向 WorkBuddy 桌面端；两者配置根不同，不可互换路径。

## 1. 存放位置与格式

| 项目 | 路径 | 要求 |
|---|---|---|
| 全局规则目录 | `~/.workbuddy/rules/` | 该目录下的 `.md` / `.mdc` 文件会被扫描并注入；目录不存在时需自行创建 |
| 角色卡目录 | `~/.workbuddy/character/` | 任意 `.md` 文件，规则中按绝对路径引用 |
| 规则文件格式 | 带 frontmatter 的 Markdown | 必须 `alwaysApply: true` 或提供 `paths`，否则规则被跳过（`alwaysApply:false` 且无 `paths` 会打印 skip 日志） |
| 单文件大小上限 | 40000 字符 | 超出则该规则被丢弃 |

配置根可被环境变量 `WORKBUDDY_CONFIG_DIR` 或 `CODEBUDDY_CONFIG_DIR` 覆盖；若被改写，上述两个路径需相应迁移。

## 2. 需复制的角色卡清单

- `character/deepseek.md`
- `character/qwen.md`
- `character/glm.md`
- `character/kimi.md`
- `character/hunyuan.md`
- `character/default.md`

## 3. 模型路由

| 当前模型 | 读取角色卡 |
|---|---|
| DeepSeek 系列模型（`deepseek-*`） | `deepseek.md` |
| 千问 / Qwen 系列模型（`qwen*`） | `qwen.md` |
| GLM / 智谱系列模型（`glm-*`） | `glm.md` |
| Kimi / 月之暗面系列模型（`kimi-*`、`moonshot-*`） | `kimi.md` |
| 混元 / Hunyuan / Hy 系列模型（`hy*`、`HY*`、`Hy*`、`hunyuan*`、`Hunyuan*`） | `hunyuan.md` |
| 其他模型或无法确定当前模型身份 | `default.md` |

## 4. 部署步骤

1. 复制角色卡（仓库根目录执行，`<REPO>` 为仓库绝对路径）：

```bash
mkdir -p ~/.workbuddy/character ~/.workbuddy/rules
cp "<REPO>/character/deepseek.md" "<REPO>/character/qwen.md" "<REPO>/character/glm.md" \
   "<REPO>/character/kimi.md" "<REPO>/character/hunyuan.md" "<REPO>/character/default.md" ~/.workbuddy/character/
```

2. 在 `~/.workbuddy/rules/01_character_routing.md` 写入规则，frontmatter 固定为：

```markdown
---
description: 按当前模型身份加载对应角色卡（deepseek / qwen / glm / kimi / hunyuan / default），仅作用于对话表达层
alwaysApply: true
enabled: true
---

# 角色卡路由

## 一、角色卡位置

- `C:\Users\<用户名>\.workbuddy\character\deepseek.md`
- `C:\Users\<用户名>\.workbuddy\character\qwen.md`
- `C:\Users\<用户名>\.workbuddy\character\glm.md`
- `C:\Users\<用户名>\.workbuddy\character\kimi.md`
- `C:\Users\<用户名>\.workbuddy\character\hunyuan.md`
- `C:\Users\<用户名>\.workbuddy\character\default.md`

## 二、模型路由

（复制本文第 3 节表格）

## 三、加载方式

- 会话首次涉及对话回复时，按上表读取命中角色卡全文，并在本会话内保持该人设；同一会话无需重复读取。
- 模型切换后重新判定一次，切换到新模型对应的卡片。
- 若卡片文件缺失或读取失败，退回 `default.md`，并在回复中说明该情况，不得凭空虚构卡片内容。

## 四、使用边界

（复制本文第 6 节三条）
```

3. 路径务必写成**本机绝对路径**（含实际用户名）。写 `~` 形式在部分工具调用中不会被展开，会导致读取失败。

## 5. 生效与验证

- 规则文件保存后热加载立即触发，但**规则正文在会话开始时注入**，需新建对话才完整生效。
- 日志验证（`~/.workbuddy/logs/<日期>/`）应出现：

```
[HotReload] Triggered by rules change: ...\.workbuddy\rules\01_character_routing.md
[MemoryLoader] Loaded 1 memory rules:
  [user] [always] ...\.workbuddy\rules\01_character_routing.md
```

其中 `[user]` 表示用户级、`[always]` 表示全量注入。若显示 `No memory rules loaded`，检查目录与 frontmatter。
- 会话内验证：提问「当前应用了哪些规则」，应列出该规则文件。

## 6. 常见误区

- **`~/.workbuddy/AGENTS.md` 与 `~/.workbuddy/CODEBUDDY.md` 不生效**。这两个文件名只在**项目根目录**被扫描（候选列表为 `CODEBUDDY.md`、`.codebuddy/CODEBUDDY.md`、`AGENTS.md`），放在配置根下不会被读取。
- **不要写进 `~/.codebuddy/rules/`** 期待桌面端生效：桌面端配置根为 `~/.workbuddy`，`~/.codebuddy` 是 CodeBuddy CLI 的遗留目录。
- **项目级配置不适合做全局**：WorkBuddy 每个任务使用独立工作目录，项目规则只对单个工作目录有效。

## 7. 使用边界

- 角色卡仅作用于日常工作对话层的称呼、互损与表达风格；全局工程规则及系统指令优先级高于角色卡。
- 代码、代码注释、Commit 信息与正式文档正文自动退出人设，使用纯粹规范的工程表达。
- 无需在回复中声明当前使用的角色卡文件名；若用户口头要求切换或关闭人设，直接听从用户。
