# github-reuse — GitHub 开源复用

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

一个 WorkBuddy Skill：**为现有项目在 GitHub 上寻找、审查并验证可复用的代码、依赖与设计，给出可复核的适配判断，需要时完成集成。**

核心不是「帮你搜仓库」，而是**先建立项目基线 → 找到真实缺口 → 对候选取证据 → 分级验证 → 交付可复核的结论**。要点在于「可复核」：每条结论都标注证据层级和实际读过的 commit SHA。

## 它解决什么问题

让 AI 找轮子，常见的失败模式是：搜一堆高 star 仓库、把 README 的宣称当成事实、然后给你一份看起来很美但跑不通的清单。

这个 Skill 强制走一条更慢但更可靠的路：

| 阶段 | 关键约束 |
|---|---|
| 建立基线 | **先确认项目是否已经解决该问题**，再推荐新组件。已有库的正确配置、已有框架的内置能力也算复用 |
| 搜索 | Star / README / 搜索摘要**只用于发现，不能证明功能正确** |
| 审查 | 定位到**具体实现函数、配置或测试**，记录实际阅读的 commit SHA |
| 验证 | 用**本项目真实需求**构造有区分力的用例；微型实验只能证明它覆盖的机制 |
| 交付 | 明确区分「建议采用」「局部验证可用」「已经接入」，**不把研究完成说成集成完成** |

## 证据层级

每个结论都要标记，可并列：

| 层级 | 含义 |
|---|---|
| 1 源码核查 | 阅读实现、依赖、许可及相关测试，**尚未运行** |
| 2 局部实测 | 在特定版本和样例上验证了部分行为 |
| 3 集成验证 | 已接入本项目，并通过相关业务回归 |
| 4 待验证 / 不采用 | 写清缺失证据、具体不匹配或成本原因 |

## 安装

把 `SKILL.md` 放到 WorkBuddy 的用户级技能目录：

```
~/.workbuddy/skills/github-reuse/SKILL.md
```

无需注册或索引文件，下次启动 WorkBuddy 时会自动被发现。

## 触发方式

说这些话会触发：

- 「不重复造轮子」
- 「找个可借鉴的项目」
- 「评估这个开源方案能不能用」
- 「这个功能别人是不是已经做过」
- 「换成开源方案」「引个现成依赖」

**不会触发**：普通功能开发、只问「最近有什么热门仓库」、纯技术科普。这三种情况缺少「当前项目的真实缺口」这个判据，本 Skill 的核心方法用不上。

## 依赖

需要 [GitHub CLI](https://cli.github.com/)（`gh`）并已登录。Skill 内用到的命令：

```bash
gh auth status
gh search repos "<能力> <生态>" --limit 20 \
  --json fullName,description,stargazersCount,pushedAt,license
gh search code  "<函数名/错误信息>" --limit 10 --json repository,path
gh api repos/<owner>/<repo> \
  --jq '{archived, pushed_at, license:.license.spdx_id, branch:.default_branch}'
gh api "repos/<owner>/<repo>/contents/<path>?ref=$SHA" -H "Accept: application/vnd.github.raw"
gh api "repos/<owner>/<repo>/license" --jq '{name, spdx:.license.spdx_id}'
```

> `quick_validate.py` 不检查 `version` 字段，而平台漏 `version` 会直接报「缺少 Skill 版本号」——
> 这是这个 Skill 之外的一个独立坑，打包上传前记得自查。

## 环境说明

SKILL.md 正文末尾有一节「环境记录」，是**原作者本机**的实测结果（`gh` 2.96.0，凭证有效，`api.github.com` 与 `raw.githubusercontent.com` 均可达）。你的环境可能不同，用前自行探测。

`raw.githubusercontent.com` 在部分网络策略或沙箱环境下会被拦截；遇到它失败时**不要下结论说「仓库挂了」**，改用 `api.github.com` 的 contents 接口取内容。

## 许可

[MIT](LICENSE)
