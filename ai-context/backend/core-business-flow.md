# 核心业务流程（CRM）

> 本文档描述可从源码确认的业务流程。未能确认的部分统一标注「待验证」。
> 状态码以 `yshop-module-crm-api/src/main/java/co/yixiang/yshop/module/crm/enums/` 下的枚举为准。

## 1. 业务主线

```
线索（clues）──转客户──▶ 客户（customer）──▶ 商机（business）──▶ 合同（contract）──▶ 回款（receivables）
                             │                                          │
                             │                                          └──▶ 发票（invoice）
                             ▼
                        联系人 / 跟进记录 / 操作日志

审批：合同 / 回款 / 发票 均通过「审批流程模板（flow）+ 审批步骤（flow_step）+ 审批记录（flow_log）」驱动
```

> 「线索转客户」的前端入口为 `CustomerForm.vue`，调用 `/crm/clues/createCustomer`，但**后端未实现该接口**（见 [api-inventory.md](api-inventory.md) 第 11 节），线索到客户的实际转换逻辑**待验证**。

## 2. 线索（Clues）

### 2.1 状态

| 状态值 | 枚举 | 含义 |
|--------|------|------|
| `-1` | `CluesStatusEnum.STATUS_NEG_1` | 无效线索（放入公海时设置） |
| `0` | `CluesStatusEnum.STATUS_0` | 未转客户（领取时设置） |
| `1` | `CluesStatusEnum.STATUS_1` | 转成客户（DDL 注释存在，代码中未检索到设置点，**待验证**） |

### 2.2 公海与领取（已确认）

| 动作 | 接口 | Service 实现 | 关键逻辑 |
|------|------|--------------|----------|
| 放入公海 | `GET /crm/clues/open?id=` | `CrmCluesServiceImpl#openClues` | 若状态为 `0` 改为 `-1`；`owner_user_id` 置 `0`；删除该线索的跟进记录 |
| 领取公海线索 | `GET /crm/clues/receive?id=` | `CrmCluesServiceImpl#receiveClues` | `owner_user_id` 置为当前登录用户；设置 `collect_time`、`follow_time`；状态置为 `0` |

> 公海列表：`GET /crm/clues/page?type=open`（`CustomerTypesEnum.OPEN`），走 `cluesMapper.selectPage2`，不按负责人过滤。

### 2.3 分页数据权限（已确认）

`CrmCluesServiceImpl#getCluesPage`：
- `relation = my`：仅本人负责的线索。
- `relation = sub`：本人及下属（`AdminUserApi.getUserListBySubordinateIds`）。
- 其它值：不限制负责人。

## 3. 客户（Customer）

### 3.1 新增与联系人（已确认）

`CrmCustomerServiceImpl#createCustomer`（`@Transactional`）：
1. `limitCustomer(loginUserId)`：限制单个用户可创建的客户数量，阈值实现**待验证**。
2. 写入 `yshop_crm_customer`，设置 `collect_time`、`follow_time` 为当前时间，`owner_user_id` 为当前用户。
3. 自动创建一条默认联系人（`yshop_crm_customer_contacts`），姓名、手机、电话、备注、微信取自客户表单。
4. 写操作日志「添加客户」。

### 3.2 公海与领取（已确认）

| 动作 | 接口 | 实现 | 关键逻辑 |
|------|------|------|----------|
| 放入公海 | `GET /crm/customer/do-open?id=` | `CrmCustomerServiceImpl#openCustomer` | `owner_user_id` 置 `0`；同步将所有联系人的 `owner_user_id` 置 `0`；删除客户跟进记录；写日志「放入公海」 |

## 5. 合同（Contract）

### 5.1 创建（已确认）

`CrmContractServiceImpl#createContract`（`@Transactional`）：
1. 查询客户，若客户有 `source` 则写入合同 `sourceId`。
2. 写入 `yshop_crm_contract`，`owner_user_id` 为当前用户；写入合同产品。
3. 调用 `updateAfter(contractId)`：查找最新的合同审批流程（`flow.types = contract`，按 ID 倒序取第一条），取其第一个审批步骤，回写合同的 `flow_id`、`step_id`。
4. 写操作日志「添加合同」。
5. 调用 `CrmFlowService.sendNextStep(contract, id, "")` 发送第一步审批通知。

> 若系统中没有合同审批流程模板，`updateAfter` 中 `crmFlowDO` 为 `null` 会导致 NPE，**是否有前置校验待验证**。

### 5.2 审核状态（`ContractStatusEnum`，已确认）

| 值 | 含义 |
|----|------|
| `0` | 待审核（新建后） |
| `1` | 审核中（某一步通过、仍有后续步骤） |
| `2` | 审核通过（所有步骤完成） |
| `3` | 审核未通过（被拒绝） |

### 5.3 审批推进（已确认，以审批记录为准）

审批的主推进逻辑在 `CrmFlowLogServiceImpl#examine`，而不是合同的 `check` 方法：

```
审批人点击「同意」（CheckTypesEnum.AGREE）
  └─ CrmFlowLogServiceImpl#examine
       ├─ 当前审批记录 status：0 → 1（通过）
       ├─ 组合已审批人（当前用户 + 合同 check_admin_id）
       ├─ CrmFlowService#sendNextStep(contract, contractId, checkedUserIds)
       │     ├─ 按 relation 顺序遍历审批步骤
       │     ├─ 任一步骤未完成 → 为待审批人写审批记录并站内通知，返回 isSuccess=false
       │     └─ 全部完成 → 返回 isSuccess=true，stepId=0
       └─ isSuccess=true 时（整条审批链完成）：
             ├─ 合同 check_status → 2，check_admin_id 更新，step_id 更新
             ├─ 客户 deal_status → 1，累加 purchase_times、purchase_total
             ├─ 若 business_id > 0，商机 is_end → 1
             └─ 将同一单据的其它待审批记录 status → 3（撤销）
```

审批人点击「拒绝」（`CheckTypesEnum.REFUSE`）：
- 当前审批记录 status → 2（审核拒绝）。
- 合同 `check_status` → 3，清空 `check_admin_id`。
- 若关联商机，`is_end` → 2（失败）。
- 站内通知发起人「您提交的审核被拒绝」。


### 6.3 在线支付（已确认）

下单（扫码）：

```
前端 ContractReceivablesForm.vue
  └─ POST /crm/order/pay  {uni: 回款编号, paytype: weixin|ali}     （@PreAuthenticated）
       └─ PayServiceImpl#pay
            ├─ 按 number + pay_status=0 查询回款，找不到则报错「没有未支付回款订单」
            ├─ 若 code_url 存在且 pay_time 距今不足 2 小时，直接复用二维码
            ├─ weixin：PayIdEnum.WX_WECHAT + NATIVE；ali：PayIdEnum.ALI_H5 + SWEEPPAY
            ├─ 生成二维码图片，保存到 yshop.file-path + qrcode/{回款编号}.jpg
            └─ 回写 code_url、pay_time，返回二维码访问路径
```

支付回调（已确认）：

```
支付渠道 ──异步回调──▶ /admin-api/crm/order/notify/payBack{detailsId}.json   （免登录，yshop.security.permit-all_urls）
       └─ PayController#payBack ──▶ PayServiceManager.payBack（pay-java-parent）
            └─ 投递 PayNoticeMessage 至 Redis Stream
                 └─ PayNoticeConsumer#onMessage ──▶ PayServiceImpl#paySuccess   （@TenantIgnore）
                      ├─ 回款 pay_status=1、return_time=now、check_status=2、account_id=4
                      └─ 合同 return_money 累加回款金额（注意：此处直接更新，未调用 incReturnMoney）
```

> 注意 1：`paySuccess` 中回款直接置 `check_status = 2`（审核通过），**未走审批流程**。是否符合预期，**待验证**。
> 注意 2：`account_id` 固定写入 `4`（硬编码），含义**待验证**。
> 注意 3：`paySuccess` 中合同回款金额的累加方式为「读取 DO 后整体 `updateById`」，与审批通过时使用的 `incReturnMoney` 不一致，并发下的安全性**待验证**。
> 注意 4：回调路径源码为 `/notify/payBack{detailsId}.json`（无分隔符），与商户后台 `notify_url` 是否一致，**待验证**。

### 6.4 回款与合同的金额关系

- 合同 `money`：合同金额。
- 合同 `return_money`：已收到款项（审批通过回款 或 支付成功时累加）。
- 合同 `invoice_money`：已开票金额，在**开票时**（`CrmInvoiceServiceImpl#issueInvoice`，`PUT /crm/invoice/issue`）累加，而不是审批通过时（已确认，`CrmInvoiceServiceImpl.java` 第 329-330 行）。

## 7. 发票（Invoice）

### 7.1 申请前置校验（已确认）

`CrmInvoiceServiceImpl#preCheck` 依次校验，任一失败即报错：
1. 合同必须存在。
2. 合同 `check_status` 必须为 `2`（审核通过），否则「合同未通过审核不可以申请发票」。
3. 同一合同下，若存在 `check_status` 或 `status` 为 `0`/`1` 的发票（未审核或未开票完成），且为新增（`id == null`）时，禁止再次申请。
4. 累计开票金额（本次金额 + 合同已开票金额 `invoice_money`）不能大于合同金额。

通过校验后，`createInvoice` 写入发票、调用 `sendNextStep(invoice, id, "")` 发起审批。

### 7.2 审批与开票（已确认）

- 审批通过（整条审批链完成）：`check_status` → `2`。
- 审批拒绝：`check_status` → `3`。
- 开票：`PUT /crm/invoice/issue` → `CrmInvoiceServiceImpl#issueInvoice`：
  - 将发票 `status` 置为 `2`（已开票，`InvoiceStatusEnum.STATUS_2`）。
  - 将合同 `invoice_money` 累加本次开票金额。

## 8. 定时任务与异步

| 任务 | Job Handler | 触发 | 逻辑 | 相关配置 |
|------|-------------|------|------|----------|
| 自动放入公海 | `customerAutoJob` → `CustomerAutoJob` | cron `0 0 0 * * ?`（每天 0 点） | 从 Redis 读取 `notRecordDay`（超过 N 天未跟进）与 `notSuccessDay`（超过 N 天未成交）。遍历 `deal_status=0` 且 `owner_user_id != 0` 的客户，满足阈值则调用 `openCustomer` | 阈值由 `CrmCustomerController#saveRedisSet` 写入 Redis |
| 每日跟进提醒 | `recordNoticeJob` → `RecordNoticeJob` | cron `0 50 8 * * ?`（每天 8:50） | 查询当天 `next_time` 落在当日的跟进记录，写入站内消息 `yshop_crm_message`，并调用 `WeixinNoticeProducer` 发送微信模板消息 | 模板 `WechatTempateEnum.RECORD_NOTICE` |

> 历史运行日志：`infra_job_log` 中 `customerAutoJob` 存在多次 `NullPointerException: LoginUser...`（2025-02-13），当前代码中该 Job 不依赖登录用户，修复状态**待验证**。
> 任务的启停由 `application-dev.yaml` 的 `spring.quartz.auto-startup` 控制（dev 为 `false`，local 为 `true`）。

## 9. 待验证汇总（业务流程）

1. 线索转客户（`/crm/clues/createCustomer`）的后端实现与线索状态 `1` 的设置位置。
2. `limitCustomer` 的具体阈值与实现。
3. 合同审批 `check`（已废弃）与 `examine`（当前使用）两套逻辑并存时，前端 `ContractForm2.vue` 的实际调用效果。
4. 合同 `updateAfter` 在无流程模板时的 NPE 风险。
5. 支付成功回款直接审核通过（跳过审批）的设计意图。
6. 支付回调地址与商户配置的一致性。
7. 商机 `status` 的销售阶段取值。

> 注意：`CrmContractServiceImpl#check(CheckInfoVO)` 标注了 `@Deprecated`。前端 `ContractForm2.vue` 仍调用 `ContractApi.checkContract` → `POST /crm/contract/check`，这一调用链与审批记录驱动的 `examine` 并存。**该页面是否仍在菜单中使用，待验证**。

### 5.4 删除与修改约束（已确认）

- `deleteContract`：仅当 `check_status = 0` 时允许删除，否则报错「合同不是待审核状态不可以删除」。
- `updateContract`：会重新写入合同产品，但**不会**重新调用 `updateAfter`，审批流程不会因修改重置（`updateAfter` 调用处已被注释，**设计意图待验证**）。

## 6. 回款（Receivables）

### 6.1 创建（已确认）

`CrmContractReceivablesServiceImpl#createContractReceivables`：写入回款（`owner_user_id` 为当前用户），调用 `flowService.sendNextStep(receivables, id, "")` 发起审批。回款与合同共用审批流程模板（`flow.types = receivables`）。

### 6.2 审批通过（已确认）

`CrmFlowLogServiceImpl#examine` 中 `RECEIVABLES` 分支（整条审批链完成后）：
- 回款 `check_status` → `2`，`step_id` 更新。
- 调用 `contractMapper.incReturnMoney(money, contractId)`，累加合同已回款金额 `return_money`。

| 领取公海客户 | `GET /crm/customer/receive?id=` | `CrmCustomerServiceImpl#receiveCustomer` | 先 `limitCustomer`；`owner_user_id` 设为当前用户；更新领取/跟进时间；联系人同步归属；写日志「公海领取客户」 |
| 自动放入公海 | 定时任务 `customerAutoJob` | `CustomerAutoJob#execute` | 见第 8 节 |

> 公海与私海：`owner_user_id = 0` 视为公海；`getCustomerPage` 中 `type=open` 时仅查 `ids=[0]`。

### 3.3 删除约束（已确认）

`CrmCustomerServiceImpl#deleteCustomer` 删除前检查以下关联，**存在任一则禁止删除**：商机、合同、回款（分别按 `customer_id` 计数）。删除时同时删除该客户的跟进记录与联系人。

### 3.4 成交状态（已确认）

成交由**合同审批通过**触发（见第 5 节），客户 `deal_status` 置 `1`，累加 `purchase_times`、`purchase_total`，记录 `deal_time`。

## 4. 商机（Business）

### 4.1 基本信息

- 实体 `CrmBusinessDO`，表 `yshop_crm_business`；商机产品在 `yshop_crm_business_product`。
- `is_end`（`BusinessStatusEnum`）：`0` 洽谈中，`1` 成交，`2` 失败，`3` 无效。
- `status`：销售阶段，具体取值**待验证**（DDL 注释未列出）。

### 4.2 创建（已确认）

`CrmBusinessServiceImpl#createBusiness`（`@Transactional`）：设置 `owner_user_id` 为当前用户，写入商机，批量写入商机产品（回填 `business_id`）。

### 4.3 与合同的关系（已确认）

- 合同可通过 `business_id` 关联商机（新增合同时可空）。
- 合同审批通过时，若 `business_id > 0`，将商机 `is_end` 置为 `1`，备注写入「合同已签署成交」。
- 合同被拒绝时（`CrmFlowLogServiceImpl#examine`），将商机 `is_end` 置为 `2`，备注写入「合同被拒绝签约失败」。
