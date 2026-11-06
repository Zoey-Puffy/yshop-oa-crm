# 页面与接口对应关系（前端 → 后端）

> 数据来源：`yshop-crm-vue/src/api/crm/**/*.ts`（API 方法定义）与 `src/views/**/*.vue`（实际调用）通过全量比对得到。
> 「页面」路径均相对于 `yshop-crm-vue/src/`。后端路径为 `/admin-api` + 所列路径。
> 标注 **⚠ 后端缺失** 的接口，表示 API 文件中有定义，但 CRM 后端 Controller 中没有对应映射（详见 [../backend/api-inventory.md](../backend/api-inventory.md) 第 11 节）。

## 1. API 文件清单

| API 文件 | 导出对象 | 后端 Controller 前缀 |
|----------|----------|----------------------|
| `api/crm/crmachievement/index.ts` | `AchievementApi` | `/crm/achievement` |
| `api/crm/crmbusiness/index.ts` | `BusinessApi` | `/crm/business` |
| `api/crm/crmclues/index.ts` | `CluesApi` | `/crm/clues` |
| `api/crm/crmcontract/index.ts` | `ContractApi` | `/crm/contract`（另含 `getIndexCount` → `/crm/index/getCount`） |
| `api/crm/crmcontractreceivables/index.ts` | `ContractReceivablesApi` | `/crm/contract-receivables`（另含 `pay` → `/crm/order/pay`） |
| `api/crm/crmcustomer/index.ts` | `CustomerApi` | `/crm/customer` |
| `api/crm/crmcustomercontacts/index.ts` | `CustomerContactsApi` | `/crm/customer-contacts` |
| `api/crm/crmflow/index.ts` | `FlowApi` | `/crm/flow` |
| `api/crm/crmflowlog/index.ts` | `FlowLogApi` | `/crm/flow-log` |
| `api/crm/crminvoice/index.ts` | `InvoiceApi` | `/crm/invoice` |
| `api/crm/crmoperatelog/index.ts` | `OperatelogApi` | `/crm/operatelog` |
| `api/crm/crmrecord/index.ts` | `RecordApi` | `/crm/record` |

> 另外，`api/crm` 目录外还有 `views/system/operatelog/index.vue` 调用 `OperateLogApi.exportOperateLog` 与 `OperatelogApi.getOperatelogPage`，属于系统模块页面，不在 CRM 范围内。

## 2. 商机（`BusinessApi`）

| 方法 | 后端接口 | 调用页面 |
|------|----------|----------|
| `getBusinessPage` | `GET /crm/business/page` | `views/crm/crmbusiness/index.vue`、`views/components/business/index.vue`（通用选择器）、`views/crm/crmcustomer/components/business/index.vue` |
| `getBusiness` | `GET /crm/business/get` | `views/crm/crmbusiness/BusinessDetail.vue`、`BusinessForm.vue`、`views/crm/crmbusinessrecord/RecordForm.vue` |
| `createBusiness` | `POST /crm/business/create` | `views/crm/crmbusiness/BusinessForm.vue` |
| `updateBusiness` | `PUT /crm/business/update` | `BusinessDetail.vue`、`BusinessForm.vue` |
| `deleteBusiness` | `DELETE /crm/business/delete` | `views/crm/crmbusiness/index.vue` |
| `getBusinessProductListByBusinessId` | `GET /crm/business/business-product/list-by-business-id` | `BusinessDetail.vue`、`crmbusiness/components/BusinessProductForm.vue`、`crmbusinessrecord/RecordForm.vue`、`crmcontract/components/ContractProductForm.vue`、`ContractProductForm2.vue` |
| `exportBusiness` | `GET /crm/business/export-excel` | ⚠ 后端缺失；前端页面未见调用 |

## 3. 线索（`CluesApi`）

| 方法 | 后端接口 | 调用页面 |
|------|----------|----------|
| `getCluesPage` | `GET /crm/clues/page` | `views/crm/crmclues/index.vue`、`views/crm/crmopenclues/index.vue`（公海） |

## 4. 合同（`ContractApi`）

| 方法 | 后端接口 | 调用页面 |
|------|----------|----------|
| `getContractPage` | `GET /crm/contract/page` | `crmcontract/index.vue`、`crmcontract/index2.vue`（菜单已删除）、`views/components/contract/index.vue`（通用选择器）、`crmcustomer/components/contract/index.vue`、`Home/components/contract/index.vue` |
| `getContract` | `GET /crm/contract/get` | `crmcontract/ContractForm.vue`、`ContractForm2.vue`、`ContractForm3.vue` |
| `createContract` | `POST /crm/contract/create` | `crmcontract/ContractForm.vue`、`ContractForm3.vue` |
| `updateContract` | `PUT /crm/contract/update` | `crmcontract/ContractForm.vue`、`ContractForm3.vue` |
| `deleteContract` | `DELETE /crm/contract/delete` | `crmcontract/index.vue`、`index2.vue` |
| `checkContract` | `POST /crm/contract/check` | `crmcontract/ContractForm2.vue`（审核弹窗；后端方法标注 `@Deprecated`，见 [../backend/core-business-flow.md](../backend/core-business-flow.md) 第 5.3 节） |
| `getContractNo` | `GET /crm/contract/getCode` | `crmcontract/ContractForm.vue`、`ContractForm3.vue` |
| `getContractProductListByContractId` | `GET /crm/contract/contract-product/list-by-contract-id` | `crmcontract/components/ContractProductForm.vue`、`ContractProductForm2.vue`、`crmcontract/ContractPrint.vue` |

## 5. 回款（`ContractReceivablesApi`）

| 方法 | 后端接口 | 调用页面 |
|------|----------|----------|
| `getContractReceivablesPage` | `GET /crm/contract-receivables/page` | `crmcontractreceivables/index.vue`、`index2.vue`（菜单已删除）、`crmcontract/components/receivables/index.vue`、`crmcustomer/components/receivables/index.vue`、`Home/components/receivables/index.vue` |
| `getContractReceivables` | `GET /crm/contract-receivables/get` | `crmcontractreceivables/ContractReceivablesForm.vue`、`ContractReceivablesForm2.vue`、`ContractReceivablesForm3.vue` |
| `createContractReceivables` | `POST /crm/contract-receivables/create` | `crmcontractreceivables/ContractReceivablesForm.vue` |
| `updateContractReceivables` | `PUT /crm/contract-receivables/update` | `crmcontractreceivables/ContractReceivablesForm.vue` |
| `deleteContractReceivables` | `DELETE /crm/contract-receivables/delete` | `crmcontractreceivables/index.vue`、`index2.vue` |
| `checkContract` | `POST /crm/contract-receivables/check` | `crmcontractreceivables/ContractReceivablesForm2.vue` |
| `getCode` | `GET /crm/contract-receivables/getCode` | `crmcontractreceivables/ContractReceivablesForm.vue` |

## 6. 客户（`CustomerApi`）与联系人（`CustomerContactsApi`）

### 6.1 客户

| 方法 | 后端接口 | 调用页面 |
|------|----------|----------|
| `getCustomerPage` | `GET /crm/customer/page` | `views/components/customer/index.vue`（通用选择器）、`crmopencustomer/index.vue`（公海客户） |
| `getCustomerPage2` | `GET /crm/customer/page2` | `crmcustomer/index.vue`（私海列表）、`crmrepeatcustomer/index.vue`（客户查重） |
| `getCustomer` | `GET /crm/customer/get` | `crmcustomer/CustomerDetail.vue`、`CustomerForm.vue`、`crmbusiness/BusinessDetail.vue`、`crmbusinessrecord/RecordForm.vue`、`crmrecord/RecordForm.vue` |
| `createCustomer` | `POST /crm/customer/create` | `crmcustomer/CustomerForm.vue` |
| `updateCustomer` | `PUT /crm/customer/update` | `crmcustomer/CustomerForm.vue` |
| `deleteCustomer` | `DELETE /crm/customer/delete` | `crmcustomer/index.vue` |
| `openCustomer` | `GET /crm/customer/do-open` | `crmcustomer/CustomerDetail.vue`、`crmcustomer/index.vue`（放入公海） |
| `receCustomer` | `GET /crm/customer/receive` | `crmopencustomer/index.vue`（领取公海客户） |
| `saveRedisSet` | `POST /crm/customer/saveRedisSet` | `crmcustomer/CustomerConfigForm.vue`（自动公海规则） |
| `getRedisSet` | `GET /crm/customer/getRedisSet` | `crmcustomer/CustomerConfigForm.vue` |
| `sendSms` | `POST /crm/customer/send-sms` | `views/components/sms/SmsTemplateSendForm.vue` |
| `sendMail` | `POST /crm/customer/send-mail` | `views/components/email/MailTemplateSendForm.vue` |
| `exportCustomer` | `GET /crm/customer/export-excel` | 后端已实现（`exportCustomerExcel`）；前端 API 中的调用位置待验证 |
| `getCustomerContactsListByCustomerId` | `GET /crm/customer/customer-contacts/list-by-customer-id` | ⚠ 后端缺失；页面调用待验证 |

### 6.2 联系人

| 方法 | 后端接口 | 调用页面 |
|------|----------|----------|
| `getCustomerContactsPage` | `GET /crm/customer-contacts/page` | `crmcustomercontacts/index.vue`、`crmcustomer/components/contacts/index.vue`、`crmcontract/ContractForm.vue`、`ContractForm2.vue`、`ContractForm3.vue`（选择签约人） |
| `getCustomerContacts` | `GET /crm/customer-contacts/get` | `crmcustomercontacts/ContactsDetail.vue`、`CustomerContactsForm.vue`、`crmcustomer/components/contacts/CustomerContactsForm.vue` |

## 7. 审批（`FlowApi`、`FlowLogApi`）与发票（`InvoiceApi`）

### 7.1 审批流程模板（`FlowApi`）

| 方法 | 后端接口 | 调用页面 |
|------|----------|----------|
| `getFlowPage` | `GET /crm/flow/page` | `crmflow/index.vue` |
| `getFlow` | `GET /crm/flow/get` | `crmflow/FlowForm.vue` |
| `createFlow` | `POST /crm/flow/create` | `crmflow/FlowForm.vue` |
| `updateFlow` | `PUT /crm/flow/update` | `crmflow/FlowForm.vue` |
| `deleteFlow` | `DELETE /crm/flow/delete` | `crmflow/index.vue` |
| `exportFlow` | `GET /crm/flow/export-excel` | ⚠ 后端缺失；`crmflow/index.vue` 调用 |
| `getFlowStepListByFlowId` | `GET /crm/flow/flow-step/list-by-flow-id` | `crmflow/components/FlowStepForm.vue` |
| `deleteFlowStep` | `DELETE /crm/flow/delete-step` | `crmflow/components/FlowStepForm.vue` |
| `getFlowUserIds` | `GET /crm/flow/flow-users` | `crmcontract/ContractForm.vue`、`ContractForm3.vue`；`crmcontractreceivables/ContractReceivablesForm.vue`、`ContractReceivablesForm3.vue`；`crminvoice/InvoiceForm.vue`、`InvoiceForm2.vue`、`InvoiceForm3.vue`（获取审批人，参数 `flowType` 为 `contract` / `receivables` / `invoice`） |

### 7.2 审批记录（`FlowLogApi`）

| 方法 | 后端接口 | 调用页面 |
|------|----------|----------|
| `getFlowLogPage` | `GET /crm/flow-log/page` | `crmflowlog/index.vue`、`index2.vue`、`index3.vue`（三个审批记录视图，菜单均为 active）；`crmcontract/components/log/index.vue`；`crmcontractreceivables/components/log/index.vue`；`crminvoice/log.vue` |
| `examine` | `POST /crm/flow-log/examine` | `crmflowlog/FlowLogForm.vue`（待办审批的同意/拒绝） |
| `createFlowLog` | `POST /crm/flow-log/create` | ⚠ 后端缺失；页面调用未见 |
| `getFlowLog` | `GET /crm/flow-log/get` | ⚠ 后端缺失；页面调用未见 |
| `deleteFlowLog` | `DELETE /crm/flow-log/delete` | ⚠ 后端缺失；页面调用未见 |
| `exportFlowLog` | `GET /crm/flow-log/export-excel` | ⚠ 后端缺失；页面调用未见 |

### 7.3 发票（`InvoiceApi`）

| 方法 | 后端接口 | 调用页面 |
|------|----------|----------|
| `getInvoicePage` | `GET /crm/invoice/page` | `crminvoice/index.vue`、`crminvoice/index2.vue`（菜单已删除）、`views/Home/components/invoice/index.vue`（首页） |

## 8. 跟进记录（`RecordApi`）、操作日志（`OperatelogApi`）与业绩（`AchievementApi`）

### 8.1 跟进记录（`RecordApi`）

| 方法 | 后端接口 | 调用页面 |
|------|----------|----------|
| `getRecordPage` | `GET /crm/record/page` | `crmrecord/index.vue`、`crmcluesrecord/index.vue`、`crmbusinessrecord/index.vue`；组件 `crmclues/components/record/index.vue`、`crmbusiness/components/record/index.vue`、`crmcustomer/components/record/index.vue` |
| `createRecord` | `POST /crm/record/create` | `crmrecord/RecordForm.vue`、`RecordForm2.vue`；`crmcluesrecord/RecordForm.vue`、`RecordForm2.vue`；`crmbusinessrecord/RecordForm.vue`、`RecordForm2.vue` |
| `deleteRecord` | `DELETE /crm/record/delete` | `crmrecord/index.vue`、`crmcluesrecord/index.vue`、`crmbusinessrecord/index.vue`；组件 `crmclues/components/record/index.vue`、`crmbusiness/components/record/index.vue`、`crmcustomer/components/record/index.vue` |
| `exportRecord` | `GET /crm/record/export-excel` | ⚠ 后端缺失；上述列表页与组件调用 |
| `updateRecord`、`getRecord` | `PUT /crm/record/update`、`GET /crm/record/get` | 页面调用未见（**待验证**） |

### 8.2 操作日志（`OperatelogApi`）

| 方法 | 后端接口 | 调用页面 |
|------|----------|----------|
| `getOperatelogPage` | `GET /crm/operatelog/page` | `crmcustomer/components/log/index.vue`（客户日志）；`views/system/operatelog/index.vue`（系统模块，**非 CRM 页面**） |
| `createOperatelog`、`updateOperatelog`、`getOperatelog`、`deleteOperatelog`、`exportOperatelog` | — | ⚠ 后端缺失（写入由服务端 `createLog` 完成）；页面调用未见 |

### 8.3 业绩（`AchievementApi`）

| 方法 | 后端接口 | 调用页面 |
|------|----------|----------|
| `getAchievementPage` | `GET /crm/achievement/page` | `crmachievement/index.vue` |
| `getAchievementDept` | `GET /crm/achievement/depts` | `crmachievement/index.vue` |
| `createAchievement` | `POST /crm/achievement/create` | `crmachievement/AchievementForm.vue`、`AchievementForm2.vue`、`index.vue` |
| `achievementCount` | `GET /crm/achievement/count` | `views/Home/components/echart/index.vue`（首页图表） |

## 9. 前后端不一致汇总（页面层）

| 类别 | 接口 | 页面影响 | 风险 |
|------|------|----------|------|
| 线索转客户 | `POST /crm/clues/createCustomer` | `crmclues/components/customer/CustomerForm.vue` 调用，后端无实现 | **高**：线索转客户功能可能不可用 |
| 导出 | 商机、线索、合同、回款、发票、跟进、流程、联系人、审批记录 的 `export-excel` | 列表页存在导出按钮时点击会失败 | 中：需逐页确认按钮是否可见 |
| 审批记录 | `flow-log/create|get|delete` | 页面未见调用 | 低 |
| 操作日志 | `operatelog/create|update|get|delete` | 页面未见调用 | 低 |
| 联系人 | `customer/customer-contacts/list-by-customer-id`、`customer-contacts/get-import-template` | 页面调用待验证 | 低 |
| 已删除菜单页 | `crmcontract/index2`、`crmcontractreceivables/index2`、`crminvoice/index2` | 菜单已删除，但页面仍调用 API | 低：历史遗留，未挂载菜单 |

> 页面层调用的统计以 `Select-String` 的正则匹配为准，动态拼接的方法名（如 `Api[name]()`）无法被统计，若存在则需人工确认。

| `getInvoice` | `GET /crm/invoice/get` | `crminvoice/InvoiceForm.vue`、`InvoiceForm2.vue`、`InvoiceForm3.vue` |
| `createInvoice` | `POST /crm/invoice/create` | `crminvoice/InvoiceForm.vue` |
| `updateInvoice` | `PUT /crm/invoice/update` | `crminvoice/InvoiceForm.vue` |
| `deleteInvoice` | `DELETE /crm/invoice/delete` | `crminvoice/index.vue`、`index2.vue` |
| `checkInvoice` | `POST /crm/invoice/check` | `crminvoice/InvoiceForm2.vue`（审核） |
| `issueInvoice` | `PUT /crm/invoice/issue` | `crminvoice/InvoiceForm3.vue`（开票） |
| `exportInvoice` | `GET /crm/invoice/export-excel` | ⚠ 后端缺失；`crminvoice/index.vue`、`index2.vue` 调用 |

| `createCustomerContacts` | `POST /crm/customer-contacts/create` | `crmcustomercontacts/CustomerContactsForm.vue`、`crmcustomer/components/contacts/CustomerContactsForm.vue` |
| `updateCustomerContacts` | `PUT /crm/customer-contacts/update` | 同上 |
| `deleteCustomerContacts` | `DELETE /crm/customer-contacts/delete` | `crmcustomercontacts/index.vue`、`crmcustomer/components/contacts/index.vue` |
| `exportCustomerContacts` | `GET /crm/customer-contacts/export-excel` | ⚠ 后端缺失；`crmcustomercontacts/index.vue`、`crmcustomer/components/contacts/index.vue` 调用 |
| `importTemplate` | `GET /crm/customer-contacts/get-import-template` | ⚠ 后端缺失；API 中定义，页面调用待验证 |

| `pay` | `POST /crm/order/pay` | `crmcontractreceivables/ContractReceivablesForm.vue`（微信扫码，`paytype: 'weixin'`） |
| `exportContractReceivables` | `GET /crm/contract-receivables/export-excel` | ⚠ 后端缺失；`crmcontractreceivables/index.vue`、`index2.vue` 调用 |

| `getIndexCount` | `GET /crm/index/getCount` | `views/Home/Index.vue`、`views/Home/wait.vue`（首页统计，接口路径不属于合同控制器） |
| `exportContract` | `GET /crm/contract/export-excel` | ⚠ 后端缺失；`crmcontract/index.vue`、`index2.vue` 调用 |

| `getClues` | `GET /crm/clues/get` | `crmclues/CluesDetail.vue`、`CluesForm.vue`、`crmclues/components/customer/CustomerForm.vue`、`crmcluesrecord/RecordForm.vue`、`crmopenclues/CluesForm.vue` |
| `createClues` | `POST /crm/clues/create` | `crmclues/CluesForm.vue`、`crmopenclues/CluesForm.vue` |
| `updateClues` | `PUT /crm/clues/update` | `crmclues/CluesDetail.vue`、`CluesForm.vue`、`crmopenclues/CluesForm.vue` |
| `deleteClues` | `DELETE /crm/clues/delete` | `views/crm/crmclues/index.vue` |
| `openClues` | `GET /crm/clues/open` | `crmclues/CluesDetail.vue`、`crmclues/index.vue`（放入公海） |
| `receClues` | `GET /crm/clues/receive` | `views/crm/crmopenclues/index.vue`（领取） |
| `createCustomer` | `POST /crm/clues/createCustomer` | `crmclues/components/customer/CustomerForm.vue`（线索转客户）⚠ 后端缺失 |
| `exportClues` | `GET /crm/clues/export-excel` | ⚠ 后端缺失；前端页面未见调用 |
