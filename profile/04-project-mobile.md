# 项目深挖：仓库与售后作业移动端（React Native + Expo）

> 角色：独立开发（从零到一），前端完全负责。
> 时间：2026 年，活跃开发中。
> 关键词：React Native、Expo Router、Zustand、TanStack Query、硬件扫码、双通道打印、OpenObserve RUM、i18n 门禁、动态表单、权限归一化

---

## 1. 一句话定位

一个面向仓库操作员与维修工程师的 React Native 业务 App：74 个页面、33 个组件、19 个 service，覆盖收货、上架、库存、出库、售后工作台、维修/翻新、任务单与标签打印。复杂点不在 UI，而在**硬件、打印、扫码、权限、可观测性这些"非页面"能力**。

---

## 2. 背景与约束

- 旧版是 uni-app 移动端，依赖 HBuilderX，构建与调试体验受限。
- 业务角色分两类：仓库操作员（收货/上架/库存/出库/拣货）与工程师（维修/翻新/DOA/物料申领）。
- 工业 PDA 使用硬件扫码枪（Android 广播），不是摄像头扫码。
- 标签打印有两条路线：蓝牙便携打印机、仓库本地局域网打印服务。
- 多仓库场景：所有业务数据都要带当前仓库上下文。
- 需要中英双语；后续还要接权限、可观测性、OTA。

---

## 3. 架构总览

```text
app/
  _layout.tsx            根 Provider + 登录态 + FCM + RUM
  login.tsx
  (tabs)/                登录后主界面：首页 / 个人中心 / 自定义 TabView
  (view)/                业务 Stack：warehouse / engineer / profile / tools / task-orders
components/              DynamicForm、ScanInput、打印弹层、Picker、Sheet 等
services/                按业务域拆分的 API 调用
store/                   Zustand：userStore、printStore
locales/{zh,en}/         7 个命名空间
utils/                   request、rum、permission、scanUtils、label 等
```

**根 Provider 层级**（顺序有意义）：

```text
QueryClient → GestureHandler → SafeArea → BottomSheet → Keyboard → ErrorBoundary
```

再加登录守卫、FCM 初始化、广告遮罩、RUM 初始化。

---

## 4. 关键决策

### 决策 1：选 Expo + React Native，而不是继续 uni-app 或裸 RN

**原因**：
- EAS Build 统一 Android / iOS 构建与签名
- EAS Update 支持 OTA 热更新，紧急修复不必等商店审核
- Expo 生态覆盖 Camera、SecureStore、ImagePicker、Video 等常用能力
- 标准 VS Code + Expo CLI，开发体验优于 HBuilderX

**代价**：
- 原生模块需要 development build，Expo Go 不能完整测试 BLE / 广播
- 部分第三方 RN 库不兼容 Expo
- EAS 构建比本地慢

**结论**：对业务 App 来说，OTA + 构建规范化的收益大于原生自由度损失。

### 决策 2：Expo Router 文件路由 + 自定义 TabView

- `(tabs)` 是登录后主界面，`(view)` 是带自定义头部的业务 Stack。
- TabBar 不是标准 BottomTabs：需要顶部仓库选择器、自定义图标、手势滑动与动画。
- 基于 TabRouter + TabView + PagerView 实现仿微信式导航。
- **取舍**：自研导航更灵活，但需要自己处理焦点、手势冲突、Tab 状态保持。

### 决策 3：Zustand 管全局状态，TanStack Query 管服务端状态

**边界**：
- Zustand：登录态、用户信息、当前仓库、打印配置等**客户端全局状态**
- TanStack Query：列表、详情、服务端缓存、刷新、乐观更新等**服务端状态**

**原因**：把服务端数据塞进 Zustand 会导致缓存、失效、重试都要手写；把 UI 全局状态塞进 Query 也不合适。两边分工后，页面逻辑明显变薄。

**持久化**：Token / 用户 / 仓库 / 打印配置在原生走 SecureStore，Web 走 localStorage；应用启动时恢复。

### 决策 4：统一请求层

- 自动注入 `Authorization: Bearer <token>`
- 自动注入仓库上下文请求头
- GET 追加 `_t` 时间戳防缓存
- 401 → 清理登录态 → 跳登录
- 双语错误：优先 `details`，其次当前语言的 message
- 记录响应头 `x-request-id`，错误提示复制时附带该 ID，便于后端按链路定位问题
- 422 / 业务错误按统一格式处理

**价值**：页面层不关心认证、仓库上下文、错误文案与追踪 ID。

### 决策 5：扫码体系（硬件 + 摄像头 + 人机判别 + 规则裁剪）

**硬件扫码枪**：
- Android PDA 通过广播返回扫码结果（如 Seuic 的 `com.seuic.scan`）
- 用 `react-native-broadcast-receiver` 在 JS 层注册监听
- 回调放 ref，避免频繁重建监听器
- 按平台拆分 hook（Android / iOS / Web）

**摄像头兜底**：
- 没有硬件扫码枪时，用 expo-camera 扫码
- 支持手电筒、权限处理、扫描后重置

**人机判别**：
- 50ms 击键间隔阈值：扫码枪是极速连贯输入，人手输入间隔更大
- 用于区分"扫码"与"手动输入"，避免误触发

**规则裁剪**：
- 后端下发规则：条件（长度 / 包含 / 前缀 / 正则）+ 动作（取子串 / 分割 / 后处理）
- 从物流单号中提取真正需要的单号
- 与 Web 端 v-scan 使用同一套规则结构

**ScanInput 统一组件**：
- `requireFocus`：必须聚焦才接收广播
- `keepFocus`：扫完保持焦点，适合连续扫码
- `clearAfterComplete`：扫完清空，适合标签录入

### 决策 6：双通道标签打印（蓝牙 + 局域网）

**蓝牙通道**：
- 基于 BLE（react-native-ble-plx）重写，移植旧版的关键参数：
  - 20 字节分包 + 20ms 间隔（防打印机缓冲区溢出）
  - 10s 连接超时
  - 连接后延时 500ms 再发现服务
  - 按打印服务 UUID 匹配
  - 设备名前缀 + RSSI 过滤
- 8s 超时自动回连上次打印机；连接成功后自动发送待打印数据

**局域网通道**：
- 自研 WebSocket 客户端，直连仓库本地打印服务
- 协议与 Web 端逐字段对齐：`{ action, payload }`；`printersList` / `printSuccess` / `printError`
- 所有 `print*` 动作共用 `print` 响应槽位（与 Web 端一致）
- pending Map 做请求-响应匹配
- 三级超时：连接 5s / 常规请求 8s / 打印 60s（多张标签服务端合成 PDF）
- 同槽位新请求先拒绝旧请求；连接断开统一 rejectAll

**printStore**：
- 服务地址（用户配置 > 环境变量默认）
- 连接状态机：idle / connecting / connected / error / unconfigured
- 打印机列表、当前打印机（按设备持久化）
- "自动打印"开关；服务断开时自动关闭，避免静默失败
- 打印优先级：调用方指定 > 用户选择 > 第一台

### 决策 7：OpenObserve RUM 全链路可观测

**覆盖**：
- 会话采样、网络资源、未捕获错误、原生崩溃
- service 按端拆分（app-android / app-ios），version 用于发布健康度
- 自家后端域名注入 W3C tracecontext，端上请求可追到后端 trace
- 登录绑定用户、登出清除

**三个关键排坑**：
1. **NativeWind v4 冲突**：SDK 的自动点击追踪靠改写 `React.createElement`，与 NativeWind 的 Babel 插件冲突，启动即崩。关闭自动追踪，改用 `addRumAction` 手动埋点关键业务动作。
2. **Web 端崩溃**：SDK 导入链在模块求值阶段调用 TurboModuleRegistry，react-native-web 不导出该对象。用 `.web.ts` 平台垫片把所有 RUM 导出降级为 no-op。
3. **HTTP 4xx 漏报**：trackResources 只记 Resource 事件，不生成 Error；axios 拦截器里的失败又是被 catch 的 Promise。在拦截器中显式调用 `reportHttpError`，统一错误标题（HTTP 状态码 > 业务码 > 网络错误），上报前剥掉查询参数。

另加启动自检 `checkRumNativeModule`：确认原生模块已编译进当前构建，解决"日志正常、平台零数据"。

### 决策 8：i18n 治理流水线

- 7 个命名空间：auth / common / engineer / home / profile / tools / warehouse
- 中英双语，约 1,800 词条
- `check-i18n`：校验中英 key 严格一致
- `audit-i18n`：基于 TypeScript AST 扫描中文硬编码，支持 i18n-ignore
- 纳入 `npm run check`（i18n + typecheck + lint），Husky pre-commit 执行

**价值**：把"文案治理"从 code review 变成提交门禁。

### 决策 9：动态表单逻辑引擎

- 13 种字段：文本、数字、日期、单选、多选、开关、标签输入、富文本、上传、纯展示等
- 条件逻辑：
  - AND / OR 关系
  - 10 种操作符：eq / neq / gt / gte / lt / lte / in / nin / contains / ncontains
  - 布尔与数字类型归一化
  - 空条件语义：AND 恒真 / OR 恒假
  - 静态模式：PC 端 FormDesigner 的 true/false 直接生效
- 与 PC 端表单设计器配置同源，移动端零开发复用

### 决策 10：权限归一化

**问题**：后端不同接口返回的权限结构不一致。
**方案**：
- 兼容 8 种字段形态：permissions / permission / permission_codes / permissionCodes / permission_list / permissionList / menus / modules
- 支持嵌套遍历与 `is_selected` / actions 选择标记
- `usePermission()` 暴露 `can(code | code[])`，页面只依赖一套 API
- 首页入口按权限码显隐

### 决策 11：波次拣货状态机

**场景**：一个波次包含多个拣货任务，每个任务指定库位、SKU、数量、目标格口；相同库位 + 相同 SKU 但不同目标格口需要批量拣货；缺货时需要标记并获取替代库位。

**方案**：
- `pickingArray`：怎么拣（库位 / SKU / 计划数量 / 目标格口）
- `cartArray`：拣货车每个格口的状态（应拣 / 已拣 / 已投递）
- 批量拣货：检测相同库位 + 相同 SKU，输入总数量后分配到多个格口
- 缺货流程：弹窗输入缺货数量 → 后端返回新推荐库位或全仓无货
- 分页加载：每次 10 条，触底加载更多
- 乐观更新 + 操作锁：确认后立即更新本地，失败回滚；操作未完成时禁用后续按钮

### 决策 12：构建与发布

- EAS Build：development / preview / production 三套 profile
- Android / iOS 独立构建脚本
- 生产版本号 autoIncrement（EAS remote 管理）
- EAS Update OTA

---

## 5. 量化结果

| 指标 | 数值 | 说明 |
|---|---:|---|
| 页面文件 | 74 | 排除 layout；仓储 50 / 工程师 10 |
| 组件 | 33 | 含 DynamicForm、ScanInput、打印弹层等 |
| Service | 19 | 按业务域拆分 |
| API 端点 | 233 | 集中在 endpoints 文件 |
| i18n 词条 | 约 1,800 | 7 个命名空间 × 中 / 英 |
| Git commits | 98 | 2026.03–2026.09 |
| 原生能力 | BLE 打印 / 局域网打印 / 广播扫码 / 摄像头 / FCM / RUM | 均在生产构建中使用 |

---

## 6. 难点与排查故事

### 6.1 监控 SDK 与 NativeWind 的构建链冲突

不是配置问题，是两套 Babel 转换在同一段 JSX 上的语义冲突。最后通过读 SDK 源码 + 官方废弃说明定位：关闭自动点击追踪，改为手动业务埋点。

### 6.2 Web 端 RUM 垫片

`@openobserve/mobile-react-native` 在模块求值阶段就调用 TurboModuleRegistry，react-native-web 没有该对象，浏览器直接崩溃。用 Metro 平台文件机制提供 `rum.web.ts`，所有导出降级为 no-op。

### 6.3 打印请求错配

局域网打印是"发出去后等一条响应"，没有天然请求 ID。用 action 作为 pending key，所有 `print*` 共用 `print` 槽位；新请求先拒绝旧请求；断线统一 rejectAll。三级超时按"连接 / 常规 / 打印"分开设置。

### 6.4 扫码枪 vs 人手输入

用 50ms 击键间隔判断。注意误判边界：首字符要重置状态；回车后要重置，准备下一次扫描；判定后立即重置，避免状态污染。

### 6.5 多仓库上下文

所有请求带仓库 ID；切换仓库时更新 store、持久化、调用接口更新默认仓库、重载应用。Query 缓存 key 包含仓库 ID，切换后旧缓存自动失效。

---

## 7. 取舍与不足

- **离线模式未实现**：目前依赖 React Query 缓存与网络重试，没有"离线操作 + 联网同步"。
- **RFID 未集成**：旧版有 RFID，新版暂未接。
- **测试覆盖有限**：以 typecheck + lint + i18n 门禁 + 人工回归为主。
- **Expo 约束**：原生模块必须 development build；Expo Go 无法完整验证 BLE / 广播。
- **自定义 TabView 成本**：灵活但需要自己处理焦点、手势、状态保持。

---

## 8. 面试追问与回答要点

1. **为什么选 Expo？**
   - OTA + EAS 构建规范化 + 官方 SDK 覆盖常用能力；代价是原生自由度与构建时间。
2. **Zustand 和 React Query 怎么分工？**
   - Zustand 管客户端全局状态；Query 管服务端缓存/刷新/乐观更新。不要让一个库承担两种职责。
3. **打印协议怎么和 Web 端保持一致？**
   - action/payload 结构、响应动作、print* 共用槽位都逐字段对齐；App 与 Web 操作同一套本地打印服务。
4. **怎么判断是扫码枪还是人手输入？**
   - 50ms 击键间隔；首字符与回车后重置；判定后立即清状态。
5. **RUM 为什么需要 Web 垫片？**
   - 原生 SDK 在 import 阶段就访问 TurboModuleRegistry；Web 没有该模块，会整个应用崩溃。平台文件在 Web 端降级为 no-op。
6. **401 怎么处理？**
   - 统一拦截 → 清理内存与持久化状态 → 跳登录；不在页面层重复处理。
7. **长列表怎么优化？**
   - FlashList + 分页 + memo/useMemo/useCallback + 避免不必要的全局状态订阅。
8. **表单配置怎么和 PC 端对齐？**
   - 同一套 Schema 与逻辑引擎；静态模式直接消费 FormDesigner 的 true/false 配置。
9. **权限结构不统一怎么办？**
   - 归一化层兼容 8 种字段形态；页面只依赖 `can()` API。
10. **如果重做一次，会改什么？**
    - 更早补测试；把离线队列设计进状态层；打印协议加请求 ID；更早拆分 i18n 命名空间。

---

## 9. 可迁移能力

- React Native 复杂业务 App 的架构与状态设计
- 原生能力集成（BLE / 广播 / 摄像头 / 打印 / 推送）
- 跨端协议对齐与共享契约
- 移动端工程化（EAS、OTA、i18n 门禁、RUM）
- 复杂流程状态机（波次拣货）