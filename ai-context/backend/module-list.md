# 后端模块清单

> 根路径：`yshop-crm/`。以下为 Maven 模块与 CRM 业务包的索引，所有路径均已在源码中确认。

## 1. Maven 模块（父 POM：`yshop-crm/pom.xml`）

| 模块 | 类型 | 说明 | 主要路径 |
|------|------|------|----------|
| `yshop-dependencies` | BOM | 统一依赖版本 | `yshop-crm/yshop-dependencies/pom.xml` |
| `yshop-framework` | 框架聚合 | 14 个 starter + `yshop-common` | `yshop-crm/yshop-framework/` |
| `yshop-server` | 启动容器 | `YshopServerApplication`，聚合业务模块 | `yshop-crm/yshop-server/` |
| `yshop-module-system` | 业务 | 用户、角色、菜单、部门、岗位、租户、字典、OAuth2、短信、邮件、站内信、操作日志 | `yshop-crm/yshop-module-system/yshop-module-system-biz/` |
| `yshop-module-infra` | 业务 | 代码生成、文件、定时任务（Job）、API 日志、Redis 监控 | `yshop-crm/yshop-module-infra/yshop-module-infra-biz/` |
| `yshop-module-pay` | 业务 | 支付商户配置（`merchant_details`）等 | `yshop-crm/yshop-module-pay/yshop-module-pay-biz/` |
| `yshop-module-mp` | 业务 | 微信公众号 | `yshop-crm/yshop-module-mp/yshop-module-mp-biz/` |
| `yshop-module-mall` | 业务聚合 | 商城：`yshop-module-product-biz`、`yshop-module-shop-biz` 等 | `yshop-crm/yshop-module-mall/` |
| `yshop-module-message` | 业务 | 站内信、微信模板消息 | `yshop-crm/yshop-module-message/yshop-module-message-biz/` |
| **`yshop-module-crm`** | **业务（CRM）** | 本知识库重点 | `yshop-crm/yshop-module-crm/` |
| ├─ `yshop-module-crm-api` | API | 枚举、错误码（`enums` 包） | `yshop-crm/yshop-module-crm/yshop-module-crm-api/` |
| └─ `yshop-module-crm-biz` | 实现 | Controller / Service / DAL / Job / MQ | `yshop-crm/yshop-module-crm/yshop-module-crm-biz/` |

## 2. 框架 starter（与 CRM 直接相关的部分）

| starter | 在 CRM 中的用途 |
|---------|-----------------|
| `yshop-spring-boot-starter-security` | `SecurityFrameworkUtils`、`@PreAuthenticated`、`@PreAuthorize` 的 `@ss` 鉴权 |
| `yshop-spring-boot-starter-mybatis` | `BaseMapperX`、`LambdaQueryWrapperX` |
| `yshop-spring-boot-starter-redis` | `RedisUtil`、`RedisKeyConstants` |
| `yshop-spring-boot-starter-mq` | `AbstractRedisStreamMessageListener` |
| `yshop-spring-boot-starter-job` | `JobHandler` 接口（Quartz 任务） |
| `yshop-spring-boot-starter-excel` | `ExcelUtils.write`（导出） |
| `yshop-spring-boot-starter-biz-tenant` | `@TenantIgnore` |
| `yshop-spring-boot-starter-biz-data-permission` | 数据权限框架（CRM 未直接使用注解，待验证） |
| `yshop-spring-boot-starter-test` | 单元测试基类（`BaseDbUnitTest` 等） |

## 3. CRM 业务包（`yshop-module-crm-biz`）

包根：`co.yixiang.yshop.module.crm`

| 业务域 | Controller 包（`controller.admin.*`） | Service 包（`service.*`） | DO 包（`dal.dataobject.*`） | Mapper 包（`dal.mysql.*`） | 对应数据表 |
|--------|----------------------------------------|---------------------------|-----------------------------|----------------------------|------------|
| 线索 | `crmclues` | `crmclues` | `crmclues` | `crmclues` | `yshop_crm_clues` |
| 客户 | `crmcustomer` | `crmcustomer` | `crmcustomer` | `crmcustomer` | `yshop_crm_customer` |
| 联系人 | `crmcustomercontacts` | `crmcustomercontacts` | `crmcustomercontacts` | `crmcustomercontacts` | `yshop_crm_customer_contacts` |
| 商机 | `crmbusiness` | `crmbusiness` | `crmbusiness`（含 `CrmBusinessProductDO`） | `crmbusiness` | `yshop_crm_business`、`yshop_crm_business_product` |
| 合同 | `crmcontract` | `crmcontract` | `crmcontract`（含 `CrmContractProductDO`） | `crmcontract` | `yshop_crm_contract`、`yshop_crm_contract_product` |
| 回款 | `crmcontractreceivables` | `crmcontractreceivables` | `crmcontractreceivables` | `crmcontractreceivables` | `yshop_crm_contract_receivables` |
| 发票 | `crminvoice` | `crminvoice` | `crminvoice` | `crminvoice` | `yshop_crm_invoice` |
| 审批流程 | `crmflow` | `crmflow` | `crmflow`（含 `CrmFlowStepDO`） | `crmflow` | `yshop_crm_flow`、`yshop_crm_flow_step` |
| 审批记录 | `crmflowlog` | `crmflowlog` | `crmflowlog` | `crmflowlog` | `yshop_crm_flow_log` |
| 跟进记录 | `crmrecord` | `crmrecord` | `crmrecord` | `crmrecord` | `yshop_crm_record` |
| 操作日志 | `crmoperatelog` | `crmoperatelog` | `crmoperatelog` | `crmoperatelog` | `yshop_crm_operatelog` |
| 业绩目标 | `crmachievement` | `crmachievement` | `crmachievement` | `crmachievement` | `yshop_crm_achievement` |
| 销售分析 | `crmanalysis` | `crmanalysis` | 无独立 DO（复用其它域） | — | — |
| 首页统计 | `crmindex` | `crmindex` | 无独立 DO | — | — |
| 在线支付 | `crmpay` | `crmpay` | 无独立 DO（复用回款、合同） | — | — |
| 站内消息 | — | — | `crmmessage` | `crmmessage` | `yshop_crm_message` |

> `crmmessage` 没有 Controller，只在审批、跟进提醒中被写入或读取（`CrmFlowServiceImpl`、`CrmFlowLogServiceImpl`、`RecordNoticeJob`）。

## 4. 其它子目录

| 目录 | 文件 | 说明 |
|------|------|------|
| `job/` | `CustomerAutoJob.java`、`RecordNoticeJob.java` | Quartz 定时任务（`JobHandler`） |
| `mq/consumer/` | `PayNoticeConsumer.java` | Redis Stream 消费者，处理支付成功通知 |
| `resources/mapper/crmcustomer/` | `CrmCustomerMapper.xml` | 客户自定义 SQL |
| `dal/redis/` | `RedisKeyConstants.java` | CRM 的 Redis Key 常量 |

## 5. 统计

| 项 | 数量 | 说明（统计于 `yshop-module-crm-biz/src/main/java`，已排除 `target/`） |
|----|------|------|
| Java 源文件 | 151 | 不含 `target/` 下的编译产物 |
| Controller 子包 | 15 | 见第 3 节 |
| Controller 类 | 15（`*Controller.java`） | 含 `PayController`、`AchievementController`、`CrmIndexController` 等 |
| Controller 接口方法映射 | 80 | `@GetMapping` / `@PostMapping` / `@PutMapping` / `@DeleteMapping` 合计 |
| Service 接口 | 15 | 与 `*ServiceImpl` 一一对应 |
| DO 实体 | 16 | `dal/dataobject/` |
| Mapper 接口 | 16 | `dal/mysql/` |
| 测试类 | 0 | 见 [../testing/existing-tests.md](../testing/existing-tests.md) |

## 6. 待验证

- `RankController` / `RankServiceImpl`：历史 SQL 日志中有 `/admin-api/crm/analysis-rank/contract` 的 NPE 记录，但当前 `yshop-module-crm-biz` 源码中未找到 `Rank*` 类；菜单表中存在 `analysis-rank-contract`（合同排行）、`analysis-rank-receivables`（回款排行），对应前端页面目录 `views/crm/analysis/` 也不存在。结论：排行功能的后端实现可能已删除或迁出，待确认。
- `yshop-module-crm-biz` 依赖的 DingTalk SDK 未见业务调用（见 [../03-architecture.md](../03-architecture.md)）。
