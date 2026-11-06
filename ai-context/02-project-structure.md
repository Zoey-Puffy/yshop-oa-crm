# 02 项目目录结构

> 路径均相对于工作区根目录 `d:\workspace\yshop-oa-crm`。

## 1. 顶层目录

| 路径 | 说明 |
|------|------|
| `yshop-crm/` | 后端 Maven 多模块工程（Java 17 / Spring Boot 3.2.2） |
| `yshop-crm-vue/` | 后台前端 Vue 3 + Vite 工程 |
| `yshop-crm/sql/` | 数据库初始化脚本与增量脚本 |
| `yshop-crm/script/` | Docker、部署脚本 |
| `ai-content/memory-bank/` | 已有的开发记忆文件（模板） |
| `.clinerules/` | 已有的 Cline 规则文件 |
| `assets/` | README 引用的图片资源 |
| `README.md`、`LICENSE`、`.gitignore`、`.clineignore` | 项目说明与忽略规则 |

## 2. 后端 `yshop-crm/`

### 2.1 Maven 模块

父 POM：`yshop-crm/pom.xml`，`<modules>` 如下：

| 模块 | 说明 |
|------|------|
| `yshop-dependencies` | 统一依赖版本（BOM） |
| `yshop-framework` | 框架层，含 `yshop-common` 与 14 个 `yshop-spring-boot-starter-*` |
| `yshop-server` | 启动入口（容器），聚合各业务模块 |
| `yshop-module-system` | 系统管理（用户、角色、菜单、租户、字典、OAuth2 等） |
| `yshop-module-infra` | 基础设施（代码生成、文件、定时任务、API 日志等） |
| `yshop-module-pay` | 支付（商户、渠道配置） |
| `yshop-module-mp` | 微信公众号 |
| `yshop-module-mall` | 商城（`yshop-module-product-biz`、`yshop-module-shop-biz` 等） |
| `yshop-module-message` | 消息（站内信、微信模板消息） |
| `yshop-module-crm` | **CRM 业务模块（本知识库重点）** |

每个业务模块下通常分为 `*-api`（对外接口、枚举、DTO）与 `*-biz`（业务实现）。

### 2.2 CRM 模块（重点）

```
yshop-crm/yshop-module-crm/
├── pom.xml                                  # 聚合 *-api、*-biz
├── yshop-module-crm-api/                    # 对外枚举、错误码
│   └── src/main/java/co/yixiang/yshop/module/crm/enums/
│       ├── ErrorCodeConstants.java          # 错误码 200000~200017
│       ├── TypesEnum.java                   # 业务类型：customer/business/clues/contract/receivables/invoice
│       ├── ContractStatusEnum.java          # 审核状态 0~3
│       ├── CluesStatusEnum.java             # 线索状态 -1/0/1
│       ├── CustomerTypesEnum.java           # open(公海)/private(私有)
│       ├── FlowStepEnum.java                # 审批步骤类型 2一人或签 / 3多人会签
│       └── ...                              # 其它枚举
└── yshop-module-crm-biz/
    ├── pom.xml
    └── src/main/
        ├── java/co/yixiang/yshop/module/crm/
        │   ├── controller/admin/            # 管理后台 REST 接口（85 个 Java 文件，按业务域分包）
        │   ├── service/                     # 业务接口与实现（每域 *Service + *ServiceImpl）
        │   ├── dal/dataobject/              # 16 个 DO 实体（表映射）
        │   ├── dal/mysql/                   # 16 个 Mapper 接口（MyBatis-Plus BaseMapperX）
        │   ├── job/                         # Quartz 定时任务 Handler（2 个）
        │   └── mq/consumer/                 # Redis Stream 消费者（1 个）
        └── resources/
            └── mapper/crmcustomer/CrmCustomerMapper.xml   # 客户自定义 SQL
```

> 注：CRM 模块 `controller/admin/` 下共 15 个业务子包（`crmachievement`、`crmanalysis`、`crmbusiness`、`crmclues`、`crmcontract`、`crmcontractreceivables`、`crmcustomer`、`crmcustomercontacts`、`crmflow`、`crmflowlog`、`crmindex`、`crminvoice`、`crmoperatelog`、`crmpay`、`crmrecord`），共 85 个 Java 文件（含 VO）。

### 2.3 启动入口与配置

| 路径 | 说明 |
|------|------|
| `yshop-crm/yshop-server/src/main/java/co/yixiang/yshop/server/YshopServerApplication.java` | Spring Boot 启动类 |
| `yshop-crm/yshop-server/src/main/resources/application.yaml` | 公共配置（端口、MyBatis-Plus、Swagger、租户忽略路径、安全放行路径等） |
| `yshop-crm/yshop-server/src/main/resources/application-local.yaml` | 本地环境（端口 48081、本地 MySQL/Redis、Quartz 默认关闭） |
| `yshop-crm/yshop-server/src/main/resources/application-dev.yaml` | 开发环境配置 |
| `yshop-crm/yshop-server/src/main/resources/logback-spring.xml` | 日志配置 |
| `yshop-crm/yshop-server/Dockerfile` | 容器构建 |

> ⚠️ 上述 YAML 中包含数据库、Redis、微信、钉钉、企业微信、地图等**敏感配置**，本知识库不记录其值。修改或新建环境时请使用独立的本地配置，不要提交真实密钥。

### 2.4 数据库脚本

| 路径 | 说明 |
|------|------|
| `yshop-crm/sql/yshop-crm-open-source.sql` | 全量初始化脚本（约 11.8 MB，含表结构、菜单、字典、示例数据） |
| `yshop-crm/sql/migration/20261008_p0_crm_business_indexes.sql` | CRM 核心表组合索引（增量脚本） |

### 2.5 测试

| 路径 | 说明 |
|------|------|
| `yshop-crm/yshop-framework/*/src/test/` | 框架层单元测试（data-permission、ip、excel、web、common） |
| `yshop-crm/yshop-module-system/yshop-module-system-biz/src/test/` | System 模块单元测试（约 40 个类） |
| `yshop-crm/yshop-module-mall/yshop-module-shop-biz/src/test/` | 商城 material 测试 |
| `yshop-crm/yshop-server/src/test/java/co/yixiang/yshop/ProjectReactor.java` | 项目重命名工具（`main` 方法，非单元测试） |

> CRM 模块 **无** `src/test` 目录，详见 [testing/existing-tests.md](testing/existing-tests.md)。

### 2.6 脚本

| 路径 | 说明 |
|------|------|
| `yshop-crm/script/docker/docker-compose.yml`、`docker.env` | Docker 编排（`docker.env` 含敏感信息，勿外传） |
| `yshop-crm/script/shell/deploy.sh` | 部署脚本 |

## 3. 前端 `yshop-crm-vue/`

```
yshop-crm-vue/
├── package.json                 # 依赖与 scripts（pnpm）
├── vite.config.ts               # Vite 配置，server.port 读取 VITE_PORT
├── .env / .env.dev / .env.local / .env.prod / .env.stage / .env.test   # 环境变量（含 API 前缀，勿提交密钥）
├── src/
│   ├── main.ts                  # 入口
│   ├── permission.ts            # 路由守卫（token、字典、用户信息、动态路由）
│   ├── api/                     # 接口封装，按业务域分目录（crm、system、infra、mp、mall ...）
│   │   └── crm/                 # 12 个 CRM API 文件（见 frontend/page-api-mapping.md）
│   ├── views/
│   │   ├── crm/                 # CRM 页面（70 个 .vue）
│   │   ├── system/ infra/ mall/ mp/ message/ ...   # 其它模块页面
│   │   └── Home/                # 首页（含 CRM 统计组件）
│   ├── router/                  # 静态路由 + 动态路由（modules/remaining.ts）
│   ├── store/modules/           # Pinia 模块（permission、user、dict、tagsView ...）
│   ├── config/axios/            # axios 封装（baseURL = VITE_BASE_URL + VITE_API_URL）
│   ├── components/              # 公共组件
│   ├── layout/                  # 布局
│   ├── utils/routerHelper.ts    # 后端菜单 → 前端路由 的转换
│   ├── locales/                 # 国际化
│   └── types/                   # 全局类型
└── types/                       # 全局 d.ts（components、router、env 等）
```

## 4. 文档与配置文件检索优先级

按需定位时，建议的检索顺序：

1. 本知识库索引文档（`ai-context/`）。
2. 路径规则：CRM 后端 `yshop-crm/yshop-module-crm/yshop-module-crm-biz/src/main/java/co/yixiang/yshop/module/crm/`；前端 `yshop-crm-vue/src/views/crm/`、`yshop-crm-vue/src/api/crm/`。
3. 排除目录：`target/`、`node_modules/`、`.idea/`、`.git/`、`dist/`（与 `.clineignore` 一致）。
