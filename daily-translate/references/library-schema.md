# 英语学习知识库结构（library-schema）

> 本文件描述 daily-translate 入库目标知识库的结构约定。知识库结构变更时只需更新本文件，无需改动 SKILL.md 主流程。
> 路径一律相对于 config.json 的 `library_path`。

## 目录结构

```
<library_path>/
├── index.html              # 主页面（浏览器查看，含 A-Z 导航 + 搜索 + TTS 发音）
├── generate-content.js     # 构建脚本
└── content/
    ├── a.md ~ z.md         # 单词条目，按首字母分文件
    ├── phrases.md          # 日常用语（中英句对照，按场景分组）
    └── content.js          # 构建产物，禁止手动编辑
```

## 校验依据

`generate-content.js` 与 `content/` 同时存在 = 完整知识库。

## 单词条目格式（content/<首字母>.md）

英文单词按首字母写入对应文件，条目格式：

```markdown
## word
> /音标/ · 词性 · 释义
### 词根拆解
| 部分 | 含义 |
|------|------|
### 同根词串记
### 例句
1. English sentence.
   中文翻译。
```

- 若文件中已有该单词条目，跳过并告知用户。

## 日常用语格式（content/phrases.md）

- 按场景分组：`## 大类`（如 职场沟通 / 日常社交）→ `### 场景名` → 两列表格 `| 中文 | English |`。
- 已有场景分组（截至 2026-09-04）：会议相关、邮件/沟通、开发协作、打招呼、道别、饮食/生活、出行/开车、家庭/带娃、闲聊/八卦、日常感叹。
- 无匹配场景时新建 `### 场景名`，顺带补充 2-3 条相关高频表达。
- 英文输入的句子：反推自然的中文口语填入「中文」列（非逐字直译）。
- 表格单元格内避免使用 `|`。

## 构建命令

```bash
cd <library_path> && node generate-content.js
```

成功标准：输出 `✅ content.js generated`。构建完成后提示用户刷新浏览器。
