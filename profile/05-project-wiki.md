# 项目深挖：AI 驱动的企业知识库（Next.js 15 全栈）

> 角色：全权独立开发——架构设计、技术选型、全栈实现、安全加固、部署运维。
> 时间：2026.08，约 4 天密集开发，12 个 Claude Code 会话。
> 关键词：Next.js 15、RAG、混合检索、RRF、意图路由、SQLite、FTS5、sqlite-vec、Drizzle、Docker、AI SDK、安全加固

---

## 1. 一句话定位

一个 4 天从零交付的 AI 知识库：内部员工编辑 + 对外 SSR 展示 + 第三方 Widget 嵌入问答；RAG 混合检索、意图路由、SQLite 单文件、Docker 单容器自托管。

---

## 2. 背景与约束

- 参考一个开源知识库项目，但其中约 70% 功能不适合内部场景（多租户、IM 机器人、评论、统计、License 分级等），需要大幅裁剪。
- 三类使用者：
  1. 内部员工：SSO 登录、编辑文档、AI 问答
  2. 外部访客：只读浏览已发布页面（SSR，SEO 友好）
  3. 第三方网站：Widget 免登录问答
- 部署目标：Docker 一键自托管，SQLite 单文件，零外部依赖。
- 时间约束：约 4 天完成从零到可部署。

---

## 3. 技术栈

| 类别 | 选型 |
|---|---|
| 框架 | Next.js 15.5 App Router、React 19、TypeScript 5.7 |
| 工程 | pnpm、Turborepo、Tailwind CSS v4、shadcn/ui |
| 数据库 | SQLite（better-sqlite3，WAL）、FTS5、sqlite-vec |
| ORM | Drizzle ORM + drizzle-kit 迁移 |
| AI | Vercel AI SDK 4.2、OpenAI 兼容协议、Ollama |
| Embedding | 远程 OpenAI 兼容 API 或本地 Transformers.js（all-MiniLM-L6-v2，384 维） |
| 编辑器 | Tiptap 7.x（约 20 个扩展） |
| 数据层 | React Query |
| 部署 | Docker 单容器自托管 |

---

## 4. 关键决策

### 决策 1：RAG 混合检索（Dense + FTS5/BM25 + RRF）

**问题**：
- 纯全文检索无法理解语义（"怎么重启服务器" vs "如何重启机器"）。
- 纯向量检索可能漏掉精确关键词命中。

**方案**：

```text
文档发布/变更
  → chunkMarkdown：标题分节 + 段落滑动窗口（256 / 50 token）
  → 向量化（远程 API 批 16、15s 超时，或本地 Transformers.js）
  → 单事务写库：清旧 → 写新 → 同步 FTS

检索：
  sqlite-vec MATCH + distance 排序 → topK
  FTS5 MATCH + LIKE 中文兜底 → topK
  → RRF 融合（k=60）→ top10
  → buildRAGPrompt → streamText（SSE + 引用回传）
```

**RRF**：`RRF_score(d) = Σ 1/(k + rank_i(d))`，k=60。只信排名、不碰分数，跨检索体系可比较。

**降级链**：
- 稠密空 → 纯稀疏
- 稀疏空 → 纯稠密
- 全空 → 诚实回复"知识库中没有相关内容"

### 决策 2：中文双轨检索

**问题**：SQLite FTS5 没有内置中文分词，整句中文无法有效命中。

**方案**：
- 含 CJK 字符的词条：保持原样，走 LIKE 兜底
- 非 CJK 词条：包双引号做短语匹配（同时修复 "C++" 被截成 "C" 的 bug）
- 当前千级文档量下毫秒级；暂不引入中文分词扩展，避免部署复杂度

### 决策 3：统一选模 + 维度守卫

- 写入与读取统一走 `getActiveEmbeddingModel()`
- 与已知模型维度表比对 + 查询向量双重校验
- 不匹配时跳过并告警
- 目的：消除"切换 Embedding 模型后余弦相似度 NaN、检索静默失效"

### 决策 4：隐私边界双层过滤

- 向量化入口过滤 `status='published'`
- 检索 SQL 再次过滤 `status='published'`
- 草稿/下架内容即使有残留索引也无法被召回
- 下架/删除时额外 `removeNodeEmbeddings` 双保险

### 决策 5：AI 问答意图路由（三阶段 fail-safe）

**问题**：此前每条消息都无条件走 RAG（embedding + 向量扫描 + 大上下文），寒暄/闲聊也在检索；检索为空时模型还会编造答案。

**方案**：

| 阶段 | 机制 | 说明 |
|---|---|---|
| 1 规则层 | 固定寒暄表（≤12 字全匹配） | 问候/致谢/道别直接短接；误判方向只有漏判，没有误杀 |
| 2 分类器（回落） | `generateText` 输出 `{"needKnowledge": bool}` | 旧模型无工具能力时启用；temperature 0，maxTokens 64 |
| 3 工具化（主路径） | `searchKnowledgeBase` tool + `maxSteps: 2` | 检索决策交给模型；闲聊零检索直接答 |

**能力检测**：Ollama `/api/show` 的 capabilities 数组（1h 缓存）；检测失败/超时一律视为不支持 → 回落分类器。

**原则**：拿不准就走检索——宁可多花 token，也不错答。

### 决策 6：零破坏性 ORM 迁移（766 行兼容层 → Drizzle）

**背景**：早期为快速迭代写了 766 行手写"Drizzle 兼容层"，存在 6+ 语义偏差：
- `delete().returning()` 恒返回假值 → 404 判断失效
- `db.execute` no-op → SQL 静默不执行
- `onConflictDoUpdate` 空实现 → upsert 无效
- `buildWhereClause` 只支持 eq/and/or/isNull → 复杂查询报错
- 嵌套 `and()` 展开错误
- 无 JOIN 支持

**迁移方案**：
1. 9 个 schema 重写为 sqliteTable（补齐缺失列、删除幻影表）
2. 保持导出面兼容：54 个调用文件仅 8 处定点适配，其余 46 个零改动
3. 三段式迁移：
   - JS 前置修复（SQLite 没有 ADD COLUMN IF NOT EXISTS）
   - Drizzle migrate（幂等簿记表 + 双候选路径定位）
   - FTS 回填保险（历史库 FTS 空时全量回填，防 BM25 静默失效）

**结果**：老库升级与新库建表最终状态一致；调用方几乎无感。

### 决策 7：安全加固矩阵

- 12 类漏洞系统修复
- 6 路并行审查代理产出约 120 项发现，其中 4 个 Critical
- 主线程启动真实服务做攻击性验证：伪造请求头、路径穿越 payload 等
- 修复后逐项复测

> 具体漏洞细节不在公开资料中展开；面试可讲验证方法与修复原则。

### 决策 8：Docker 单容器自托管

- SQLite 单文件 + WAL
- FTS5 + sqlite-vec C 扩展打包进镜像
- 一条 `docker run` 启动
- 目标：内部部署零外部依赖、零额外运维

### 决策 9：编辑器保存安全网

- Tiptap 自动保存 + 显式保存
- 保存状态可见、失败可重试
- 防止"编辑很久但没保存"的内容丢失

### 决策 10：文件夹批量导入三 Pass 管线

- 扫描 → 解析 → 入库分段处理
- 支持批量导入 Markdown 文档
- 失败项可定位、可重试

### 决策 11：MCP 生态接入

- 把知识库能力暴露为 AI 工具源
- 让外部 Agent 可以通过 MCP 检索知识库
- 与内部 AI 工作流打通

---

## 5. 量化结果

| 指标 | 数值 |
|---|---|
| 从零到可部署 | 约 4 天 |
| Claude Code 会话 | 12 个 |
| 参考项目功能裁剪 | 约 70% |
| 安全审查发现 | 约 120 项，4 个 Critical |
| 检索延迟（千级文档） | 约 150ms 以内 |
| 部署形态 | Docker 单容器 + SQLite 单文件 |
| 支持终端 | 内部编辑 / 对外 SSR / Widget 嵌入 |

---

## 6. 难点与排查故事

### 6.1 zod v4 与 AI SDK 的 JSON Schema 兼容性

- 项目用 zod v4，AI SDK 锁定的 `zod-to-json-schema@3.25` 不兼容
- 转换输出 `type: null`，DeepSeek 拒绝执行工具调用
- 解决：工具参数改用 `jsonSchema()` 手写 JSON Schema，不升级 SDK
- 教训：库的依赖与项目的依赖之间可能存在版本张力

### 6.2 流式协议版本演进导致静默失败

- AI SDK v4 的 SSE 协议变化：`2:` 承载 StreamData，`0:` 承载文本增量，`d:` 仅承载 finish/usage
- 影响：conversationId 与 references 静默丢失；前端"假调用真失败"
- 解决：前端三处 SSE 统一双格式兼容；`2:` 行显式解析；用 typeof 检查替代 try-catch

### 6.3 Next.js instrumentation.ts 静默不执行

- 在 Next.js 15.5.22 中实测未调用
- 解决：初始化逻辑收敛到 db 模块加载阶段；instrumentation 保留双保险

### 6.4 Drizzle 兼容层的 6 个语义偏差

- 见决策 6
- 本质是"不拆车换发动机"：保持导出接口兼容，让大多数调用方无感

### 6.5 sqlite-vec 触发器生命周期

- 迁移中无条件创建触发器 → 没有 embedding 模型的部署会报 `no such table`
- 解决：触发器与 vec0 表同生命周期，由 `ensureVecTable()` 创建
- 教训：DDL 迁移与运行时附属对象解耦

### 6.6 Docker 构建中的 sqlite-vec .so 追踪

- C 扩展的 `.so` 在构建阶段容易被漏掉
- 解决：显式复制 + 产物检查；镜像内验证加载

### 6.7 PDF 中文导出三连坑

- 字体、编码、布局三类问题
- 解决：内嵌 CJK 字体 + 统一编码 + 分页/换行处理

### 6.8 RSC 导航粘滞感

- App Router 的 RSC 导航体验问题
- 解决：合理使用客户端组件边界 + loading 状态 + 预取策略

### 6.9 SQLite WAL 备份

- 直接复制 .db 文件在 WAL 模式下可能拿到不一致快照
- 解决：使用 SQLite 备份 API / 正确处理 WAL checkpoint

### 6.10 middleware body 大小限制

- 大文件上传在 middleware 层被拦截
- 解决：调整上传路径与 body 处理策略

### 6.11 FTS5 BM25 分数换算 bug

- 排序分数换算错误导致结果顺序异常
- 解决：校准 BM25 分数语义，统一排序逻辑

### 6.12 管理员改自己资料被踢下线

- 修改账号信息后 token 失效/会话被清
- 解决：区分"账号资料变更"与"认证凭据变更"，避免误清会话

---

## 7. 取舍与不足

- 4 天交付意味着测试覆盖有限；优先保证核心链路可用与安全验证
- 中文分词用 LIKE 兜底是量级取舍；文档量上来后需要引入分词扩展或外部检索
- SQLite 适合单机/小团队；大规模多租户需要换数据库
- 本地 Embedding 与远程 API 二选一，切换需要模型维度守卫
- Widget 的匿名安全边界需要继续加固

---

## 8. 面试追问与回答要点

1. **为什么用 RRF，而不是加权分数？**
   - 稠密与稀疏的分数不可比；RRF 只用排名，天然跨体系，k=60 是常用稳健值。
2. **中文检索怎么解决？**
   - FTS5 没有中文分词；CJK 走 LIKE 兜底，非 CJK 走短语匹配；量级上来再引分词。
3. **如何避免切模型后检索静默失效？**
   - 统一选模入口 + 维度表校验 + 查询向量校验 + 不匹配告警。
4. **意图路由为什么三层？**
   - 规则层便宜、工具调用最准、分类器兜底旧模型；fail-safe 原则是拿不准就检索。
5. **ORM 迁移为什么能做到 54 个文件只改 8 个？**
   - 保持导出面兼容；迁移的是引擎，不是调用方。
6. **安全审查怎么做的？**
   - 多代理并行审查 + 真实服务攻击性验证 + 逐项复测；不是只看静态代码。
7. **SQLite 能撑住吗？**
   - 当前千级文档、单团队场景够用；WAL + FTS5 + vec 扩展；大规模要换。
8. **4 天怎么保证质量？**
   - 先砍范围、先搭骨架、优先核心链路；安全与部署单独验证；AI 生成 + 人工审查。
9. **Docker 单容器有什么代价？**
   - 扩展性有限、备份与升级要自己处理；换来部署零依赖。
10. **如果重做一次，会改什么？**
    - 更早补测试；中文分词前置评估；把 Schema/迁移做成版本化契约；Widget 安全模型单独设计。

---

## 9. 可迁移能力

- Next.js App Router 全栈开发
- RAG / 混合检索 / LLM 工程化
- SQLite + FTS5 + 向量扩展
- ORM 迁移与兼容层设计
- 安全加固与攻击性验证
- Docker 自托管与零依赖部署
- AI 辅助的快速全栈交付