# Career Profile

AI-readable, desensitized career profile. 5+ years frontend, focused on senior frontend / frontend architecture roles.

**Start here:** [llms.txt](./llms.txt)

## How to use

1. Open [llms.txt](./llms.txt) (or paste its raw URL into your AI).
2. Add the JD or your internal project description.
3. Ask the AI to assess skill proficiency, match against the JD / projects, and list interview risks.

Raw URL:

```text
https://raw.githubusercontent.com/eq1024/career-profile/main/llms.txt
```

## Suggested prompt

> 请阅读这份候选人资料：https://github.com/eq1024/career-profile/blob/main/llms.txt
> 1）分析他的技术栈真实熟练度（强 / 中 / 弱 / 无，逐项附证据，不要按工作年限推断）；
> 2）将他的经历与以下 JD / 项目做横向匹配：【粘贴 JD，或项目 A、B 的描述】；
> 3）判断是否适合进入下一轮，并列出 3 个最需要在面试中验证的风险点。
> 只依据资料中的证据回答，不确定的地方请明确标注。

More prompt templates: [examples/prompts.md](./examples/prompts.md)

## Structure

```text
llms.txt                       # index + quick facts + reading paths
profile/00-evaluation.md       # recommended evaluation lens
profile/01-overview.md         # career narrative
profile/02-skills.md           # full skill matrix
profile/03-project-platform.md # Vue 3 Monorepo rebuild
profile/04-project-mobile.md   # React Native app
profile/05-project-wiki.md     # Next.js AI knowledge base
profile/06-other-experience.md # IoT / Nuxt 3 / mini programs / open source
profile/07-engineering-ai.md   # engineering + AI methodology
examples/prompts.md            # copy-paste prompts
```

## Privacy

This repository contains only the public, desensitized profile. No contact details, salary, leave reasons, interview history, internal domains, customer information, or business secrets are included.