# 01 项目概览

## 1. 基本信息

| 项 | 内容 | 来源 |
|----|------|------|
| 项目名称 | YSHOP 意象 CRM（后端 artifactId 为 `yshop`，前端 `name` 为 `yshop-crm-vue3`） | `yshop-crm/pom.xml`、`yshop-crm-vue/package.json` |
| 定位 | 支持企业微信的前后端分离 CRM，覆盖「线索 → 客户 → 商机 → 合同 → 回款 → 项目」销售全流程 | `README.md` |
| 后端版本 | `2.1.3`（`revision`） | `yshop-crm/pom.xml` |
| 前端版本 | `2.1.3` | `yshop-crm-vue/package.json` |
| 开源协议 | MIT | `README.md`、`LICENSE` |
| 后端 Java | 17 | `yshop-crm/pom.xml` |
| Spring Boot | 3.2.2 | `yshop-crm/pom.xml` |
| 后端端口 | 48081（`application-local.yaml` / `application-dev.yaml`） | `yshop-crm/yshop-server/src/main/resources/` |
| 前端开发端口 | `.env` 中 `VITE_PORT=80`（Vite `server.port` 读取该值） | `yshop-crm-vue/.env`、`vite.config.ts` |

## 2. 技术栈

### 后端

| 类别 | 技术 | 版本/说明 | 来源 |
|------|------|-----------|------|
| 基础框架 | Spring Boot | 3.2.2 | `yshop-crm/pom.xml` |
| 安全 | Spring Security + OAuth2（框架内 `yshop-spring-boot-starter-security`） | 版本由 starter 决定 | `yshop-crm/yshop-framework/` |
| ORM | MyBatis-Plus（含 mybatis-plus-join） | 版本见 `yshop-dependencies` | `yshop-crm/yshop-dependencies/pom.xml` |
| 缓存/锁 | Redis（Spring Data Redis + Redisson 分布式锁） | — | `application.yaml`、`CrmContractServiceImpl` |
| 定时任务 | Quartz（JDBC JobStore）+ 框架 `yshop-spring-boot-starter-job` | — | `application-dev.yaml`、`job/*.java` |
| 消息队列 | 框架 `yshop-spring-boot-starter-mq`（Redis Stream 消费者） | — | `mq/consumer/PayNoticeConsumer.java` |
| 支付 | `pay-java-parent`（`com.egzosn.pay`）+ 支付模块 `yshop-module-pay` | — | `PayServiceImpl.java`、`PayController.java` |
| 接口文档 | springdoc + knife4j（Swagger UI 路径 `/swagger-ui`） | — | `application.yaml` |
| 工具 | Lombok、Hutool、MapStruct | `lombok 1.18.30`、`mapstruct 1.5.5.Final` | `yshop-crm/pom.xml` |
| 测试 | JUnit 5 + Mockito（`mockito-inline 5.2.0`）+ H2 内存库 | — | `yshop-dependencies/pom.xml`、`yshop-spring-boot-starter-test` |
| 企业微信/钉钉/云呼 | 依赖 `alibaba-dingtalk-service-sdk`、`dingtalk`；企业微信/云呼源码 | 待验证（见下） | `yshop-module-crm-biz/pom.xml` |

### 前端

| 类别 | 技术 | 版本 | 来源 |
|------|------|------|------|
| 框架 | Vue | 3.4.21 | `yshop-crm-vue/package.json` |
| 构建 | Vite | 5.1.4 | `package.json` |
| UI | Element Plus | 2.6.1 | `package.json` |
| 状态管理 | Pinia（含 persistedstate 插件） | `^2.1.7` | `package.json` |
| 路由 | vue-router | `^4.3.0`（`createWebHistory`） | `src/router/index.ts` |
| HTTP | axios（封装在 `src/config/axios`） | `^1.6.8` | `package.json`、`src/config/axios/config.ts` |
| 国际化 | vue-i18n | 9.10.2 | `package.json` |
| 样式 | UnoCSS、Sass | `0.58.5`、`1.69.5` | `package.json` |
| 工作流设计 | bpmn-js（仅 devDependencies，用于系统流程设计器） | `8.9.0` | `package.json` |
| 表单设计 | @form-create/designer | `^3.1.3` | `package.json` |
| 包管理 | pnpm（`engines.pnpm >= 8.6.0`） | — | `package.json` |
| Node | `>= 16.0.0` | — | `package.json` |

## 3. 业务范围

来自 `README.md` 功能列表与 `yshop-crm/yshop-module-crm` 实际包结构，CRM 核心业务包括：

| 业务域 | 后端包（`controller/admin/` 下） | 说明 |
|--------|----------------------------------|------|
| 线索 | `crmclues` | 线索管理、公海领取/放入公海 |
| 客户 | `crmcustomer`、`crmcustomercontacts` | 客户管理、公海、联系人、发短信/邮件 |
| 商机 | `crmbusiness` | 商机与商机产品 |
| 合同 | `crmcontract` | 合同与合同产品、合同审核 |
| 回款 | `crmcontractreceivables` | 回款计划/记录与审核、在线支付 |
| 发票 | `crminvoice` | 开票申请、审核、开票 |
| 审批流程 | `crmflow`、`crmflowlog` | 审批流程模板、审批记录 |
| 跟进记录 | `crmrecord` | 客户/商机/线索跟进 |
| 业绩 | `crmachievement`、`crmanalysis` | 业绩目标、销售分析 |
| 首页统计 | `crmindex` | 首页 CRM 统计 |
| 操作日志 | `crmoperatelog` | CRM 操作日志 |
| 支付 | `crmpay` | 回款在线支付与回调 |

`README.md` 中提到的「企微、第三方（云呼、钉钉）、公众号、商城」等功能：
- 企微、云呼的 CRM 业务源码在当前 CRM 模块中**未找到对应实现**（待验证，可能已移除或在其它模块）。
- 公众号、商城位于 `yshop-module-mp`、`yshop-module-mall`，与 CRM 的依赖关系见 `03-architecture.md`。

## 4. 多模块关系概览

- 后端为 Maven 多模块：框架层 `yshop-framework`，业务层 `yshop-module-*`，启动容器 `yshop-server`。
- 前端为单体 Vue 工程，通过 `/admin-api` 前缀访问后端。
- 详见 [02-project-structure.md](02-project-structure.md) 与 [03-architecture.md](03-architecture.md)。

## 5. 待验证项

- 企微（WorkWx）与云呼（CloudCall）是否仍有源码：当前 CRM 模块内未检索到实现，前端 `api/` 下也无对应调用，需与项目负责人确认。
- README 标注「项目」「任务管理」「公海/二开」等能力：CRM 模块内未检索到项目/任务相关包，待验证。
- `README.md` 的「第三方：云呼中心、企业钉钉」与 `yshop-module-crm-biz/pom.xml` 中存在钉钉 SDK 依赖，但 CRM 源码中未找到调用，待验证。
