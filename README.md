# PPTPRO.SKILL

PPT 生成技能合集（Agent Skills 格式），来自长期实战迭代优化后的 PowerPoint 自动化工作流。

## 内容

| 技能 | 用途 |
|------|------|
| `skills/pptx` | 完整的 .pptx/.potx 创建、读取、编辑、分析技能包（含 OOXML 脚本工具链：add_slide / clean / chart / theme helpers 与 ECMA-376 schema 校验）。Anthropic 官方技能，许可见其目录内 LICENSE.txt。 |
| `skills/hermes-pptx-slides` | Windows + python-pptx 实战经验：非 ASCII 路径绕行、指定位置插页（sldIdLst 重排）、完整性校验，以及 **HTML → PPTX 纯离线转换器**（容器偏移栈模型，无需 Playwright/浏览器引擎）。 |
| `skills/browser-native-slide-deck` | 无 PPTX 工具链时的兜底：单文件 HTML 幻灯片（固定 1280×720、浏览器原生导航、零运行时依赖）。 |

## 快速决策表

| 需求 | 用哪个 |
|------|--------|
| 程序化生成/编辑真实 .pptx（python-pptx） | `hermes-pptx-slides` + `pptx` |
| HTML 草稿转高保真 PPTX（离线，无浏览器） | `hermes-pptx-slides` → `references/html-to-pptx-converter.md` |
| 完整 OOXML 工具链 / schema 校验 / 模板操作 | `pptx` |
| 只要能在浏览器里放的演示稿 | `browser-native-slide-deck` |

## 安装

每个子目录都是一个标准 Agent Skill（`SKILL.md` + 支撑文件）。将所需子目录整体拷入你的 agent 技能目录即可，例如：

```bash
cp -r skills/hermes-pptx-slides ~/.claude/skills/      # Claude Code
cp -r skills/hermes-pptx-slides ~/AppData/Local/hermes/skills/productivity/   # Hermes (Windows)
```

## 许可

- `skills/pptx`：Anthropic 专有许可，详见 `skills/pptx/LICENSE.txt`。
- 其余两个技能为本仓库原创，MIT。
