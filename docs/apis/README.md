# 协议层接口文档索引

> 协议层接口的「契约源」, PRD 通过 `@api` 锚点引用具体接口。

## 目录结构

```
docs/apis/
├── README.md              # 本文件
├── openapi/               # 主源 OpenAPI JSON (后端导出, 不手改)
│   └── *.json
└── <tag-slug>.md          # 按 tag 拆分的人类可读 + AI 可读索引
```

## 维护规则

1. **主源**: `openapi/*.json` (或 `workspace/api-spec/*.json`) 是事实来源, 由后端协议层导出, **不手改**
2. **生成**: 替换 JSON 后执行 `python3 tools/gen_api_md.py` 重新生成所有 `.md`
3. **业务上下文**: 每个 operation 的 「旧实现」/「迁移备注」段落由人工补充 (注: 当前脚本每次重生成会覆盖, 后续如需保留人工标注, 改造为合并模式)
4. **PRD 引用**: `@api docs/apis/<tag-slug>.md#<operation-id>`

## 当前协议源

### `openapi.json` — 3 个接口 / 1 个 tag

| Tag (业务域) | 接口数 | 索引文件 |
|-------------|--------|----------|
| User | 3 | [user.md](user.md) |
