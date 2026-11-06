# 前端模块清单

> 根路径：`yshop-crm-vue/`（Vue 3.4 + Vite 5 + Element Plus 2.6 + Pinia + vue-router 4）。
> 菜单状态以 `yshop-crm/sql/yshop-crm-open-source.sql` 中 `system_menu` 表为准（`deleted` 字段判断是否下线）。

## 1. 目录与职责

| 目录 | 职责 | 关键文件 |
|------|------|----------|
| `src/main.ts`、`src/App.vue` | 应用入口 | — |
| `src/permission.ts` | 路由守卫：token 校验、字典与用户信息加载、动态路由生成、白名单 | 白名单：`/login`、`/social-login`、`/auth-redirect`、`/bind`、`/register`、`/oauthLogin/gitee` |
| `src/router/` | 静态路由与动态路由骨架 | `index.ts`（`createWebHistory`）、`modules/remaining.ts` |
| `src/store/modules/` | Pinia 状态 | `permission.ts`（动态路由）、`user.ts`、`dict.ts`、`tagsView.ts`、`app.ts`、`locale.ts`、`lock.ts`、`simpleWorkflow.ts` |
| `src/utils/routerHelper.ts` | 后端菜单 → 前端路由（`generateRoute`），组件通过 `import.meta.glob('../views/**/*.{vue,tsx}')` 动态加载 | — |
| `src/api/crm/` | CRM 接口封装（12 个文件，见 [page-api-mapping.md](page-api-mapping.md)） | `crmcustomer/index.ts` 等 |
| `src/config/axios/` | axios 实例，`baseURL = VITE_BASE_URL + VITE_API_URL` | `config.ts`、`service.ts` |
| `src/views/crm/` | CRM 页面（70 个 `.vue`） | 见第 3 节 |
| `src/views/Home/` | 首页，包含 CRM 统计组件（合同、回款、发票、业绩图表、待办） | `Index.vue`、`wait.vue`、`components/` |
| `src/views/components/` | 跨业务复用组件（客户选择、商机选择、合同选择、短信/邮件发送表单） | `customer/index.vue`、`business/index.vue`、`contract/index.vue`、`sms/SmsTemplateSendForm.vue`、`email/MailTemplateSendForm.vue` |
| `src/components/` | 全局通用组件 | `Table`、`Search`、`Pagination`、`UploadFile`、`SimpleProcessDesigner`（流程设计）等 |

## 2. 路由与权限

### 2.1 路由生成方式（已确认）

1. 登录后 `permission.ts` 调用 `userStore.setUserInfoAction()` 与 `permissionStore.generateRoutes()`。
2. `generateRoutes()` 从本地缓存 `CACHE_KEY.ROLE_ROUTERS` 读取后端返回的菜单树（`AppCustomRouteRecordRaw[]`）。
3. `generateRoute()` 将菜单的 `component` 字段（如 `crm/crmcustomer/index`）转换为组件路径，由 `registerComponent` 通过 `import.meta.glob` 找到对应 `.vue` 文件。
4. 若菜单 `component` 找不到对应 `.vue`，页面无法加载（**这是前后端菜单与页面必须一致的原因**）。

因此，**CRM 页面的访问入口由数据库菜单决定，而不是前端路由表**。新增 CRM 页面时，必须同时：
- 在 `views/crm/` 下新增 `.vue`；
- 在 `system_menu` 中插入对应菜单（`component` 字段指向该页面，去掉 `.vue` 后缀）；
- 在菜单下插入按钮权限（`crm:xxx:yyy`）。

### 2.2 按钮权限（已确认）

- 使用 `v-hasPermi="['crm:customer:create']"` 指令控制按钮显示（如 `views/crm/crmcustomer/index.vue`）。
- 同一页面中还使用 `scope.row.permission`（`all`/`edit`）控制行级操作（如 `index.vue` 第 225、235、244 行）。
- 后端接口权限使用 `@PreAuthorize("@ss.hasPermission('crm:...')")`，前后端权限标识需保持一致。

## 3. CRM 页面清单（`src/views/crm/`）

### 3.1 菜单已挂载的页面（`system_menu` 中 `deleted = 0`）

| 业务域 | 页面（`component`） | 说明 |
|--------|---------------------|------|
| 客户 | `crm/crmcustomer/index` | 客户列表（私海）|
| 客户 | `crm/crmcustomer/CustomerConfigForm`（菜单 `component` 中存在，是配置弹窗） | 自动公海规则配置（读写 Redis） |
| 客户 | `crm/crmopencustomer/index` | 公海客户 |
| 客户 | `crm/crmrepeatcustomer/index` | 客户查重 |
| 联系人 | `crm/crmcustomercontacts/index` | 联系人 |
| 线索 | `crm/crmclues/index` | 线索列表 |
| 线索 | `crm/crmopenclues/index` | 公海线索 |
| 线索 | `crm/crmcluesrecord/index` | 线索跟进记录 |
| 商机 | `crm/crmbusiness/index` | 商机列表 |
| 商机 | `crm/crmbusinessrecord/index` | 商机跟进记录 |
| 合同 | `crm/crmcontract/index` | 合同列表 |
| 回款 | `crm/crmcontractreceivables/index` | 回款列表 |
| 发票 | `crm/crminvoice/index` | 发票列表 |
| 审批 | `crm/crmflow/index` | 审批流程模板 |
| 审批 | `crm/crmflowlog/index`、`index2`、`index3` | 审批记录（三种视图） |
| 跟进 | `crm/crmrecord/index` | 跟进记录 |
| 业绩 | `crm/crmachievement/index` | 业绩目标 |

### 3.2 已下线（菜单 `deleted = 1`，页面文件仍存在或已删除）

| 页面 | 状态 | 说明 |
|------|------|------|
| `crm/crmcontract/index2` | 菜单已删除，**文件仍在** | 历史版本「全部合同」 |
| `crm/crmcontractreceivables/index2` | 菜单已删除，**文件仍在** | 历史版本 |
| `crm/crminvoice/index2` | 菜单已删除，**文件仍在** | 历史版本 |
| `crm/analysis/*`（业绩/客户/排行分析等 9 个） | 菜单已删除，**`views/crm/analysis/` 目录不存在** | 排行与分析页面已移除 |
| 云呼（`crmcloudcall`、`crmcloudcalllog`） | 菜单已删除，**页面不存在** | 第三方功能已下线 |
| 钉钉（`crmdingconfig`、`crmdingcustomer`、`crmdinguser`） | 菜单已删除，**页面不存在** | 第三方功能已下线 |
| 企微（`crmworkwxconfig`、`crmworkwxcustomer`、`crmworkwxuser`） | 菜单已删除，**页面不存在** | 第三方功能已下线 |

> 结论：README 中列出的「企微、云呼、钉钉」功能，在当前数据库菜单中均已标记删除，前端页面已不存在。

### 3.3 仅作为组件 / 弹窗使用的页面（非菜单页）

| 页面 | 使用方式 |
|------|----------|
| `crmcontract/ContractForm`、`ContractForm2`、`ContractForm3` | 合同新增/审核/修改表单（`ContractForm2` 为审核弹窗，调用 `checkContract`，见 [page-api-mapping.md](page-api-mapping.md)） |
| `crmcontract/ContractPrint` | 合同打印 |
| `crmcontract/components/*` | 合同产品、审批日志、回款列表子组件 |
| `crmcustomer/CustomerDetail`、`CustomerForm` | 客户详情、表单 |
| `crmcustomer/components/*` | 客户详情页的子表：商机、合同、回款、联系人、跟进、操作日志、地图 |
| `crmclues/components/customer/CustomerForm` | 线索转客户表单（依赖未实现的后端接口，见第 4 节） |
| `crminvoice/InvoiceForm`、`InvoiceForm2`、`InvoiceForm3`、`log` | 发票申请、审核、开票、日志 |
| 其它 `*Detail.vue`、`*Form.vue`、`*RecordForm*.vue` | 各业务的详情与表单 |

### 3.4 前端 `views/crm` 与 `system_menu` 的差异

- `views/crm/crmcontract/index2.vue`、`crmcontractreceivables/index2.vue`、`crminvoice/index2.vue` 在仓库中仍存在，但菜单已删除，**属于历史遗留文件**，修改前请先确认是否仍需要保留。
- `views/crm/crmcustomer/components/*` 等为子组件，不对应菜单。

## 4. 已知问题与待验证

1. 线索转客户：`views/crm/crmclues/components/customer/CustomerForm.vue` 调用 `CluesApi.createCustomer`，对应后端接口未实现（见 [../backend/api-inventory.md](../backend/api-inventory.md) 第 11 节）。
2. 排行/分析页面：`system_menu` 中存在 `crm/analysis/contractRank`、`crm/analysis/receivablesRank` 等记录但已标记删除，前端对应目录不存在，无需处理，但需确认是否还有残留调用。
3. 首页（`views/Home/`）中 `components/echart` 引用 `AchievementApi`，具体调用方法待验证。
