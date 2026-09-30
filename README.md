# 人话工作汇报 · human-readable-work-report

让 Codex 把工作讲给非技术读者听：先说结果，再说明证据、风险和下一步。任务复杂时，额外生成一个可以双击打开的单文件 HTML 报告。

这个 Skill 面向需要快速判断“现在做到哪了、到底做成了什么、还缺什么”的人。它不替 Agent 编造进度，也不把命令流水账当成成果。

## 它解决什么问题

技术工作常见的汇报问题是：结论埋在过程里，术语没有解释，测试通过被误写成已经上线，推测被写成事实，读者还要自己从日志里找答案。

本 Skill 规定了一套更容易读懂的汇报方式：

- 第一屏先回答：做到了什么、当前状态、卡点、下一步和需要谁决定。
- 把内容分为已验证事实、基于事实的判断、未验证部分和需要决策的事项。
- 用具体动作和结果说话，首次出现技术术语时顺手解释。
- 简单任务用聊天短报；复杂任务用聊天结论加本地 HTML 阅读版。
- 命令、日志、完整文件清单和深层技术细节放进 HTML 折叠区，不遮住结论。

## 安装到 Codex 全局技能目录

当前 Codex 的 Skill Installer 支持从 GitHub 仓库指定技能路径。这个仓库把 Skill 放在根目录，因此使用 `--path .`，并用 `--name` 指定安装目录名称：

```bash
CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"
python3 "$CODEX_HOME/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo zhoutian1995/human-readable-work-report \
  --path . \
  --name human-readable-work-report
```

安装脚本会把它放到 `${CODEX_HOME}/skills/human-readable-work-report`。如果当前 Codex 窗口还没有刷新，重新打开会话即可。Skill Installer 会拒绝覆盖已经存在的同名目录；升级前请先备份或移走旧目录，再重新安装。

也可以手动安装：

```bash
REPO_DIR="${HOME}/projects/human-readable-work-report"
git clone --depth 1 https://github.com/zhoutian1995/human-readable-work-report.git "$REPO_DIR"
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills/human-readable-work-report"
cp -R "$REPO_DIR/." "${CODEX_HOME:-$HOME/.codex}/skills/human-readable-work-report/"
```

安装后可显式调用：

```text
$human-readable-work-report
$human-readable-work-report HTML
$human-readable-work-report 短版
```

## 什么时候会触发

它允许隐式触发，也支持显式调用。下面的优先级用于解决“要不要生成 HTML”的歧义：

| 优先级 | 你的说法或任务 | 交付方式 |
| --- | --- | --- |
| 1 | 明确说“做成 HTML 汇报给我”“用 HTML 给我汇报”“给我一个 HTML 工作报告”“做成网页汇报”，或明确说“不要 HTML” | 先遵守本轮格式要求；要求 HTML 时聊天先给结论，再生成单文件 HTML，要求不要 HTML 时只给聊天短报 |
| 2 | 明确说“只要三句话”或“给我短版” | 只给聊天短报，遵守你的格式限制 |
| 3 | 日报、周报、交接、事故复盘；或任务超过三个相互依赖的步骤、包含页面/视觉结果/流程图/方案对比、需要取舍，或跨多个文件/项目/Agent 且需要读者比较或回看 | 聊天短报加 HTML 阅读版 |
| 4 | 普通的“刚刚做了什么”“现在到哪了”，且任务很简单 | 直接给两到四句话，不强行生成 HTML |

显式调用后，`HTML`、`短版`、`周报`、`交接` 等词是格式偏好。明确的本轮要求优先于默认判断；例如你说“复杂任务也不要 HTML”，就只在聊天里汇报。

## 默认聊天格式

短任务可以压缩成三行。内容足够复杂时，通常使用下面的结构：

```markdown
## 一句话结论
登录页面已经修好，测试也通过了。

**状态：** 已完成

## 这次真正做成了什么
- 修复登录失败时没有显示错误提示的问题。
- 增加了两个覆盖成功和失败路径的测试。

## 目前卡在哪里
- 暂无。

## 下一步
1. 在真实测试环境再走一遍登录流程。

## 需要你决定
- 暂无。

<details>
<summary>证据与技术细节</summary>

测试命令、文件和限制放在这里。
</details>
```

## HTML 报告长什么样

HTML 报告是单文件，CSS 和必要的 JavaScript 都内联，默认不依赖外部 CDN。首屏只放影响判断的内容：

1. 一句话结论
2. 当前状态
3. 已完成的结果
4. 卡点或风险
5. 下一步
6. 需要你决定的事项

详细命令、日志、文件清单和技术解释放在 `<details>` 折叠区。报告优先写到宿主环境规定的产物目录；没有约定时写到当前项目的 `work/` 目录，或者写到你指定的本地草稿路径。不会因为生成 HTML 就自动提交、推送、发布或上传。

聊天中的简短结论仍然会同时给出，避免只留下一个没人打开的文件。HTML 中的“已完成”必须能链接到真实文件、测试结果或网页；没有证据的内容会标成未验证。

## 证据纪律

汇报时按下面四类写清楚：

- **已验证事实：** 能从实际 diff、工作区状态、测试输出、生成文件或真实页面回读中确认的内容。
- **基于事实的判断：** 由事实推导出的解释，明确写成判断，不伪装成测量结果。
- **未验证/未知：** 还没有跑过、没有访问权限或无法从当前环境确认的部分。
- **需要决策：** 需要你选择方案、确认范围或提供信息的事项。

“改了文件”不等于“功能完成”，“测试通过”不等于“已经上线”。Skill 会区分本地修改、已提交、已推送、已部署和已在真实环境确认可用这几种状态，也不会猜完成率、根因、负责人、日期、业务影响或用户反馈。

## 使用边界

- 这是汇报写作规则，不是项目管理系统，不会自动收集缺失的业务数据。
- 它只能根据当前 Agent 实际看到的证据写报告；没有证据的结论仍然需要人工验证。
- HTML 是本地阅读版，不代表已经部署成网页。
- Skill 默认使用中文；你要求英文或其他语言时，可以按你的语言输出。
- 它不会替你修改代码、上线服务或上传文件。

## 设计参考

本项目是独立实现，下面的公开项目和文章用于学习“读者优先、证据可见、渐进披露和 HTML 汇报”等思路，不是运行时依赖，也不表示存在合作或背书关系：

- [human-readable-reports](https://github.com/SummerRiversound/human-readable-reports)
- [dev-report](https://github.com/delpicorp/dev-report)
- [project-status-report](https://github.com/Seneku/project-status-report)
- [codex-html-report-skill](https://github.com/Sologa/codex-html-report-skill)
- [Using Claude Code: The Unreasonable Effectiveness of HTML](https://claude.dev/blog/using-claude-code-the-unreasonable-effectiveness-of-html/)

## 许可

本项目采用 [MIT License](LICENSE)。你可以自由使用、修改和分发，但请保留许可声明。
