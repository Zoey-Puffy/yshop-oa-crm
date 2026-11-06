# 03 系统架构

## 1. 总体架构

```
┌────────────────────┐        HTTP /admin-api/...        ┌─────────────────────────────┐
│  yshop-crm-vue     │  ───── axios (Bearer Token) ───▶  │  yshop-server (Spring Boot) │
│  Vue3 + Element    │                                   │  - 聚合各 yshop-module-*    │
│  Plus / Pinia      │  ◀──── JSON CommonResult ──────   │  - Security / Tenant / XSS  │
└────────────────────┘                                   └──────────────┬──────────────┘
                                                                        │
                    ┌───────────────────────────────────────────────────┼───────────────────────────┐
                    │                                                   │                           │
          ┌─────────▼─────────┐                             ┌───────────▼─────────┐        ┌────────▼────────┐
          │ MySQL（主库，     │                             │ Redis（缓存、分布式  │        │ Quartz（JDBC）   │
          │ 动态数据源 master/│                             │ 锁、Stream 消息）    │        │ 定时任务         │
          │ slave）           │                             └─────────────────────┘        └─────────────────┘
          └───────────────────┘
```

- 前端与后端通过 `/admin-api` 前缀通信（`yshop-crm-vue/.env.dev` 中 `VITE_API_URL=/admin-api`）。
- 后端由 `yshop-server` 聚合所有业务模块，CRM 以 Maven 依赖方式引入（`yshop-server/pom.xml` 依赖 `yshop-module-crm-biz`）。

## 2. 后端分层（以 CRM 为例）

| 层 | 包路径（`co.yixiang.yshop.module.crm.`） | 职责 | 示例 |
|----|--------------------------------------------|------|------|
| Controller | `controller.admin.<业务域>` | REST 接口、参数接收、权限注解 | `crmcustomer/CrmCustomerController.java` |
| VO | `controller.admin.<业务域>.vo` | 请求/响应对象（`*SaveReqVO`、`*PageReqVO`、`*RespVO`） | `CrmCustomerSaveReqVO.java` |
| Service 接口 | `service.<业务域>` | 业务接口 | `CrmCustomerService.java` |
| Service 实现 | `service.<业务域>` | 业务逻辑、事务（`@Transactional(rollbackFor = Exception.class)`）、调用其它域 Service | `CrmCustomerServiceImpl.java` |
| Mapper | `dal.mysql.<业务域>` | 继承 `BaseMapperX<DO>`（MyBatis-Plus 扩展），部分含自定义查询 | `CrmCustomerMapper.java` |
| DO | `dal.dataobject.<业务域>` | 表实体，`@TableName`、`@KeySequence` | `CrmCustomerDO.java` |
| 枚举/错误码 | `yshop-module-crm-api` 下 `enums` | 状态、类型常量；`ErrorCodeConstants` 统一错误码 | `ErrorCodeConstants.java` |
| 定时任务 | `job` | 实现 `JobHandler`，由 Quartz 调度 | `CustomerAutoJob.java` |
| 消息消费 | `mq.consumer` | 继承 `AbstractRedisStreamMessageListener` | `PayNoticeConsumer.java` |

### 2.1 典型调用链（已确认）

以「新增合同」为例（源码：`service/crmcontract/CrmContractServiceImpl.java#createContract`）：

```
CrmContractController  POST /crm/contract/create
  └─ CrmContractService.createContract            [@Transactional]
       ├─ CrmCustomerMapper.selectById             （取客户来源 sourceId）
       ├─ CrmContractMapper.insert
       ├─ createContractProductList                （子表 CrmContractProductMapper）
       ├─ updateAfter                              （设置审批人、审核状态；内部使用 Redisson 锁 contract:check:lock）
       ├─ CrmOperatelogService.createLog           （操作日志）
       └─ CrmFlowService.sendNextStep              （生成审批日志、站内消息）
```

### 2.2 跨模块依赖（已确认）

| 依赖模块 | 源码中的使用点 |
|----------|----------------|

## 3. 认证与鉴权

| 项 | 实现 | 位置 |
|----|------|------|
| 登录态 | 框架基于 OAuth2（README 描述为 Spring Security OAuth2 + JWT）；前端通过 `getAccessToken()` 读取令牌 | `yshop-crm-vue/src/utils/auth.ts`、`src/permission.ts` |
| 接口权限 | `@PreAuthorize("@ss.hasPermission('crm:xxx:yyy')")`，CRM Controller 中共 70 处 | `controller/admin/**/*Controller.java` |
| 免登录接口 | `@PreAuthenticated` 注解（仅 1 处，`PayController#pay`，路径 `/crm/order/pay`） | `controller/admin/crmpay/PayController.java` |
| 安全放行 | `yshop.security.permit-all_urls` 包含 `/admin-api/crm/order/notify/**`（支付回调） | `yshop-server/src/main/resources/application.yaml` |
| 菜单/按钮权限 | 菜单表 `system_menu.permission`（如 `crm:customer:create`）；前端按钮使用 `v-hasPermi` | `sql/yshop-crm-open-source.sql`；`views/crm/crmcustomer/index.vue` |
| 数据权限 | 按 `owner_user_id` 与 `relation`（`my`/`sub`/`all`）在 Service 层过滤 | `service/crmcustomer/CrmCustomerServiceImpl.java#getCustomerPage` |

> 前端路由守卫 `permission.ts` 的 `whiteList`：`/login`、`/social-login`、`/auth-redirect`、`/bind`、`/register`、`/oauthLogin/gitee`。

## 4. 多租户

- 配置：`application.yaml` 中 `yshop.tenant.enable: false`；前端 `.env` 中 `VITE_APP_TENANT_ENABLE=false`。
- 忽略 URL（`yshop.tenant.ignore-urls`）中**不包含** CRM 支付回调，支付回调的放行在 `yshop.security.permit-all_urls` 中配置。
- 业务代码 `PayServiceImpl#paySuccess` 标注 `@TenantIgnore`，因为支付回调不携带租户编号。

## 5. 数据访问

| 项 | 实现 |
|----|------|

## 6. 异步与定时

| 类型 | 实现 | 位置 | 说明 |
|------|------|------|------|
| Quartz 定时任务 | 实现 `JobHandler` 的 `@Component` | `job/CustomerAutoJob.java`（`customerAutoJob`，cron `0 0 0 * * ?`，自动公海）；`job/RecordNoticeJob.java`（`recordNoticeJob`，cron `0 50 8 * * ?`，跟进提醒） | 任务配置存于 `infra_job`，Quartz 表 `QRTZ_*`（见 `sql/yshop-crm-open-source.sql`） |
| Redis Stream 消费 | 继承 `AbstractRedisStreamMessageListener<T>` | `mq/consumer/PayNoticeConsumer.java`（消费 `PayNoticeMessage`，调用 `PayService.paySuccess`） | 支付成功后更新回款状态与合同回款金额 |
| 站内消息 / 微信通知 | `CrmMessageMapper`（CRM 站内消息）、`WeixinNoticeProducer`（微信模板消息） | `RecordNoticeJob.java`、`CrmFlowServiceImpl#addStepLog` | 审批待办、跟进提醒 |
| 分布式锁 | Redisson `RLock`，key 为 `contract:check:lock` | `CrmContractServiceImpl#updateAfter` | 合同审核并发控制（锁的完整释放逻辑**未逐行核对**） |

> `infra_job_log` 历史记录中，`customerAutoJob` 曾出现 `NullPointerException: LoginUser...`（2025-02-13）。当前 `CustomerAutoJob` 代码不依赖登录用户，该问题的修复状态**待验证**。

## 7. 前端架构

| 项 | 实现 | 位置 |
|----|------|------|
| 入口 | `main.ts`、`App.vue` | `yshop-crm-vue/src/` |
| 路由 | `createWebHistory`（History 模式），静态路由 + 动态路由 | `src/router/index.ts`、`src/router/modules/remaining.ts` |
| 动态路由 | 登录后由后端菜单（`system_menu`）生成路由 | `src/store/modules/permission.ts`、`src/utils/routerHelper.ts` |
| 组件加载 | `import.meta.glob('../views/**/*.{vue,tsx}')`，按菜单 `component` 字段（如 `crm/crmcustomer/index`）匹配 | `src/utils/routerHelper.ts#registerComponent` |
| 状态 | Pinia 模块（`permission`、`user`、`dict`、`tagsView` 等） | `src/store/modules/` |
| HTTP | axios 实例，`baseURL = VITE_BASE_URL + VITE_API_URL`，超时 30s，成功码 `200` | `src/config/axios/config.ts`、`src/config/axios/service.ts` |
| 字典 | 登录后 `dictStore.setDictMap()`，页面通过 `getIntDictOptions(DICT_TYPE.xxx)` 取值 | `src/store/modules/dict.ts`、`src/utils/dict.ts` |
| UI | Element Plus；通用组件 `Table`、`Search`、`Pagination` 等 | `src/components/` |

## 8. 配置与环境

| 环境 | 前端 | 后端 |
|------|------|------|
| 本地（local） | `.env.local`：`VITE_BASE_URL=http://localhost:48081`，`VITE_API_URL=/admin-api` | `application-local.yaml`：端口 48081，MySQL 库 `yshop-crm-open-source`，Quartz `auto-startup: true` |
| 开发（dev） | `.env.dev`：`VITE_BASE_URL` 指向外部域名（**是否为正式环境待验证**） | `application-dev.yaml`：MySQL 库 `yshop_crm`，Redis database=2，Quartz `auto-startup: false` |
| 生产/预发/测试 | `.env.prod`、`.env.stage`、`.env.test` | 未检索到 `application-prod.yaml`（**待验证**） |

> 以上配置文件含数据库账号、密码、第三方密钥等，本知识库**不记录任何具体值**。

## 9. 关键设计约定（已确认）

- 后端 Controller 统一返回 `CommonResult<T>`，分页返回 `PageResult<T>`（`yshop-framework/yshop-common`）。
- 业务异常统一使用 `ServiceExceptionUtil.exception(ErrorCode)`，CRM 错误码集中在 `ErrorCodeConstants`（200000~200017）；源码中也存在 `new ErrorCode(2024xxxxx, ...)` 的内联错误码，属历史写法。
- 对象转换统一使用 `BeanUtils.toBean`（`yshop-framework/yshop-common`）。
- 审批流程由 `CrmFlowServiceImpl#sendNextStep` 驱动，审批结果由 `CrmFlowLogServiceImpl#examine` 写回业务表（合同、回款、发票）。
- 操作日志统一调用 `CrmOperatelogService.createLog(title, customerId, contractId, receivableId)`。

## 10. 待验证

- 企业微信（WorkWx）、云呼（CloudCall）：CRM 业务源码中未找到对应实现，但 `ErrorCodeConstants` 仍保留 200010~200017 错误码，待确认是否已迁出。
- `CrmContractServiceImpl#updateAfter` 中分布式锁的释放与 `tryLock` 失败分支的处理，需对照源码确认。
- `yshop-crm-vue/src/views/crm/` 中带 `2`、`3` 后缀的页面（如 `crmcontract/index2.vue`、`ContractForm2.vue`）是否仍在菜单中使用，待验证。

| ORM | MyBatis-Plus，`BaseMapperX<DO>` 扩展（`yshop-spring-boot-starter-mybatis`） |
| 数据源 | `dynamic-datasource` + Druid，主库 `master`，从库 `slave`（`lazy: true`） |
| 主键 | `mybatis-plus.global-config.db-config.id-type: NONE`（智能模式）；DO 上的 `@KeySequence` 仅对 Oracle/PG 等有效，MySQL 无需 |
| 逻辑删除 | `logic-delete-value: 1`、`logic-not-delete-value: 0`，DO 字段 `deleted` |
| 多表关联 | `mybatis-plus-join`（如 `CrmContractMapper.selectContractTop`） |
| 加密 | `mybatis-plus.encryptor.password` 已配置（**值不记录**） |
| 自定义 XML | `yshop-module-crm-biz/src/main/resources/mapper/crmcustomer/CrmCustomerMapper.xml` |

| `yshop-module-system-biz` | 用户、部门（`AdminUserApi`、`AdminUserService`）、短信/邮件（`SmsSendService`、`MailSendService`） |
| `yshop-module-pay-biz` | 商户配置（`MerchantDetailsService`）、支付框架（`PayServiceManager`） |
| `yshop-module-product-biz` | 商品与商品属性（`StoreProductMapper`、`StoreProductAttrValueMapper`，用于合同/商机产品） |
| `yshop-module-message-biz` | 微信模板消息（`WeixinNoticeProducer`、`WechatTempateEnum`） |
| `yshop-spring-boot-starter-security` | 登录用户（`SecurityFrameworkUtils.getLoginUserId()`） |
| `yshop-spring-boot-starter-redis` | `RedisUtil`（定时任务读取参数） |
| `yshop-spring-boot-starter-mq` | Redis Stream 消费者基类 |
| `yshop-spring-boot-starter-biz-tenant` | 多租户忽略（`@TenantIgnore`） |
| `yshop-spring-boot-starter-excel` | 客户导出（`ExcelUtils.write`，`CrmCustomerController`） |
| `alibaba-dingtalk-service-sdk`、`dingtalk` | 依赖已声明，CRM 源码中未检索到调用（**待验证**） |
