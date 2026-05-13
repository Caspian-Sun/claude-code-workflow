# [模块名] PRD

> 写 PRD 的核心原则: **小标题即锚点**, 后续 `@prd docs/prds/xxx.md#<锚点>` 全靠这些标题定位。
> 所以小标题命名要稳定、明确, 不要随意改动。

## 元信息

| 项 | 值 |
|----|----|
| 模块代号 | `user-list` (英文, 与文件名一致) |
| 负责人 | 张三 |
| 创建日期 | 2026-04-14 |
| 最后更新 | 2026-04-14 |
| 状态 | draft / reviewing / approved / shipped |
| 需求来源 | 全新业务 / 旧版本迁移 / 内部重构 (迁移类需求补充下方「旧版本 → 新版本迁移映射」段) |

## 背景与目标

> 为什么做这个需求? 解决什么问题? 目标用户是谁?
> 一段话说清楚, 不要超过 200 字。

## 名词解释

> 仅当模块涉及业务术语时填写。让 AI 和新人不会理解错。

| 术语 | 含义 |
|------|------|
| 配额 | 用户每月可用的接口调用次数上限 |

## 设计稿

> 设计稿是视觉规范的唯一来源, 与 PRD 的业务规则互补: PRD 管「做什么」, 设计稿管「长什么样」。
> 三种来源方式可并存, 有什么填什么, 留空不阻塞。

| 项 | 值 |
|----|----|
| 来源类型 | link / file / mcp (可多选) |
| Figma 链接 | `https://figma.com/file/xxx` (无则留空) |
| 本地文件 | `docs/designs/xxx.png` 或 `docs/designs/xxx.sketch` (无则留空) |
| MCP 配置 | figma-mcp 已接入 / 未接入 |

### 功能点与设计帧映射

> 每个功能点对应设计稿的哪一帧 / 哪一页, 便于 `/plan` 拆任务和 `/code` 编码时对照。

| 功能点 (PRD 锚点) | 设计引用 |
|-------------------|---------|
| #搜索表单 | Figma: `<URL>#Frame-SearchForm` 或 `docs/designs/search-form.png` |
| #数据列表 | Figma: `<URL>#Frame-DataTable` |

> **注意**:
> - Figma 链接优先 (永远指向最新版), 本地文件会过期
> - MCP 接入后, `/code` 阶段可实时从 Figma 提取 Design Token, 不需手动导出
> - 切图 / 图标等代码直接消费的资源, 导出到 `workspace/public/images/`

---

## 功能点 1: [小标题作为锚点]

> 每个功能点一个二级标题, 标题就是锚点。
> 例如 `## 搜索表单` → `@prd docs/prds/user-list.md#搜索表单`

### 信息源对照

> 写规则前先把三源摆齐, 避免漏。任何一源缺失都要标注出来 (推动设计/协议层补)。
> 这一小段是 PRD 编写者的「原料齐全检查」, 评审时也优先看这张表。

| 类型 | 引用 |
|------|------|
| 📜 旧版本实现 (迁移类需求) | `https://legacy.example.com/xxx` 或 `docs/designs/legacy-snapshots/xxx.png` (全新业务可填「-」) |
| 🎨 新设计稿 | Figma: `<URL>#Frame-Xxx` 或 `docs/designs/<xxx>.png` |
| 🔌 协议接口 | `@api docs/apis/<tag>.md#<op-id>` (多个用逗号; 无接口的纯前端功能填「-」) |
| 🆕 是否新增功能 | 否 (旧版本已有) / 是 (新增功能, 旧版本无对应) |
| 📝 备注 | 缺源说明, 如「旧版有但行为有 bug, 本期按新设计稿走」 |

### 用户故事

> 作为 [角色], 我希望 [功能], 以便 [价值]。

### 字段定义 (如有表单/数据)

| 字段 | 类型 | 必填 | 校验规则 | 默认值 |
|------|------|------|---------|--------|
| 手机号 | string | 否 | 11 位, 1 开头, 第二位 3-9 | - |
| 状态 | enum | 否 | 启用 / 禁用 / 全部 | 全部 |

### 业务规则 (重要, 这是测试断言的来源)

> 每条规则必须可测试, 用「当...时, 应...」句式更清晰。
> 避免技术实现描述, 只写业务语义。

1. 手机号格式不合法时, 表单实时显示错误提示, 搜索按钮禁用
2. 所有字段为空时, 搜索按钮禁用
3. 重置按钮清空字段后, 自动触发一次查询 (不用用户再点搜索)

### 数据契约 (引用协议层 OpenAPI)

> **字段细节以协议层 OpenAPI 为准** — 主源放在 `docs/apis/openapi/*.json` (推荐) 或 `workspace/api-spec/openapi.json` (向后兼容)。
> `tools/gen_api_md.py` 自动生成 `docs/apis/<tag>.md` 索引层, PRD 通过 `@api docs/apis/<tag>.md#<op-id>` 锚点引用。
> 接口字段类型 / 必填 / 枚举值在 OpenAPI 一处定义, **不在 PRD 重复抄写**; 前端 `pnpm gen:api` 自动生成 `workspace/src/types/api.ts`。

#### 调用的接口

> 用 `@api docs/apis/<tag>.md#<operation-id>` 锚点引用, 格式与代码 JSDoc 的 `@api` 字段保持一致。

| 业务操作 | 接口锚点 | 方法 | 状态 |
|---------|---------|------|------|
| 搜索用户 | `@api docs/apis/user.md#searchusers` | GET | ✅ 已存在 |
| 查看详情 | `@api docs/apis/user.md#getuserbyid` | GET | ✅ 已存在 |
| 导出 Excel | `@api docs/apis/user.md#exportusers` | POST | 🆕 待协议层实现 (见下方接口提议) |

> **状态字段**:
> - ✅ 已存在: 协议层 OpenAPI 已定义, 锚点可点开查看字段
> - 🆕 待协议层实现: PRD 先写接口提议, 评审后由后端加入 `docs/apis/openapi/*.json` 并重跑脚本

#### 接口提议 (仅当有 🆕 接口时填写)

> PRD 提议新接口 → 协议层评审 → 后端实现 → 更新 `docs/apis/openapi/*.json` (或 `workspace/api-spec/openapi.json`) → 跑 `python3 tools/gen_api_md.py` 重新生成索引层。
> 应急情况下也可以放入 `workspace/api-spec/openapi.local.json` 前端先开发, 后端就绪后合入主源并移除 local 文件。

```yaml
# 示例: 导出用户列表接口
paths:
  /api/users/export:
    post:
      operationId: exportUsers
      summary: 导出符合筛选条件的用户为 Excel
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                phone: { type: string }
                status: { type: string, enum: [enabled, disabled] }
      responses:
        '200':
          description: Excel 文件流
          content:
            application/octet-stream:
              schema: { type: string, format: binary }
```

> ⚠️ 评审通过后记得把状态从 🆕 改为 ✅, 避免前端以为还要等后端。

#### 错误码映射 (业务侧定义)

> OpenAPI 只定义错误码存在, **如何处理是业务决策**, 必须在这里写。

| code | 含义 | 前端处理 |
|------|------|---------|
| 0 | 成功 | - |
| 40001 | 参数非法 | 表单 inline 错误提示 |
| 40301 | 无权限 | 跳 `/403` |
| 50001 | 服务异常 | 自动重试 1 次, 仍失败弹 toast |

#### Mock 数据约定

- 后端未就绪期间, 在 `workspace/mock/` 写假数据, **必须 import 生成的类型** 保证结构对齐:
  ```typescript
  import type { paths } from '@/types/api';
  type SearchResp = paths['/api/users/search']['get']['responses']['200']['content']['application/json'];
  ```
- 联调时通过 `workspace/config/proxy.ts` 切换到真实后端
- 任何字段变更不要手改 mock, 先推后端更新 OpenAPI, 拉取新 json 后让 TS 编译告诉你哪里要改

#### DTO/Type 与 OpenAPI Schema 的对齐

- 业务 api 层入参/出参 type 文件头注释加 `@api docs/apis/<tag>.md#schema-<schema-name>` 指向 Schema 锚点
- 协议层 Schema 变更 → 重跑 `tools/gen_api_md.py` + `pnpm gen:api` → 对照锚点检查业务 type
- **禁止**在 api 层之外的层手写 JSON 字段名字符串

### 交互流程

> 文字描述或简单流程图。

```
用户输入手机号 → 实时校验 → 通过则启用搜索按钮 → 点击搜索 → loading → 列表更新
```

### 异常场景

| 场景 | 预期行为 |
|------|---------|
| 接口超时 | 显示 `加载失败, 请重试`, 提供「重试」按钮 |
| 无权限 | 跳转 `/403` |
| 数据为空 | 显示 antd `Empty` 组件 |

---

## 功能点 2: [下一个小标题]

(同上结构)

---

## 旧版本 → 新版本迁移映射 (仅迁移类需求填写)

> 当「需求来源」为「旧版本迁移」时填写此段, 全新业务可跳过。
> 适用场景示例: H5 → React 重构 / jQuery 老站 → Umi 迁移 / 老 Vue → 新 Vue 升级 / Web → App 跨端。
> 目的: 给 `/plan` 拆任务提供清单, 给 `/code` 编码提供"对照实现"参照, 给 `/review` 验证"旧版本行为是否完整复刻"。

### 页面/路由对照表

> 「接口来源」列说明本接口的状态:
> - ✅ 已存在: 协议层 OpenAPI 已定义, 锚点可点开
> - 🆕 提议: 前端基于 PRD 提议的新接口, 待协议层评审 + 后端实现 (见「数据契约 → 接口提议」段)
> - 🅽 新版独有: 该行的新版模块旧版没有 (即新增功能, 顺带也用本接口)

| 旧版页面/路由 | 新版模块/路由 | 调用的协议接口 | 接口来源 | 行为差异 / 备注 |
|------------|--------------|--------------|---------|----------------|
| `/legacy/profile` | `features/user/profile` → `/profile` | `@api docs/apis/user.md#getuserinfo`, `@api docs/apis/user.md#updateuser` | ✅ 已存在 | 头像上传改用 antd Upload; 昵称校验规则保留 |
| `/legacy/security/email` | `features/user/components/EmailBindForm` → `/profile/email` | `@api docs/apis/security.md#sendemailcaptcha`, `@api docs/apis/security.md#bindemail` | ✅ 已存在 | 验证码倒计时 60s 与旧版一致 |
| _(旧版无对应页)_ | `features/dashboard/widgets/RealTime` → `/dashboard` | `@api docs/apis/metrics.md#getrealtimemetrics` | 🅽 新版独有 | - |
| _(旧版无对应页)_ | `features/export` | `@api docs/apis/export.md#exportbatch` | 🆕 提议 (协议层待加 `exportBatch`) | 评审后补到 `docs/apis/openapi/*.json` |

### 迁移范围 (in-scope / out-of-scope)

**本期迁移 (旧版已有 → 新版实现)**:
- [ ] 旧版页面 A → 新版模块 X
- [ ] 旧版页面 B → 新版模块 Y

**本期新增 (新版独有, 旧版不动)**:
- [ ] 新版模块 Z (功能描述, 为什么旧版不做)

**本期不迁移 (旧版有但新版不做)**:
- 旧版页面 C — 原因: 依赖旧栈特有 API, 新栈无对应实现, 留下期处理
- 旧版页面 D — 原因: 业务侧确认下线

### 行为差异统一约定

> 旧版 → 新版不是 1:1 复刻, 列出本模块共性的差异。

| 旧版行为 | 新版行为 | 原因 |
|--------|--------|------|
| jQuery alert 弹窗 | antd Modal | 与设计稿/组件库一致 |
| 全局 toast | antd notification | 区分级别 (success/warning/error) |
| 同步刷新页面 | useRequest + 局部 loading | SPA 体验 |
| 表单校验在 submit 时 | TextInput + onBlur + 实时校验 | UX 升级 |
| `localStorage` 缓存 | Zustand + 持久化插件 / IndexedDB | 状态管理统一 |

### 数据兼容性

- 用户登录态: 旧版 token 是否需要复用? 是 → 在哪个接口换取新版 session
- 本地缓存迁移: 旧版的 `localStorage` 数据是否需要迁移到新版? 一般不需要 (除非要保留用户偏好)
- 后端字段是否对新版改造? 一般不应改, 如确需改造在「接口提议」段说明

---

## 验收清单 (可选)

> 上线前的整体验收点, 跨多个功能点的集成性要求。

- [ ] 整个模块在 Chrome / Safari / 移动端 H5 三端正常显示
- [ ] 所有接口都有 loading 和 error 处理
- [ ] 国际化文案齐全 (中/英)
- [ ] (迁移类) 页面/路由对照表中所有"本期迁移"项已实现并与旧版行为对齐

## 变更记录

| 日期 | 变更内容 | 变更人 |
|------|---------|--------|
| 2026-04-14 | 初版 | 张三 |
