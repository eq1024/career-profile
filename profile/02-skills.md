# 技能矩阵

> 等级定义：**专家** = 能架构设计、解决疑难问题、指导他人；**熟练** = 能独立完成复杂业务、常见问题自解；**掌握** = 能完成常规任务；**了解** = 有实践但深度有限。
>
> 每项附证据、置信度与面试验证方式。未列出的能力不代表不具备，建议面试确认。

## 一、前端核心

| 技能 | 等级 | 证据 | 置信度 | 面试验证方式 |
|---|---|---|---|---|
| HTML / CSS | 专家 | 6+ 项目；Less / Sass / Tailwind / UnoCSS / NativeWind 均在生产使用 | 高 | 布局、层叠上下文、响应式、动画 |
| JavaScript (ES6+) | 专家 | 5 年+ 日常使用；熟悉原型链、闭包、事件循环、异步、内存与性能 | 高 | 手写 Promise / 事件循环题 / 性能排查 |
| TypeScript | 熟练 | RN App 全量 TS；Nuxt 3 全量 TS；Monorepo 预留 TS 迁移 | 高 | 泛型、类型收窄、模块声明、类型边界设计 |
| Vue 2 | 专家 | 维护 3 个大型 Vue 2 项目（合计约 2,000 个 .vue）；理解响应式原理 | 高 | Vue 2 响应式缺陷、Vuex 设计、老项目升级 |
| Vue 3 | 专家 | 从零架构三端 Vue 3 Monorepo；Composition API、Pinia、Vue Router 4 | 高 | 响应式原理、组合式复用、性能与依赖设计 |
| React | 熟练 | React Native App（74 页，全 TS）；hooks、React Query、Zustand、React Navigation | 高 | hooks 闭包、状态归属、渲染性能 |
| React Native | 熟练 | 独立开发 74 页 RN + Expo App；BLE、广播扫码、打印、RUM | 高 | 原生模块、平台差异、列表性能、EAS |
| Nuxt 3 / SSR | 熟练 | 2 个项目从零搭建到 PM2 部署；SSR、Nitro、i18n、SEO、自动部署 | 高 | SSR 数据流、hydration、缓存、部署 |
| uni-app | 熟练 | 190 页移动端 + 4+ 个 uni-app 项目；PDA / 打印 / 多端适配 | 高 | 跨端差异、条件编译、原生插件 |
| 原生微信小程序 | 熟练 | 智慧办公小程序 + 房产类小程序；支付、权限分组、Canvas/SVG | 高 | 小程序生命周期、支付、性能 |
| Electron | 熟练 | 独立交付 Electron + Vue 3 物联网超管平台（Web + 桌面双端） | 高 | 主进程/渲染进程、IPC、打包与更新 |
| Tauri v2 | 掌握（方案） | 完成双窗口 + 多进程隔离 + CSP 方案设计，未完整落地 | 中 | 只能问方案取舍，不宜当作实战 |

## 二、状态管理

| 技能 | 等级 | 证据 | 置信度 |
|---|---|---|---|
| Pinia | 专家 | Monorepo 三端 + Nuxt 3；persist、工厂模式、跨应用共享方案 | 高 |
| Vuex | 熟练 | 维护 3 个 Vuex 项目（最大单 store 约 13,875 行）；module / plugin | 高 |
| Zustand | 熟练 | RN App 全局状态；跨平台持久化（SecureStore / localStorage） | 高 |
| TanStack Query | 熟练 | RN App 服务端状态、缓存、自动刷新、乐观更新 | 高 |

## 三、UI 与样式

| 技能 | 等级 | 证据 |
|---|---|---|
| Element Plus | 专家 | Monorepo 三端统一 UI 库；二次封装 28+ 共享组件 |
| Ant Design Vue | 熟练 | 物联网超管平台全面使用 |
| Element UI / iView | 熟练 | 维护多个 Vue 2 项目；处理过版本冲突与多 UI 库混用 |
| Tailwind / NativeWind | 熟练 | RN App 快速布局；Web 部分场景 |
| UnoCSS | 掌握 | Nuxt 3 项目 |
| Vant Weapp / uView | 熟练 | 原生小程序 / uni-app 项目 |

## 四、工程化

| 技能 | 等级 | 证据 | 面试验证 |
|---|---|---|---|
| pnpm Monorepo | 专家 | 独立搭建三端 + 10 共享包；依赖边界、singleton、overrides | 共享包边界、版本冲突、发布策略 |
| Turborepo | 熟练 | 任务编排与缓存；多应用并行构建 | 缓存失效、任务依赖图 |
| Vite | 专家 | Vue 3 三端；Webpack → Vite 迁移；冷启动分钟级 → 秒级 | 插件、分包、构建优化 |
| Webpack | 熟练 | 维护 Webpack 4 项目；loader / plugin / 分包 | 构建体积、缓存、兼容性 |
| ESLint 9 | 熟练 | flat config；三端统一规则 | 规则设计、渐进式迁移 |
| Husky / lint-staged | 熟练 | pre-commit 门禁：typecheck + lint + i18n | 如何避免"门禁拖慢提交" |
| CI/CD | 熟练 | 部署脚本、GitLab CI/CD、Flow CD、EAS Build | 回滚、环境隔离、密钥管理 |
| PM2 | 掌握 | Nuxt 3 集群部署（3 实例） | 进程模型、日志、重启策略 |

## 五、网络与数据

| 技能 | 等级 | 证据 |
|---|---|---|
| Axios 封装 | 专家 | 3 套不同风格请求层：Vue 2 旧版、Vue 3 Monorepo、RN |
| HTTP 客户端工厂 | 专家 | 共享拦截器 + 依赖注入；各应用注入 Token / 登录跳转 / 仓库上下文 |
| SSE | 熟练 | 全局单例连接 + 多事件订阅 + 自动重连 |
| WebSocket | 熟练 | 局域网打印客户端；请求-响应匹配、超时、断线重连 |
| FingerprintJS | 掌握 | 请求层浏览器指纹 |
| TanStack Query | 熟练 | 服务端缓存、失效、乐观更新 |

## 六、国际化

| 技能 | 等级 | 证据 |
|---|---|---|
| Vue I18n | 专家 | Monorepo 三语（中 / 英 / 德）3,000+ 词条 |
| i18next | 熟练 | RN App 7 个命名空间、中英双语、1,800+ 词条 |
| i18n 工程化 | 熟练 | key 一致性校验 + 中文硬编码 AST 扫描 + pre-commit |

## 七、可观测性

| 技能 | 等级 | 证据 |
|---|---|---|
| OpenObserve RUM / Logs | 熟练 | Web 三端 + RN 端统一接入；会话、资源、错误、原生崩溃 |
| 端到端 trace | 熟练 | W3C tracecontext 注入自家后端域名；请求 ID 贯通 |
| 错误上报设计 | 熟练 | 4xx 显式上报、错误标题分级、查询参数剥离聚合 |

## 八、移动端 / 硬件

| 技能 | 等级 | 证据 |
|---|---|---|
| BLE 蓝牙打印 | 熟练 | 20 字节分包 + 20ms 间隔、连接时序、UUID 匹配、自动回连 |
| 局域网打印 | 熟练 | WebSocket 协议与 Web 端逐字段对齐；pending Map + 三级超时 |
| 硬件扫码枪 | 熟练 | Android 广播（PDA）、摄像头兜底、50ms 人机判别 |
| 推送 | 熟练 | FCM + Notifee；本地通知与权限处理 |
| 安全存储 | 熟练 | SecureStore / MMKV；Token / 仓库 / 打印配置持久化 |

## 九、AI 工程化

| 技能 | 等级 | 证据 |
|---|---|---|
| Claude Code | 熟练 | 日常主力工具；复杂功能 Plan-first |
| Plan-first | 熟练 | 波次拣货、架构重构等先方案后编码 |
| Multi-agent | 熟练 | 并行审查、国际化翻译、测试生成 |
| OpenSpec | 熟练 | propose → explore → apply → archive 闭环 |
| MCP | 熟练 | 需求系统 / 设计工具上下文接入 |
| Skills | 熟练 | 项目级 Skills + 个人 Skills 库 |
| AI 代码治理 | 熟练 | 四维审查：数据模型、字段复用、架构边界、副作用 |

## 十、后端 / 运维 / 其他语言

| 技能 | 等级 | 证据 |
|---|---|---|
| Node.js | 掌握 | 监控脚本、Express 微型服务、Nitro 服务端路由 |
| Docker | 掌握 | 容器制作、私有仓库、单容器自托管 |
| Nginx | 掌握 | 反向代理、静态资源、部署 |
| Linux | 掌握 | 常用命令、部署、排查 |
| Rust | 了解 | WinPaste（个人开源，270+ Stars） |
| C# | 了解 | 上位机方向探索，未用于生产 |

---

## 技能与项目映射

| 能力 | 主要项目 |
|---|---|
| Monorepo / 工程化 | 跨境售后与仓储管理系统（Vue 3 Monorepo） |
| React Native / 原生能力 | 仓库与售后作业移动端 |
| RAG / LLM 工程化 | 企业知识库（Next.js 15） |
| Electron / 桌面端 | 物联网超管平台 |
| SSR / SEO | Nuxt 3 官网重构、自维修平台 |
| 小程序 / 支付 | 智慧办公小程序、早期小程序项目 |
| AI 工作流 | 知识库、自维修平台、Monorepo 重构 |

---

## 正在补强

- **自动化测试**：Vitest / 组件测试 / E2E，目前项目以类型检查 + Lint + i18n 门禁 + 人工回归为主
- **React Web 大型项目**：RN 经验可迁移，但仍在补 React Web 的 SSR / RSC 经验
- **Node.js 深度**：高并发、消息队列、服务端缓存
- **Rust**：从个人工具向更复杂的系统编程能力扩展

---

## 给面试官的建议

- 不要用"技术名词数量"判断熟练度；用项目中的决策、取舍和排障过程验证。
- 对每个"专家"级技能，至少问一个原理题 + 一个真实项目问题。
- 对"掌握/了解"级技能，按实际岗位需要评估，不要求达到专家级。