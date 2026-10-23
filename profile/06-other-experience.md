# 其他经历

> 这些项目补充了"多端、多行业、从 0 到 1"的广度：桌面端、SSR、原生小程序、支付、物联网、开源。

---

## 1. Electron + Vue 3 物联网超管平台（2023，独立从零负责）

**定位**：设备管理 + 能耗管理一体的物联网后台，同时交付 Electron 桌面端与 Web 端。

**技术栈**：Electron、Vue 3、TypeScript、Pinia、Vue Router、Ant Design Vue、ECharts、Vite、Monaco Editor、OverlayScrollbars、Rollup Visualizer、Mock.js、Flow CD。

**个人贡献**：
- 独立完成系统架构：路由守卫、多环境配置、Mock 开发模式
- 设计三层权限：路由级 + 菜单级 + 按钮级
- 能耗管理：Pinia 全局统一管理能耗抽象对象，实现对象与能耗设置的自动对应
- 请求层：统一拦截器；用 `declare module` 重定义 Axios 返回类型与泛型
- 集成 Monaco Editor、ECharts、打包体积分析
- 自动部署（Flow CD）

**可迁移能力**：桌面端架构、主进程/渲染进程协作、复杂状态建模、可视化。

---

## 2. Nuxt 3 官网重构（2023，架构与框架负责人）

**定位**：从旧版 CMS 迁移到 Nuxt 3 SSR 的 SEO 友好官网。

**技术栈**：Nuxt 3、TypeScript、Pinia、Lodash、Sass、Vite 4、ESLint。

**个人贡献**：
- 架构设计与技术文档输出
- 双端请求封装：SSR 端与客户端统一（`useFetch` / `useAsyncData`）
- 多环境变量管理（dev / staging / production）
- 路由规则：动态路由 + 静态生成混合策略
- SEO：`useHead` / `useSeoMeta`、结构化数据、sitemap、搜索引擎提交
- 页面动画、404 逻辑、自动部署

**可迁移能力**：SSR 数据流、hydration、SEO、Nuxt 生态、部署。

---

## 3. 智慧办公原生微信小程序（2023）

**定位**：通行、访客、工位、会议、企服、停车缴费一体的智能办公小程序。

**技术栈**：原生微信小程序、Vant Weapp、crypto-js。

**个人贡献**：
- 企服模块：购物车增删改查 + 库存校验；微信支付（统一下单 → 支付 → 回调 → 订单状态同步）
- 权限分组：不同用户组展示不同功能与个人主页菜单
- 工位可视化：SVG / Canvas 渲染平面图，支持缩放拖拽选座
- 无痛登录：中间页按需跳转 + 获取手机号，减少授权弹窗
- 通行表单封装：字段驱动验证方式

**可迁移能力**：原生小程序、支付链路、权限分组、Canvas/SVG 交互。

---

## 4. 智慧办公 SaaS（Vue 2 + TypeScript，2023）

**定位**：通行、会议室、工位、企业服务、物业服务一体的物联网 SaaS，含超管后台与用户后台。

**技术栈**：Vue 2、TypeScript、Vuex、Vue Router、Element UI、Axios、Webpack、WangEditor。

**个人贡献**：
- 权限：动态菜单 + 角色管理 + 自定义指令按钮级控制
- 工位可视化拖拽：`ondrop` + `draggable` + 相对坐标记录；后台拖拽添加，小程序端读取坐标渲染
- 性能：虚拟滚动优化长列表；loading 动画 + 代码分割 + gzip 优化白屏
- GitLab 自动部署
- 请求层统一拦截与错误处理

**可迁移能力**：Vue 2 大型项目、权限体系、可视化拖拽、性能优化。

---

## 5. Nuxt 3 自维修平台（全栈前端，约 1 个月）

**定位**：面向海外用户的 B2C 自维修平台，维修教程 CMS + 配件电商（Stripe）+ 品牌商入驻。

**技术栈**：Nuxt 3.15、Vue 3.5、TypeScript、Pinia + persist、Element Plus、UnoCSS、WangEditor、Fabric.js、vue-draggable-next、@nuxtjs/i18n、Stripe、PM2。

**个人贡献**：
- 独立负责前端从 0 到 1 的架构与实现
- 教程内容系统：富文本、步骤拖拽排序、Canvas 标注
- 电商：商品、购物车、Stripe 支付
- 多语言框架、SEO、PM2 集群部署（3 实例）
- 约 70% 代码由 AI 辅助生成（Gemini 2.5 Pro + Claude Sonnet），人工负责架构、审查与修复

**可迁移能力**：Nuxt 全栈、内容系统、支付、Canvas、AI 辅助交付。

---

## 6. 早期项目（2021–2022）

- 原生小程序 + uni-app 小程序：商品、订单、支付、预约、表单
- 通用后台 RBAC：动态菜单 + 角色 + 按钮级指令
- 统一请求封装、XSS 防护、时间选择器/表单组件二次封装
- Linux / Docker / Nginx 部署与 Node 监控脚本

**价值**：打下了"从页面到部署"的完整基础，不是只会写业务组件。

---

## 7. 开源与个人项目

### WinPaste（Rust，270+ GitHub Stars）

- Windows 剪贴板增强工具，替代系统 Win+V
- 独立发现需求、设计并实现
- 价值：证明能走出前端舒适区，用 Rust 做桌面工具；也体现产品思维

链接：https://github.com/eq1024/WinPaste

### trs（Monorepo 模板）

- 基于 pnpm + Turborepo 的 Monorepo 入门套件
- 与平台重构项目直接相关：说明 Monorepo 不是被动学的，而是主动沉淀

### skills（个人 Skills 库）

- 前端开发体验增强的 Skills 集合
- 把重复工作工具化

---

## 8. 这些经历补充了什么

- **广度**：Web / 移动端 / 小程序 / 桌面端 / SSR / 全栈
- **行业**：跨境电商、仓储物流、物联网、智慧办公、房产、本地生活
- **0→1**：多个项目从架构到部署独立负责
- **产品思维**：工位可视化、WinPaste、自维修平台
- **工程习惯**：CI/CD、权限、性能、部署、监控