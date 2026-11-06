# 数据库表与实体索引（CRM）

> 数据源：`yshop-crm/sql/yshop-crm-open-source.sql`（全量脚本）与 `yshop-crm/sql/migration/20261008_p0_crm_business_indexes.sql`（增量索引）。
> 所有 CRM 表均包含通用字段：`creator`、`create_time`、`updater`、`update_time`、`deleted`（逻辑删除，1 为删除）、`tenant_id`（租户编号）。字段注释以 DDL 为准。
> 实体位于 `yshop-crm/yshop-module-crm/yshop-module-crm-biz/src/main/java/co/yixiang/yshop/module/crm/dal/dataobject/`。
> 表之间**无物理外键**（DDL 中未发现 `FOREIGN KEY`），以下关联均为业务层逻辑关联。

## 1. 表与实体对照

| 表名 | 实体（DO） | Mapper | 业务说明 | 主要关联字段（逻辑关联） |
|------|------------|--------|----------|--------------------------|
| `yshop_crm_clues` | `CrmCluesDO` | `CrmCluesMapper` | 线索 | `customer_id`（转客户后的客户ID）、`owner_user_id` |
| `yshop_crm_customer` | `CrmCustomerDO` | `CrmCustomerMapper` | 客户 | `owner_user_id`、`source`、`level`、`industry`、`province`/`city`/`area` |
| `yshop_crm_customer_contacts` | `CrmCustomerContactsDO` | `CrmCustomerContactsMapper` | 客户联系人 | `customer_id`、`owner_user_id` |
| `yshop_crm_business` | `CrmBusinessDO` | `CrmBusinessMapper` | 商机 | `customer_id`、`owner_user_id` |
| `yshop_crm_business_product` | `CrmBusinessProductDO` | `CrmBusinessProductMapper` | 商机产品明细 | `business_id`、`product_id` |
| `yshop_crm_contract` | `CrmContractDO` | `CrmContractMapper` | 合同 | `customer_id`、`business_id`、`contacts_id`、`flow_id`、`step_id`、`owner_user_id` |
| `yshop_crm_contract_product` | `CrmContractProductDO` | `CrmContractProductMapper` | 合同产品明细 | `contract_id`、`product_id` |
| `yshop_crm_contract_receivables` | `CrmContractReceivablesDO` | `CrmContractReceivablesMapper` | 回款 | `customer_id`、`contract_id`、`flow_id`、`step_id` |
| `yshop_crm_invoice` | `CrmInvoiceDO` | `CrmInvoiceMapper` | 发票申请 | `contract_id`、`customer_id`、`flow_id`、`step_id` |
| `yshop_crm_flow` | `CrmFlowDO` | `CrmFlowMapper` | 审批流程模板 | `types`（contract/receivables/invoice） |
| `yshop_crm_flow_step` | `CrmFlowStepDO` | `CrmFlowStepMapper` | 审批步骤 | `flow_id` |
| `yshop_crm_flow_log` | `CrmFlowLogDO` | `CrmFlowLogMapper` | 审批记录（每个审批人一条） | `flow_id`、`types`、`types_id`（单据ID）、`admin_id` |
| `yshop_crm_record` | `CrmRecordDO` | `CrmRecordMapper` | 跟进记录 | `types`（business/clues/customer）、`types_id` |
| `yshop_crm_operatelog` | `CrmOperatelogDO` | `CrmOperatelogMapper` | 操作日志 | `customer_id`、`contract_id`、`receivable_id` |
| `yshop_crm_message` | `CrmMessageDO` | `CrmMessageMapper` | 站内消息（审批通知等） | `relation_type`、`relation_id`、`to_user_id`、`from_user_id` |
| `yshop_crm_achievement` | `CrmAchievementDO` | `CrmAchievementMapper` | 业绩目标 | `type`（2员工/3部门）、`type_id`、`config`（1销售/2回款） |

> 全库 16 张 `yshop_crm_*` 表与 16 个 DO、16 个 Mapper 一一对应。

## 2. 核心表字段要点

### 2.1 `yshop_crm_clues`（线索，`CrmCluesDO`）

| 字段 | 类型 | 含义（来源：DDL 注释与枚举） |
|------|------|------|
| `status` | int | `-1` 无效线索，`0` 未转客户，`1` 转成客户（`CluesStatusEnum`） |
| `customer_id` | bigint | 转成客户后的客户 ID |
| `change_time` | datetime | 转客户时间 |
| `owner_user_id` | bigint | 负责人；放入公海时为 `0` |
| `collect_time`、`follow_time`、`next_time` | datetime | 领取时间、最后跟进时间、下次联系时间 |
| `follow_status` | tinyint | 跟进状态：1 新客，2 待再次沟通，3 有意向，4 已加微信，5 转商机，0 无效 |

### 2.2 `yshop_crm_customer`（客户，`CrmCustomerDO`）

| 字段 | 类型 | 含义 |
|------|------|------|
| `deal_status` | tinyint | 成交状态：`0` 未成交，`1` 成交（`ShopCommonEnum.IS_STATUS_0/1`） |
| `deal_time` | datetime | 成交时间 |
| `owner_user_id` | bigint | 负责人；公海时为 `0` |
| `is_lock` | bit | 锁定标记 |
| `rw_user_id`、`ro_user_id` | varchar | 读写 / 只读权限用户（实体为 String，**用途与使用位置待验证**） |
| `purchase_total`、`purchase_times` | decimal / int | 消费总额、消费次数 |
| `follow_status` | varchar | 跟进状态（取值同线索） |
| `collect_time`、`follow_time`、`next_time` | datetime | 同线索 |

### 2.3 `yshop_crm_business`（商机，`CrmBusinessDO`）

| 字段 | 类型 | 含义 |
|------|------|------|
| `customer_id` | bigint | 所属客户 |
| `status` | int | 销售阶段（DDL 注释仅写「销售阶段」，**具体取值待验证**，默认 1） |
| `is_end` | tinyint | 成交结果：`0` 洽谈中，`1` 成交，`2` 失败，`3` 无效（对应 `BusinessStatusEnum` 的 STATUS_0~3） |
| `money`、`total_price`、`discount_rate` | decimal | 商机金额、产品总金额、整单折扣 |
| `deal_time` | datetime | 预计成交日期 |
| `owner_user_id` | bigint | 负责人 |

### 2.4 `yshop_crm_contract`（合同，`CrmContractDO`）

### 2.5 `yshop_crm_contract_receivables`（回款，`CrmContractReceivablesDO`）

| 字段 | 类型 | 含义 |
|------|------|------|
| `number` | varchar | 回款编号，在线支付时作为 `uni` 参数 |
| `contract_id` / `customer_id` | bigint | 所属合同、客户 |
| `return_time` | datetime | 回款日期（支付成功时写入当前时间） |
| `money` | decimal | 回款金额 |
| `check_status` | int | 同合同 `check_status` |
| `flow_id` / `step_id` / `check_admin_id` / `flow_admin_id` | — | 同合同 |
| `pay_status` | tinyint | 在线支付状态：`1` 已付款，`0` 未付款（`OrderInfoEnum.PAY_STATUS_0/1`） |
| `pay_type` | tinyint | 收款方式：`1` 默认，`2` 在线收款，`3` 续费 |
| `code_url`、`pay_time` | varchar / datetime | 支付二维码路径、二维码生成时间（距今不足 2 小时则复用） |
| `account_id` | int | 收款账户 ID（支付成功时固定写入 `4`，见 `PayServiceImpl#paySuccess`） |

### 2.6 `yshop_crm_invoice`（发票，`CrmInvoiceDO`）

| 字段 | 类型 | 含义 |
|------|------|------|
| `contract_id` / `customer_id` | bigint | 关联合同、客户（DDL 索引：`contract_id`、`customer_i`） |
| `invoice_issue` | int | 抬头类型：`1` 企业，`2` 个人，`3` 事业单位 |
| `invoice_type` | int | `1` 增值税普通发票，`2` 增值税专用发票 |
| `money`、`tax_rate` | decimal | 发票税前金额、税率 |
| `invoice_admin_id` | bigint | 开票人（DDL 索引） |
| `check_status` | int | 同合同 `check_status` |
| `status` | int | 开票状态：`0` 未开票，`1` 开票中，`2` 已开票，`3` 开票失败（`InvoiceStatusEnum`） |
| `files` | varchar | 开票附件 |

### 2.7 审批相关表

| 表 | 关键字段 | 说明 |
|----|----------|------|
| `yshop_crm_flow` | `types`（`contract`/`receivables`/`invoice`）、`status`（1 启用，0 禁用）、`category`（1 固定审批，2 授权审批） | 审批流程模板；`createFlow` 时同步写入步骤 |
| `yshop_crm_flow_step` | `flow_id`、`type`、`admin_ids`（逗号分隔）、`relation`（顺序） | `type`：`1` 上级管理员，`2` 指定用户（任意一人），`3` 指定用户（多人会签）。代码中 `FlowStepEnum` 仅定义 2、3。`relation` 决定 `sendNextStep` 的审批顺序 |
| `yshop_crm_flow_log` | `flow_id`、`types`、`types_id`（单据 ID）、`admin_id`（审批人）、`status`、`remark`、`admin_time` | 每个待审批人一条记录。`status`：`0` 待审核，`1` 审核通过，`2` 审核拒绝，`3` 撤销；代码中还使用 `5` 审核中（`FlowLogStatusEnum.STATUS_5`），DDL 注释未列出 |

### 2.8 其它表


## 3. 核心关联关系（逻辑关联）

```
线索 yshop_crm_clues
  └─(转客户：customer_id)─▶ 客户 yshop_crm_customer ──┬─ 联系人 yshop_crm_customer_contacts (customer_id)
                                                     │
                                                     ├─ 商机 yshop_crm_business (customer_id)
                                                     │    └─ 商机产品 yshop_crm_business_product (business_id)
                                                     │
                                                     └─ 合同 yshop_crm_contract (customer_id, business_id 可空)
                                                          ├─ 合同产品 yshop_crm_contract_product (contract_id)
                                                          ├─ 回款 yshop_crm_contract_receivables (contract_id)
                                                          └─ 发票 yshop_crm_invoice (contract_id)

审批：yshop_crm_flow (types) ─▶ yshop_crm_flow_step (flow_id)
      yshop_crm_flow_log (types + types_id ⇒ 合同 / 回款 / 发票 单据)
跟进：yshop_crm_record (types + types_id ⇒ customer / business / clues)
日志：yshop_crm_operatelog (customer_id / contract_id / receivable_id)
消息：yshop_crm_message (relation_type + relation_id)
```

> 线索「转客户」的 DDL 字段关系已确认（`customer_id`、`change_time`、`status=1`），但**具体的转换代码（`createCustomer`）后端未实现**，见 [api-inventory.md](api-inventory.md) 第 11 节。

## 4. 注意事项

1. **无物理外键**：DDL 中未发现 `FOREIGN KEY`，删除/更新时的级联由业务代码控制。例如 `CrmCustomerServiceImpl#deleteCustomer` 会检查商机、合同、回款是否存在，存在则禁止删除。
2. **多态关联**：`yshop_crm_flow_log`、`yshop_crm_record`、`yshop_crm_message` 通过 `types` + `types_id` 或 `relation_type` + `relation_id` 指向不同业务单据，查询时必须同时带类型条件。
3. **跟进记录的 `types` 与审批的 `types` 取值集合不同**：
   - `yshop_crm_record.types`（DDL 枚举）为 `customer` / `business` / `clues`，仅用于跟进记录。
   - `yshop_crm_flow.types` / `yshop_crm_flow_log.types` 取值为 `contract` / `receivables` / `invoice`（`TypesEnum` 对应常量）。
   - 两者按业务分工使用，并非错误；`TypesEnum` 同时包含两组值。
4. **逗号分隔的 ID 列表**：`check_admin_id`、`flow_admin_id`、`admin_ids`、`rw_user_id`、`ro_user_id` 为字符串存储的多值，不能直接做 `IN` 关联，需用 `StrUtil.split` 等方式解析（见 `CrmFlowServiceImpl`）。
5. **金额字段**：统一使用 `decimal(18,2)`，计算应在 Service 中使用 `BigDecimal`（`CrmContractServiceImpl` 等已使用）。
6. **索引**：`sql/migration/20261008_p0_crm_business_indexes.sql` 为 CRM 核心查询补充了组合索引，主要覆盖 `owner_user_id + create_time`、`owner_user_id + next_time`、`deal_status + owner_user_id + follow_time` 等查询模式。修改分页查询条件时，请参考此文件。
7. **字段注释与代码不完全一致**：
   - `yshop_crm_flow_log.status` 的 DDL 注释未包含 `5`（审核中），代码中在 `FlowLogStatusEnum` 与 `CrmFlowServiceImpl#getFlowUserIds` 中使用。以代码为准。
   - `yshop_crm_business.status` 的 DDL 注释仅写「销售阶段」，具体取值待验证。
8. **DDL 中 `deleted` 为 bit(1)**：逻辑删除值与 `application.yaml` 中 `logic-delete-value: 1` 一致。

| 表 | 关键字段 | 说明 |
|----|----------|------|
| `yshop_crm_record` | `types` enum(`business`,`clues`,`customer`)、`types_id`、`content`、`record_type`、`next_time`、`lng` / `lat` | 跟进记录。**`types` 枚举不含 `contract`**，与 `TypesEnum`（含 contract/receivables/invoice）不一致，见第 4 节 |
| `yshop_crm_operatelog` | `customer_id`、`contract_id`、`receivable_id`、`title`、`content`、`url`、`ip` | 操作日志，由 `CrmOperatelogService.createLog` 写入 |
| `yshop_crm_message` | `relation_type`（如 `examine`）、`relation_id`、`to_user_id`、`from_user_id`、`status`（0 未操作，1 已操作）、`read_time` | 站内消息。审批通知与跟进提醒写入此表 |
| `yshop_crm_achievement` | `type`（2 员工，3 部门）、`type_id`、`config`（1 销售目标，2 回款目标）、`year`、`january`~`december`、`yeartarget` | 业绩目标，按月拆分 |
| `yshop_crm_customer_contacts` | `customer_id`、`name`、`mobile`、`decision`（1 是，0 否，-1 未知）、`sex`（1 男，0 女，-1 未知）、`owner_user_id` | 客户联系人 |
| `yshop_crm_business_product` / `yshop_crm_contract_product` | `business_id` / `contract_id`、`product_id`、`price`、`nums`、`subtotal`（合同另有 `discount`） | 产品明细，`product_id` 指向商城商品表（`yshop-module-product`） |


| 字段 | 类型 | 含义 |
|------|------|------|
| `customer_id` / `business_id` / `contacts_id` | bigint | 客户、来源商机、客户签约人（联系人） |
| `number` | varchar | 合同编号（`getCode` 生成） |
| `money` / `total_price` / `return_money` / `invoice_money` | decimal | 合同金额、产品总金额、已回款、已开票金额 |
| `check_status` | int | `0` 待审核、`1` 审核中、`2` 审核通过、`3` 审核未通过（`ContractStatusEnum`） |
| `flow_id` / `step_id` | bigint | 审批流程 ID、当前审批步骤 ID（`yshop_crm_flow_step.id`） |
| `check_admin_id` | varchar | 已审批人 ID 列表（逗号分隔） |
| `flow_admin_id` | varchar | 当前待审批人 ID（逗号分隔） |
| `order_admin_id` | bigint | 公司签约人 |
| `expire_handle` | int | `0` 合同过期未处理，`1` 已续签，`2` 不再合作 |
| `source_id` | int | 客户来源（新增/修改时取自客户的 `source`） |
