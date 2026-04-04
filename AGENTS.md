# Obsidian Vault 规则

这是一个 LLM-managed wiki。你是这个知识库的主要编辑者和维护者。

## 目录结构

- `raw/` - 原始素材（PDF摘录、网页剪藏、笔记），**绝对不要修改**
- `wiki/` - LLM 编译生成的结构化文章
- `wiki/index.md` - 文章索引，每次新增/删除文章后必须更新
- `projects/` - 项目文件夹，每个含 overview.md, tasks.md, ideas.md
- `diary/` - 日记，格式 `YYYY-MM-DD [摘要].md`
- `tasks.md` - 全局任务列表

## URL 剪藏

当用户粘贴一个 URL（或多个 URL）时，自动执行：

1. 使用 WebFetch 抓取页面内容，转为干净的 Markdown
2. 保存到 `raw/` 目录，文件名格式：`YYYY-MM-DD-简短英文标题.md`
3. 在文件顶部添加 frontmatter：
   ```yaml
   ---
   url: "原始URL"
   clipped: YYYY-MM-DD
   title: "文章标题"
   ---
   ```
4. 保留原文的完整内容，不要做摘要裁剪
5. 清理无关内容（导航栏、页脚、广告、cookie提示等），只保留正文
6. 保存完成后，自动进入编译流程：生成 wiki 文章并更新索引

如果一次粘贴多个 URL，逐个处理。

## 编译规则

当用户要求处理 raw/ 中的素材时：

1. 读取原始文件，理解核心内容
2. 在 wiki/ 下创建对应文章，文件名使用英文短横线格式（如 `vue-reactivity-system.md`）
3. 文章顶部包含 YAML frontmatter：
   ```yaml
   ---
   tags: [标签1, 标签2]
   created: YYYY-MM-DD
   source: "原始文件路径或URL"
   ---
   ```
4. 使用 `[[wikilink]]` 语法链接到已有的相关文章
5. 更新 `wiki/index.md` 索引

## 文章结构

每篇 wiki 文章应包含：
- 一句话摘要（什么是 X）
- 核心概念（要点列表）
- 详细内容
- 相关链接（指向 wiki 内其他文章的 wikilink）

## 研究与问答

当用户提问时：
1. 先搜索 wiki/ 中的现有文章
2. 综合相关内容回答
3. 如果答案有价值，建议保存为新文章归档

## 健康检查

当用户要求做健康检查时，扫描 wiki/ 所有文章：
- 找出缺少反向链接的孤立文章
- 找出内容重叠可以合并的文章
- 找出需要更新的过时内容
- 检查 index.md 是否与实际文章一致

## 禁止事项

- 不要修改或删除 raw/ 中的原始内容
- 不要创建超过两层嵌套的目录
- 不要在摘要中丢失原文的关键细节
- 不要生成没有实际内容的占位文章
