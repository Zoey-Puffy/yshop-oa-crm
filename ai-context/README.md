# YShop CRM 项目 AI 知识库（入口）

> 本目录是 YShop CRM 项目的 AI 辅助开发知识库索引。所有内容基于源码与配置分析生成，无法从源码确认的内容统一标注为「待验证」。
> 分析基准：工作区 `d:\workspace\yshop-oa-crm`，后端版本 `2.1.3`。

## 一、使用方式

1. 开始任务前先阅读本文件，按下方索引选择相关文档。
2. 文档中的源码路径用于定向读取，不要为了了解项目而全量扫描仓库。
3. 若文档与源码不一致，以源码为准，并更新对应文档。
4. 本知识库不包含任何密钥、密码、Token，相关配置只记录位置，不记录值。

## 二、文档索引

| 文档 | 内容 |
|------|------|
| [01-project-overview.md](01-project-overview.md) | 项目概览：定位、技术栈、版本、业务范围 |
| [02-project-structure.md](02-project-structure.md) | 目录结构、模块划分、核心文件路径 |
| [03-architecture.md](03-architecture.md) | 系统架构、分层、认证鉴权、多租户、异步任务 |
| [backend/module-list.md](backend/module-list.md) | 后端模块清单（Maven 模块与 CRM 业务包） |
| [backend/api-inventory.md](backend/api-inventory.md) | 后端 CRM 接口清单（路径、方法、Controller、Service） |
| [backend/database-tables.md](backend/database-tables.md) | CRM 数据库表与实体索引、关联关系 |
| [backend/core-business-flow.md](backend/core-business-flow.md) | 线索、客户、商机、合同、回款、发票、审批、支付流程 |
| [frontend/module-list.md](frontend/module-list.md) | 前端模块清单（views、api、路由、权限） |
| [frontend/page-api-mapping.md](frontend/page-api-mapping.md) | 页面与接口对应关系、前后端不一致项 |
| [testing/existing-tests.md](testing/existing-tests.md) | 已有测试情况、测试框架与规范 |
| [development/startup-guide.md](development/startup-guide.md) | 本地启动方式（后端、前端、依赖服务） |

## 三、关键入口速查

| 需求 | 优先查看 |
|------|----------|
| 新增/修改 CRM 接口 | `yshop-crm/yshop-module-crm/yshop-module-crm-biz/src/main/java/co/yixiang/yshop/module/crm/controller/admin/` |
| 修改 CRM 业务逻辑 | `.../module/crm/service/` 下对应子包的 `*ServiceImpl.java` |
| 修改 CRM 数据访问 | `.../module/crm/dal/mysql/` 与 `.../dal/dataobject/` |
| 修改 CRM 枚举/错误码 | `yshop-crm/yshop-module-crm/yshop-module-crm-api/src/main/java/co/yixiang/yshop/module/crm/enums/` |
| 修改前端 CRM 页面 | `yshop-crm-vue/src/views/crm/` |
| 修改前端 CRM API 调用 | `yshop-crm-vue/src/api/crm/` |
| 数据库结构 | `yshop-crm/sql/yshop-crm-open-source.sql`，增量脚本在 `yshop-crm/sql/migration/` |

## 四、已有规则与记忆文件（只读参考）

- 根目录 `.clinerules/`：全局与后端开发规则（`00-global.md`、`01-backend.md`）。
- 根目录 `ai-context/memory-bank/`：开发记忆文件（`activeContext.md`、`decisions.md`、`progress.md`），当前内容基本为空模板，待项目组补充。

## 五、本次分析范围与限制

- 本次为「只读分析 + 文档创建」，未修改任何业务源码。
- 分析重点是 CRM 业务模块（`yshop-module-crm`）及其依赖的前端页面。其它模块（system、infra、mall、pay、mp、message）仅记录与 CRM 的依赖关系，未逐一分析。
- 运行期行为（如真实数据、定时任务执行结果、支付回调联调结果）未验证，相关内容标注为「待验证」。
