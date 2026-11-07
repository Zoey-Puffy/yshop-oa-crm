# 当前开发状态

AI 知识库已建立，12 份文档已就绪。

## 当前模块

- 知识库位于 `ai-context/` 目录
- 入口文件：`ai-context/README.md`

## 已完成

1. AI 知识库 12 份文档全部创建完成
   - `01-project-overview.md` — 项目概览
   - `02-project-structure.md` — 目录结构与模块划分
   - `03-architecture.md` — 系统架构与设计约定
   - `backend/module-list.md` — 后端模块清单
   - `backend/api-inventory.md` — CRM 接口清单（章节顺序已修正）
   - `backend/database-tables.md` — 数据库表与实体索引
   - `backend/core-business-flow.md` — 核心业务流程
   - `frontend/module-list.md` — 前端模块清单
   - `frontend/page-api-mapping.md` — 页面-接口对应关系
   - `testing/existing-tests.md` — 现有测试情况
   - `development/startup-guide.md` — 启动指南
   - `README.md` — 知识库入口与索引
2. 接口清单章节顺序已修复（§10 在线支付已移到 §11 之前）
3. README 中错误路径 `ai-content/memory-bank/` 已修正为 `ai-context/memory-bank/`
4. 已确认未修改任何业务源码
5. Git 工作区干净（知识库文件已提交）

## 正在进行

无

## 下一步

后续开发任务请按以下流程执行：
1. 优先读取 `ai-context/README.md` 了解知识库结构
2. 根据文档索引定向读取相关文档
3. 若文档与源码不一致，以源码为准并更新对应文档
4. 运行期行为（如支付回调、定时任务执行结果）尚未验证

## 注意事项

- 知识库不包含任何密钥、密码、Token，仅记录配置位置
- 文档中标注「待验证」的内容需在实际运行时确认
- 前端部分页面带 `2`、`3` 后缀（如 `index2.vue`、`Form2.vue`）是否仍在菜单中使用，待验证
- Dockerfile 使用 JRE 21，本地开发为 JDK 17，需留意差异
- `application-local.yaml` 中 `yshop.file-path` 为 macOS 路径，Windows 环境需修改

## 更新频率

每次任务结束
尽量控制在一页以内