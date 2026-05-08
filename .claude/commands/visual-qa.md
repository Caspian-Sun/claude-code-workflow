---
description: Visual QA — 截取当前应用截图，与设计稿像素对比，输出 P0/P1/P2 报告（支持 Web / Android / 鸿蒙）
argument-hint: [<reference_image> [web|android|ohos|auto] [device_id]]
allowed-tools: Bash, Read, Edit, Glob, Grep
helper: true
---

你现在是视觉 QA 工程师。执行截图对比，输出结构化的差异报告。

## 执行步骤

### 第一步: 运行对比脚本

```bash
bash docs/designs/screenshots/run-visual-qa.sh $ARGUMENTS
```

脚本会自动完成：
- **平台检测**：`auto` 模式下，检测到 adb/hdc 设备 → 移动端截图；否则 → Web Playwright 截图
- **Web**：检测 dev server 是否在 port 8000 运行 → 直接截图；未运行 → 自动 `pnpm dev` 启动
- **Android**：`adb exec-out screencap -p` 截图
- **鸿蒙**：`hdc shell snapshot_display -f xxx.jpeg`（注意：必须 .jpeg，.png 会报错）→ 接收后转 .png
- ImageMagick 像素对比，生成差值图 + 左右拼接对比图
- 结果写入 `docs/designs/screenshots/actual/<分类>/<文件名>/qa-result.json`

### 第二步: 读取结果并视觉分析

1. 读取量化指标（路径跟随 reference 目录结构镜像生成）：
   ```bash
   # 示例：reference/主功能/01_home.jpg 的结果在：
   cat docs/designs/screenshots/actual/主功能/01_home/qa-result.json
   ```
2. 用 Read 工具读取以下图像做视觉判断：
   - `docs/designs/screenshots/actual/<分类>/<名称>/side-by-side.png`（左右拼接对比图）
   - `docs/designs/screenshots/actual/<分类>/<名称>/diff.png`（红色=差异区域）

### 第三步: 输出报告

严格按以下格式输出：

```
📸 Visual QA 报告
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
平台: web (localhost:8000) | android (设备ID) | ohos (设备ID)
参考: docs/designs/screenshots/reference/<分类>/xx.png
实际: docs/designs/screenshots/actual/<分类>/<名称>/actual.png

P0 结构 (必须通过)
  ✅/❌ 整体布局       RMSE=x.xxx

P1 视觉 Token (≥ 90% 相似)
  ✅/⚠️  整体相似度    RMSE=x.xxx
  差异项:
    - [ ] 区域/元素: 期望 xx, 实际 xx

P2 像素级 (≥ 95% 相似, 不阻塞)
  ✅/📌 精确度        RMSE=x.xxx

产出文件:
  差值图:  docs/designs/screenshots/actual/<分类>/<名称>/diff.png
  对比图:  docs/designs/screenshots/actual/<分类>/<名称>/side-by-side.png
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
结论: ✅ 通过 | ❌ P0 失败，必须修复后重跑 | ⚠️ P1 有差异，建议修复
下一步: /code --only <taskId> 修复差异 | 或确认接受差异继续
```

### 第四步: 上游溯源 — 确定根因层级

差异不一定是代码写错了。先做根因分类，再决定修哪一层。**直接改代码是最后一步，不是第一步。**

```
diff.png 有差异
    │
    ▼
【Q1】这个 UI 区域在实际截图里完全不存在吗？
    │
    ├─ 是（整块红 / 区域缺失）
    │       │
    │       ▼
    │   【Q2】对应的 tasks.json 里有这个组件的任务吗？
    │       ├─ 没有任务 → 根因: PRD 遗漏  → 走「路径 A: 回修 PRD」
    │       ├─ 有任务但 status=pending → 根因: 任务未执行 → 走「路径 B: 补跑 /code」
    │       └─ 有任务且 status=done → 根因: 代码逻辑错误 → 走「路径 C: 修代码」
    │
    └─ 否（区域存在但样式偏差）
            │
            ▼
        【Q3】差异区域对应任务的 designRef 字段是空的吗？
            ├─ 是（designRef 为空）→ 根因: 计划缺设计引用 → 走「路径 B+: 补 designRef 后重跑」
            └─ 否（有 designRef）
                    │
                    ▼
                【Q4】PRD 里这个功能点的业务规则有视觉约束吗？（颜色/尺寸/间距）
                    ├─ 没有视觉规则 → 根因: PRD 规则不完整 → 走「路径 A+: 补 PRD 规则」
                    └─ 有规则但代码没遵守 → 根因: 实现偏差 → 走「路径 C: 修代码」
```

---

> **精准修复原则：只改有问题的那一处，不全量重跑上游。** 每条路径都给出最小改动范围。

#### 路径 A: 回修 PRD（根因在需求层）

**情况 A1 — PRD 完全缺少某功能章节（需要新建任务）**

```
最小改动:
  1. 只在 PRD 里新增缺失的 ## 功能章节 + 业务规则（不动其他章节）
  2. /prd-check → 确认通过
  3. 手动在 tasks.json 末尾追加新任务对象（不要重跑 /plan 覆盖整个文件）:
       - 给新任务分配下一个 taskId（如 T099）
       - prdRef 指向新增的章节锚点
       - designRef 指向 "图片路径#区域名"
       - dependencies 填写它依赖的已完成任务 ID
  4. /code @docs/tasks/tasks-xxx.json --only T099
  5. /visual-qa 验证
```

**情况 A2 — PRD 有章节但缺少视觉约束规则（尺寸/颜色/间距）**

```
最小改动:
  1. 只在 PRD 对应 ### 业务规则 里追加视觉约束条目（不改其他规则）
  2. /prd-check → 确认通过
  3. 在 tasks.json 找到对应任务，把新规则追加进 businessRules 数组（不重跑 /plan）
  4. /code @docs/tasks/tasks-xxx.json --only <受影响 taskId>
  5. /visual-qa 验证
```

---

#### 路径 B: 补跑 /code（根因在执行层）

**情况 B1 — 任务未执行（status=pending）**

```
最小改动:
  /code @docs/tasks/tasks-xxx.json --only <taskId>
  （只跑这一个任务，其余 done 的任务不重做）
```

**情况 B2 — designRef 为空，开发时没有视觉参考**

```
最小改动:
  1. 用 Edit 直接改 tasks.json 里对应任务的 designRef 字段:
       "designRef": "docs/designs/screenshots/reference/<分类>/xx.png#区域名"
     （只改这一个字段，不动其他任务）
  2. /code @docs/tasks/tasks-xxx.json --only <taskId>
  3. /visual-qa 验证
```

---

#### 路径 C: 修代码（根因在实现层）

**只改出问题的那个文件，不碰其他文件。**

```
最小改动:
  1. 读 diff.png 定位红色区域 → 找到唯一的责任文件
  2. 读 side-by-side.png 肉眼对比，提取期望值
  3. Grep 找到对应文件（Web：*.less / *.module.css；移动端：*.dart）
  4. 只用 Edit 修改该文件里出问题的那几行
  5. /visual-qa 验证，最多 3 轮
```

> ❌ 不要因为改了一处就顺手"优化"周边代码 — 只改导致差异的那一处

---

#### 接受差异（不修复）

| 差异类型 | 原因 | 处置 |
|---------|------|------|
| 空列表 vs 设计稿有数据 | 数据内容差异，预期行为 | 报告注明，不修复 |
| 字体渲染微差（P2） | 平台抗锯齿 vs 设计工具 | P2 接受 |
| 系统状态栏（时间/信号） | 内容动态变化 | P2 接受 |
| 3 轮修复后仍未收敛 | 根因复杂，超出自动修复范围 | 停下告知用户人工介入 |
| 设计稿本身与 PRD 矛盾 | 设计稿未更新 | 让用户决定以哪个为准 |

## 注意事项

### 通用
- **RMSE 阈值仅供参考**：动态内容/未登录状态会拉高 RMSE，需结合 side-by-side.png 视觉判断
- **参考图放哪**：按功能分类放入 `docs/designs/screenshots/reference/<分类>/` 目录
- **产出自动分类**：每张设计稿对应独立子目录，多屏同跑不互相覆盖

### Web 模式
- dev server 端口默认 8000（UmiJS），如项目用其他端口修改脚本里的 `PORT` 变量

### 移动端（Android / 鸿蒙）
- **真机工作流**：先手动操作 App 到目标页面，再触发 `/visual-qa` 截取当前屏幕
- **多设备**：`adb devices` / `hdc list targets` 查询 ID，作为第三个参数传入
- **鸿蒙截图**：`snapshot_display` 只接受 `.jpeg` 扩展名，传 `.png` 会生成空文件（脚本已处理）
- **HDC 路径**：需 DevEco Studio 已安装，脚本自动注入 PATH

$ARGUMENTS
