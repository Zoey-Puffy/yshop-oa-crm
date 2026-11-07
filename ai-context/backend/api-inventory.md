# 后端接口清单（CRM）

> 基础路径：`/admin-api`（前端 `VITE_API_URL`）+ Controller 的 `@RequestMapping`。
> 完整路径 = `/admin-api` + 下表中的路径。
> 统一返回 `CommonResult<T>`。权限注解为 `@PreAuthorize("@ss.hasPermission('crm:...')")`，除特别说明外均需登录。
> 「前端调用」列依据 `yshop-crm-vue/src/api/crm/*/index.ts` 中的方法定义与 `views/` 下的实际调用整理。仅写出已确认的调用页面；标注「待验证」表示未能确认调用位置。前端 API 方法存在但页面未调用的情况，以页面实际调用为准（见 [../frontend/page-api-mapping.md](../frontend/page-api-mapping.md)）。

## 1. 线索（`CrmCluesController`，`/crm/clues`）

| HTTP | 路径 | 方法 | Service 方法 | 用途 | 前端调用 |
|------|------|------|--------------|------|----------|
| POST | `/crm/clues/create` | `createClues` | `CrmCluesService.createClues` | 新增线索 | `api/crm/crmclues` `createClues`，`views/crm/crmclues/CluesForm.vue` |
| PUT | `/crm/clues/update` | `updateClues` | `updateClues` | 修改线索 | `updateClues`，`CluesForm.vue` |
| DELETE | `/crm/clues/delete?id=` | `deleteClues` | `deleteClues` | 删除线索（同时删除跟进记录） | `deleteClues`，`views/crm/crmclues/index.vue` |
| GET | `/crm/clues/get?id=` | `getClues` | `getClues` | 线索详情 | `getClues`，`CluesDetail.vue`、`CluesForm.vue` |
| GET | `/crm/clues/page` | `getCluesPage` | `getCluesPage` | 线索分页（`type=open` 时走公海查询） | `getCluesPage`，`crmclues/index.vue`、`crmopenclues/index.vue` |
| GET | `/crm/clues/open?id=` | `openClues` | `openClues` | 线索放入公海（owner 置 0，状态 -1） | `openClues`，`crmclues/index.vue`、`CluesDetail.vue` |
| GET | `/crm/clues/receive?id=` | `receiveClues` | `receiveClues` | 领取公海线索（owner 置当前用户，状态 0） | `receClues`，`crmopenclues/index.vue` |
| POST | `/crm/clues/createCustomer` | 后端未找到对应 Mapping | — | 线索转客户（**前后端不一致**，见第 11 节） | `createCustomer`，`crmclues/components/customer/CustomerForm.vue` |
| GET | `/crm/clues/export-excel` | 后端未找到映射 | — | 线索导出（**前后端不一致**，见第 11 节） | `exportClues`，前端 API 中存在 |

> 注：线索「转客户」的后端实现待验证。前端 `CustomerForm.vue` 调用 `/crm/clues/createCustomer`，但 CRM 后端未检索到该映射；客户创建的后端接口为 `POST /crm/customer/create`。

## 2. 客户（`CrmCustomerController`，`/crm/customer`）

| HTTP | 路径 | 方法 | Service 方法 | 用途 | 前端调用 |
|------|------|------|--------------|------|----------|
| POST | `/crm/customer/create` | `createCustomer` | `CrmCustomerService.createCustomer` | 新增客户（同时创建默认联系人，受 `limitCustomer` 限制） | `CustomerForm.vue` |
| PUT | `/crm/customer/update` | `updateCustomer` | `updateCustomer` | 修改客户 | `CustomerForm.vue` |
| DELETE | `/crm/customer/delete?id=` | `deleteCustomer` | `deleteCustomer` | 删除客户（有商机/合同/回款时禁止删除） | `crmcustomer/index.vue` |
| GET | `/crm/customer/get?id=` | `getCustomer` | `getCustomer` | 客户详情 | `CustomerDetail.vue`、`CustomerForm.vue`、`BusinessDetail.vue` 等 |
| GET | `/crm/customer/page` | `getCustomerPage` | `getCustomerPage` | 客户分页（公海/私海，返回 `CrmCustomerDO`） | `components/customer/index.vue`、`crmopencustomer/index.vue` |
| GET | `/crm/customer/page2` | `getCustomerPage2` | `getCustomerPage2` | 客户分页（含关系 `all`，返回 `CrmCustomerRespVO`） | `crmcustomer/index.vue`、`crmrepeatcustomer/index.vue` |
| GET | `/crm/customer/do-open?id=` | `openCustomer` | `openCustomer` | 客户放入公海（owner 置 0） | `CustomerDetail.vue`、`crmcustomer/index.vue` |
| GET | `/crm/customer/receive?id=` | `receiveCustomer` | `receiveCustomer` | 领取公海客户（**含 `limitCustomer` 限制**） | `crmopencustomer/index.vue` |
| GET | `/crm/customer/export-excel` | `exportCustomerExcel`（`ExcelUtils.write`，权限 `crm:customer:export`） | 无（直接调用 `getCustomerPage`） | 导出客户 Excel | 前端 API 文件中有 `exportCustomer`，调用位置待验证 |
| POST | `/crm/customer/send-sms` | `sendSms`（直接调用 Service） | `CrmCustomerService.sendSms` | 批量发送短信（`SmsTemplateSendVO`） | `components/sms/SmsTemplateSendForm.vue` |
| POST | `/crm/customer/send-mail` | `sendMail` | `CrmCustomerService.sendMail` | 批量发送邮件（`MailTemplateSendVO`） | `components/email/MailTemplateSendForm.vue` |
| POST | `/crm/customer/saveRedisSet` | `saveRedisSet`，直接调用 `RedisUtil.set` | 无 | 保存自动公海规则：`customerNum`、`notRecordDay`、`notSuccessDay` 三个 Redis key | `crmcustomer/CustomerConfigForm.vue` |
| GET | `/crm/customer/getRedisSet` | `getRedisSet`，直接调用 `RedisUtil.get` | 无 | 读取上述规则配置（与 `CustomerAutoJob` 读取的 key 一致） | `CustomerConfigForm.vue` |
| GET | `/crm/customer/customer-contacts/list-by-customer-id?customerId=` | 后端未找到映射 | — | 客户联系人列表（**前后端不一致**，见第 11 节） | `api/crm/crmcustomer/index.ts` 中定义，页面调用位置待验证 |

> 注：`saveRedisSet`/`getRedisSet` 读写的 Redis key（`customerNum`、`notRecordDay`、`notSuccessDay`）与 `CustomerAutoJob` 读取的 key 一致，修改时需同步检查。

## 3. 联系人（`CrmCustomerContactsController`，`/crm/customer-contacts`）
| HTTP | 路径 | 方法 | Service 方法 | 用途 | 前端调用 |
|------|------|------|--------------|------|----------|
| POST | `/crm/customer-contacts/create` | `createCustomerContacts` | `CrmCustomerContactsService` | 新增联系人 | `CustomerContactsForm.vue` |
| PUT | `/crm/customer-contacts/update` | `updateCustomerContacts` | 同上 | 修改联系人 | `CustomerContactsForm.vue` |
| DELETE | `/crm/customer-contacts/delete?id=` | `deleteCustomerContacts` | 同上 | 删除联系人 | `crmcustomercontacts/index.vue` |
| GET | `/crm/customer-contacts/get?id=` | `getCustomerContacts` | 同上 | 联系人详情 | `ContactsDetail.vue` |
| GET | `/crm/customer-contacts/page` | `getCustomerContactsPage` | 同上 | 联系人分页 | `crmcustomercontacts/index.vue` |
| GET | `/crm/customer-contacts/export-excel` | 前端 API 定义，后端未找到映射 | — | 导出联系人（**前后端不一致**） | `exportCustomerContacts`，`crmcustomercontacts/index.vue` |
| GET | `/crm/customer-contacts/get-import-template` | 前端 API 定义，后端未找到映射 | — | 导入模板（**前后端不一致**） | `api/crm/crmcustomercontacts` 中定义，调用位置待验证 |


## 4. 商机（`CrmBusinessController`，`/crm/business`）

| HTTP | 路径 | 方法 | Service 方法 | 用途 | 前端调用 |
|------|------|------|--------------|------|----------|
| POST | `/crm/business/create` | `createBusiness` | `CrmBusinessService.createBusiness` | 新增商机（同时保存商机产品） | `BusinessForm.vue` |
| PUT | `/crm/business/update` | `updateBusiness` | `updateBusiness` | 修改商机 | `BusinessForm.vue`、`BusinessDetail.vue` |
| DELETE | `/crm/business/delete?id=` | `deleteBusiness` | `deleteBusiness` | 删除商机 | `crmbusiness/index.vue` |
| GET | `/crm/business/get?id=` | `getBusiness` | `getBusiness` | 商机详情 | `BusinessDetail.vue`、`BusinessForm.vue` |
| GET | `/crm/business/page` | `getBusinessPage` | `getBusinessPage` | 商机分页 | `crmbusiness/index.vue`、`components/business/index.vue` |
| GET | `/crm/business/business-product/list-by-business-id?businessId=` | `getBusinessProductListByBusinessId` | `getBusinessProductListByBusinessId` | 商机产品列表（子表） | `BusinessDetail.vue`、`BusinessProductForm.vue`、`ContractProductForm.vue` |
| GET | `/crm/business/export-excel` | 后端未找到映射 | — | 导出商机（**前后端不一致**） | `BusinessApi.exportBusiness`，前端 API 中存在 |

## 5. 合同（`CrmContractController`，`/crm/contract`）

| HTTP | 路径 | 方法 | Service 方法 | 用途 | 前端调用 |
|------|------|------|--------------|------|----------|
| POST | `/crm/contract/create` | `createContract` | `CrmContractService.createContract` | 新增合同（写入合同产品、生成审批） | `ContractForm.vue`、`ContractForm3.vue` |
| PUT | `/crm/contract/update` | `updateContract` | `updateContract` | 修改合同 | `ContractForm.vue`、`ContractForm3.vue` |
| DELETE | `/crm/contract/delete?id=` | `deleteContract` | `deleteContract` | 删除合同（仅「待审核」状态可删） | `crmcontract/index.vue` |
| GET | `/crm/contract/get?id=` | `getContract` | `getContract` | 合同详情 | `ContractForm.vue`、`ContractPrint.vue` |
| GET | `/crm/contract/page` | `getContractPage` | `getContractPage` | 合同分页 | `crmcontract/index.vue`、`Home/components/contract/index.vue` |
| GET | `/crm/contract/contract-product/list-by-contract-id?contractId=` | `getContractProductListByContractId` | `getContractProductListByContractId` | 合同产品列表（子表） | `ContractProductForm.vue`、`ContractPrint.vue` |
| GET | `/crm/contract/getCode` | `getCode` | `getCode` | 生成合同编号 | `ContractForm.vue`、`ContractForm3.vue` |
| POST | `/crm/contract/check` | `check` | `check(CheckInfoVO)` | 合同审核（同意/拒绝；内部使用 Redisson 锁） | `ContractForm2.vue` |
| GET | `/crm/contract/export-excel` | 后端未找到映射 | — | 导出合同（**前后端不一致**） | `ContractApi.exportContract`，`crmcontract/index.vue` |
| GET | `/crm/index/getCount` | 见第 9 节（首页统计，非合同控制器） | — | 首页合同统计 | `Home/Index.vue`、`Home/wait.vue` |

## 6. 回款（`CrmContractReceivablesController`，`/crm/contract-receivables`）

| HTTP | 路径 | 方法 | Service 方法 | 用途 | 前端调用 |
|------|------|------|--------------|------|----------|
| POST | `/crm/contract-receivables/create` | `createContractReceivables` | `CrmContractReceivablesService.createContractReceivables` | 新增回款（生成审批） | `ContractReceivablesForm.vue` |
| PUT | `/crm/contract-receivables/update` | `updateContractReceivables` | `updateContractReceivables` | 修改回款 | `ContractReceivablesForm*.vue` |
| DELETE | `/crm/contract-receivables/delete?id=` | `deleteContractReceivables` | `deleteContractReceivables` | 删除回款 | `crmcontractreceivables/index.vue` |
| GET | `/crm/contract-receivables/get?id=` | `getContractReceivables` | `getContractReceivables` | 回款详情 | `ContractReceivablesForm*.vue` |
| GET | `/crm/contract-receivables/page` | `getContractReceivablesPage` | `getContractReceivablesPage` | 回款分页 | `crmcontractreceivables/index.vue`、`Home/components/receivables/index.vue`、`crmcustomer/components/receivables/index.vue` |
| GET | `/crm/contract-receivables/getCode` | `getCode` | `getCode` | 生成回款编号 | `ContractReceivablesForm.vue` |
| POST | `/crm/contract-receivables/check` | `check` | `check(CheckInfoVO)` | 回款审核 | `ContractReceivablesForm2.vue` |
| POST | `/crm/order/pay` | 见第 10 节（`PayController`） | — | 回款在线支付（生成二维码） | `ContractReceivablesForm.vue`（`pay({from:'', paytype:'weixin', uni})`） |
| GET | `/crm/contract-receivables/export-excel` | 后端未找到映射 | — | 导出回款（**前后端不一致**） | `ContractReceivablesApi.exportContractReceivables` |

## 7. 审批流程与审批记录

### 7.1 审批流程模板（`CrmFlowController`，`/crm/flow`）

| HTTP | 路径 | 方法 | Service 方法 | 权限 | 用途 | 前端调用 |
|------|------|------|--------------|------|------|----------|
| POST | `/crm/flow/create` | `createFlow` | `CrmFlowService.createFlow` | `crm:flow:create` | 新增审批流程（含步骤） | `FlowForm.vue` |
| PUT | `/crm/flow/update` | `updateFlow` | `updateFlow` | `crm:flow:update` | 修改审批流程 | `FlowForm.vue` |
| DELETE | `/crm/flow/delete?id=` | `deleteFlow` | `deleteFlow` | `crm:flow:delete` | 删除审批流程 | `crmflow/index.vue` |
| GET | `/crm/flow/get?id=` | `getFlow` | `getFlow` | `crm:flow:query` | 审批流程详情 | `FlowForm.vue` |
| GET | `/crm/flow/page` | `getFlowPage` | `getFlowPage` | `crm:flow:query` | 审批流程分页 | `crmflow/index.vue` |
| GET | `/crm/flow/flow-step/list-by-flow-id?flowId=` | `getFlowStepListByFlowId` | `getFlowStepListByFlowId` | `crm:flow:query` | 审批步骤列表 | `FlowStepForm.vue` |
| GET | `/crm/flow/flow-users?flowType=&id=` | `getFlowUserIds` | `getFlowUserIds(flowType, dataId)` | 无 `@PreAuthorize` | 获取某业务单据的审批人（`flowType` 取值 `contract`/`receivables`/`invoice`） | `ContractForm.vue`、`ContractReceivablesForm.vue`、`InvoiceForm.vue` |
| DELETE | `/crm/flow/delete-step?id=` | `deleteFlowStep` | `deleteFlowStep` | 无 `@PreAuthorize` | 删除审批步骤 | `FlowStepForm.vue` |

> 注：前端 `FlowApi.getFlowUserIds(type, id)` 的参数名为 `flowType` 与 `id`，与后端 `@RequestParam` 一致。

### 7.2 审批记录（`CrmFlowLogController`，`/crm/flow-log`）

| HTTP | 路径 | 方法 | Service 方法 | 用途 | 前端调用 |
|------|------|------|--------------|------|----------|
| GET | `/crm/flow-log/page` | `getFlowLogPage` | `CrmFlowLogService.getFlowLogPage` | 审批记录分页（按 `types` 分派到不同 SQL） | `crmflowlog/index.vue`、`index2.vue`、`index3.vue`、`crmcontract/components/log/index.vue`、`crmcontractreceivables/components/log/index.vue`、`crminvoice/log.vue` |
| POST | `/crm/flow-log/examine` | `examine` | `CrmFlowLogService.examine` | 审批通过 / 拒绝（`CheckTypesEnum.AGREE` / `REFUSE`） | `crmflowlog/FlowLogForm.vue` |

> 注：`CrmFlowLogController` 的两个接口未标注 `@PreAuthorize`，`examine` 内部通过「当前用户 + `FlowLogStatusEnum.STATUS_0`」校验审批权限。

## 8. 发票（`CrmInvoiceController`，`/crm/invoice`）

| HTTP | 路径 | 方法 | Service 方法 | 权限 | 用途 | 前端调用 |
|------|------|------|--------------|------|------|----------|
| POST | `/crm/invoice/create` | `createInvoice` | `CrmInvoiceService.createInvoice` | `crm:invoice:create` | 新增开票申请（生成审批） | `InvoiceForm.vue` |
| PUT | `/crm/invoice/update` | `updateInvoice` | `updateInvoice` | `crm:invoice:update` | 修改发票申请 | `InvoiceForm.vue` |
| DELETE | `/crm/invoice/delete?id=` | `deleteInvoice` | `deleteInvoice` | `crm:invoice:delete` | 删除发票 | `crminvoice/index.vue` |
| GET | `/crm/invoice/get?id=` | `getInvoice` | `getInvoice` | `crm:invoice:query` | 发票详情 | `InvoiceForm*.vue` |
| GET | `/crm/invoice/page` | `getInvoicePage` | `getInvoicePage` | `crm:invoice:query` | 发票分页 | `crminvoice/index.vue`、`index2.vue`、`Home/components/invoice/index.vue` |
| POST | `/crm/invoice/check` | `check` | `check(CheckInfoVO)` | 无 `@PreAuthorize` | 发票审核 | `InvoiceForm2.vue` |
| PUT | `/crm/invoice/issue` | `issueInvoice` | `issueInvoice` | 无 `@PreAuthorize` | 开票（状态置为已开票） | `InvoiceForm3.vue` |
| GET | `/crm/invoice/export-excel` | 后端未找到映射 | — | 导出发票（**前后端不一致**） | `InvoiceApi.exportInvoice`，`crminvoice/index.vue` |

## 9. 跟进记录、操作日志、首页统计、业绩

### 9.1 跟进记录（`CrmRecordController`，`/crm/record`）

| HTTP | 路径 | 方法 | 权限 | 用途 | 前端调用 |
|------|------|------|------|------|----------|
| POST | `/crm/record/create` | `createRecord` | `crm:record:create` | 新增跟进记录（`types` 为 customer/business/clues） | `RecordForm.vue`、`RecordForm2.vue`（各业务域） |
| PUT | `/crm/record/update` | `updateRecord` | `crm:record:update` | 修改跟进记录 | 待验证 |
| DELETE | `/crm/record/delete?id=` | `deleteRecord` | `crm:record:delete` | 删除跟进记录 | `crmrecord/index.vue`、`components/record/index.vue` |
| GET | `/crm/record/get?id=` | `getRecord` | `crm:record:query` | 跟进记录详情 | 待验证 |
| GET | `/crm/record/page` | `getRecordPage` | `crm:record:query` | 跟进记录分页 | `crmrecord/index.vue`、`components/record/index.vue` |
| GET | `/crm/record/export-excel` | 后端未找到映射 | — | 导出（**前后端不一致**） | `RecordApi.exportRecord` |

### 9.2 操作日志（`CrmOperatelogController`，`/crm/operatelog`）

| HTTP | 路径 | 方法 | 用途 | 前端调用 |
|------|------|------|------|----------|
| GET | `/crm/operatelog/page` | `getOperatelogPage` | 操作日志分页（**未标注 `@PreAuthorize`**） | `components/log/index.vue`（客户、合同、回款的日志组件） |

> 操作日志的写入通过 `CrmOperatelogService.createLog(title, customerId, contractId, receivableId)`，不对外暴露写接口。

### 9.3 首页统计（`CrmIndexController`，`/crm/index`）

| HTTP | 路径 | 方法 | 用途 | 前端调用 |
|------|------|------|------|----------|
| GET | `/crm/index/getCount?relation=` | `getCount`（`relation` 默认 `my`） | 首页 CRM 统计（`CrmIndexRespVO`） | `Home/Index.vue`、`Home/wait.vue`（通过 `ContractApi.getIndexCount`） |

> 注：前端 `ContractApi.getIndexCount` 定义在合同 API 文件中，但请求的是 `/crm/index/getCount`，并非合同控制器。

### 9.4 业绩分析（`AchievementController`，`/crm/analysis-achievement`）

| HTTP | 路径 | 方法 | Service 方法 | 用途 | 前端调用 |
|------|------|------|--------------|------|----------|
| GET | `/crm/analysis-achievement/sales?relation=` | `getSales`（`relation` 默认 `my`） | `AchievementService.getSales` | 销售简报 | `Home/components/echart/index.vue`（引用 `AchievementApi`，具体方法待验证） |

### 9.5 业绩目标（`CrmAchievementController`，`/crm/achievement`）

| HTTP | 路径 | 方法 | 权限 | 用途 | 前端调用 |
|------|------|------|------|------|----------|
| POST | `/crm/achievement/create` | `createAchievement` | `crm:achievement:create` | 新增业绩目标（部门或员工） | `crmachievement/index.vue`、`AchievementForm*.vue` |

## 10. 在线支付（`PayController`，`/crm/order`）

| HTTP | 路径 | 方法 | 鉴权 | 用途 | 说明 |
|------|------|------|------|------|------|
| POST | `/crm/order/pay` | `pay` | `@PreAuthenticated`（需登录） | 回款在线支付：根据回款编号 `uni` 生成微信/支付宝二维码 | 前端 `ContractReceivablesApi.pay`（`ContractReceivablesForm.vue`） |

| GET/POST（待验证） | `/crm/order/notify/payBack{detailsId}.json` | `payBack` | 免登录（`yshop.security.permit-all_urls` 放行 `/admin-api/crm/order/notify/**`） | 支付渠道异步回调，交由 `PayServiceManager.payBack` 处理 | 成功后经 `PayNoticeConsumer` 消费 MQ 消息，调用 `PayServiceImpl.paySuccess` |


> 注：源码中 `@RequestMapping(value = "/notify/payBack{detailsId}.json")` 的 `{detailsId}` 与 `.json` 之间没有分隔符，实际回调地址与商户后台 `notify_url` 是否一致，**待验证**。
## 11. 前后端不一致汇总（已通过路径差集核实）

以下路径为「前端 `src/api/crm/**/*.ts` 中定义，但后端 CRM Controller 无对应映射」。判定方法：提取前端所有 `/crm/...` 字符串与后端 `@RequestMapping` + `@XxxMapping` 拼接结果做差集。

| 类别 | 前端路径 | 说明 |
|------|----------|------|
| 导出 | `/crm/business/export-excel`、`/crm/clues/export-excel`、`/crm/contract/export-excel`、`/crm/contract-receivables/export-excel`、`/crm/flow/export-excel`、`/crm/invoice/export-excel`、`/crm/record/export-excel`、`/crm/customer-contacts/export-excel`、`/crm/flow-log/export-excel`、`/crm/operatelog/export-excel` | 后端仅 `/crm/customer/export-excel` 实现了导出（`ExcelUtils.write`），其它导出接口未实现，页面按钮若点击会 404（**待验证页面是否有入口**） |
| 线索转客户 | `/crm/clues/createCustomer` | 前端 `CustomerForm.vue` 调用，后端未实现；线索转客户的实际逻辑需确认（**高优先级**） |
| 联系人 | `/crm/customer/customer-contacts/list-by-customer-id`、`/crm/customer-contacts/get-import-template` | 后端未实现；前端仅在 API 文件中定义，页面调用位置待验证 |
| 审批记录 | `/crm/flow-log/create`、`/crm/flow-log/get`、`/crm/flow-log/delete` | 后端 `CrmFlowLogController` 仅有 `page`、`examine`；`create/get/delete` 未实现，前端 `FlowLogApi.createFlowLog`、`getFlowLog`、`deleteFlowLog` 在页面中未见调用（**低风险**） |
| 操作日志 | `/crm/operatelog/create`、`/crm/operatelog/get`、`/crm/operatelog/update`、`/crm/operatelog/delete` | 后端仅 `page`；写入由服务端内部 `createLog` 完成。前端 `OperatelogApi.createOperatelog` 等在页面中未见调用（**低风险**） |

> 反向差异：后端存在但前端 `src/api/crm` 未引用的路径仅 `/crm/analysis-achievement/sales`（销售简报）。该接口可能被其它页面直接调用（待验证）。

## 12. 待验证汇总（接口层面）

1. `/crm/clues/createCustomer` 的线索转客户实现，需确认是否由其它接口或定时任务承担。
2. `/crm/order/notify/payBack{detailsId}.json` 回调地址与第三方商户后台配置是否一致（源码中 `{detailsId}` 无分隔符）。
3. `/crm/achievement/count`、`/crm/achievement/detailCount`、`/crm/analysis-achievement/sales` 在前端的具体调用方法。
4. `customerNum`、`notRecordDay`、`notSuccessDay` 三个 Redis key 的前缀与 `RedisUtil` 实现（未读取 `RedisUtil` 源码）。
5. 历史 SQL 日志中出现过 `/admin-api/crm/analysis-rank/contract`，当前 CRM 源码中无 `Rank*` 类，排行接口是否已删除，待确认。

| PUT | `/crm/achievement/update` | `updateAchievement` | `crm:achievement:update` | 修改业绩目标 | `AchievementForm*.vue` |
| DELETE | `/crm/achievement/delete?id=` | `deleteAchievement` | `crm:achievement:delete` | 删除业绩目标 | 待验证 |
| GET | `/crm/achievement/get?id=` | `getAchievement` | `crm:achievement:query` | 业绩目标详情 | 待验证 |
| GET | `/crm/achievement/depts?type=&year=` | `getDeptsAchievement` | `crm:achievement:query` | 部门业绩列表 | `crmachievement/index.vue` |
| GET | `/crm/achievement/page` | `getAchievementPage` | `crm:achievement:query` | 员工业绩分页 | `crmachievement/index.vue` |
| GET | `/crm/achievement/count?type=&year=&month=` | `getAchievementCount` | 无 `@PreAuthorize` | 业绩统计（首页图表） | `Home/components/echart/index.vue`（**具体方法待验证**） |
| GET | `/crm/achievement/detailCount` | `getAchievementDetailCount` | 无 `@PreAuthorize` | 员工业绩明细 | `crmachievement/index.vue`（待验证） |

