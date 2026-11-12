# 项目启动指南

> 依据：`README.md`、`yshop-crm/pom.xml`、`yshop-crm/yshop-server/src/main/resources/application*.yaml`、`yshop-crm-vue/package.json`、`yshop-crm-vue/.env*`、`yshop-crm/script/` 下的部署文件。
> 配置中的账号、密码、密钥均**不在本文档中记录**，启动前请在本地自行核对配置。

## 1. 环境要求

| 依赖 | 要求版本 | 实际安装版本 | 来源 |
|------|----------|--------------|------|
| JDK | **17**（父 POM `java.version`） | **17.0.12** ✅ | `yshop-crm/pom.xml` |
| Maven | 3.8+ | **3.9.11** ✅（IntelliJ 内置） | `README.md` |
| MySQL | 8 | **8.0.45** ✅ | `README.md`、`docker-compose.yml`（`mysql:8`） |
| Redis | 6+ | **Docker redis:6-alpine**（需启动容器） | `README.md`、`docker-compose.yml`（`redis:6-alpine`） |
| Node.js | 16+（`engines.node >= 16.0.0`） | **24.16.0** ✅ | `yshop-crm-vue/package.json` |
| pnpm | 8.6+（`engines.pnpm >= 8.6.0`） | **12.18.0** ✅ | `yshop-crm-vue/package.json` |
| Docker Desktop | — | **29.9.3** ✅（用于运行 Redis） | — |
| IDE | 后端 IntelliJ IDEA；前端 VS Code | **IntelliJ IDEA 2026.1** ✅ | `README.md` |

> **Maven 说明**：Maven CLI 通过 IntelliJ IDEA 内置 Maven（`C:\Program Files\JetBrains\IntelliJ IDEA 2026.1\plugins\maven\lib\maven3\bin`）添加到系统 PATH，使用 `mvn` 命令即可调用。单独打开新终端后需确保 PATH 已刷新。

> **环境变量配置**
> - `JAVA_HOME = C:\Program Files\Java\jdk-17`（已配置 ✅）
> - Maven 路径已追加到用户 PATH（持久化），新终端自动生效

> ⚠ **版本差异**：`yshop-crm/yshop-server/Dockerfile` 基础镜像为 `eclipse-temurin:21-jre`（JRE 21），与开发环境要求的 JDK 17 不一致。Docker 部署与本地开发的 JDK 版本需要统一确认（**待验证**）。

## 2. 后端启动

### 2.1 初始化数据库（已确认）

1. 创建数据库，例如 `yshop-crm-open-source`（本地配置 `application-local.yaml` 使用的库名）。
2. 导入 `yshop-crm/sql/yshop-crm-open-source.sql`（约 11.8 MB，包含表结构与示例数据）。
3. 增量索引脚本 `yshop-crm/sql/migration/20261008_p0_crm_business_indexes.sql` 不包含在全量脚本中，是否已在目标库执行需要自行确认（**待验证**）。

> 注：`application-dev.yaml` 指向的库名为 `yshop_crm`，与本地库名不同，请按所用 profile 创建对应库。

### 2.2 修改配置（已确认）

配置文件位于 `yshop-crm/yshop-server/src/main/resources/`：

| 文件 | 说明 |
|------|------|
| `application.yaml` | 公共配置；`spring.profiles.active: local`（默认激活 local） |
| `application-local.yaml` | 本地：端口 `48081`，MySQL 地址 `127.0.0.1:3306`，Redis `127.0.0.1:6379` database `0`，Quartz `auto-startup: true` |
| `application-dev.yaml` | 开发/测试：端口 `48081`，MySQL 库 `yshop_crm`，Redis database `2`，Quartz `auto-startup: false` |

需要按本机环境修改的项（只列键名，不记录值）：
- `spring.datasource.dynamic.datasource.master.url` / `username` / `password`
- `spring.data.redis.host` / `port` / `password` / `database`
- `wx.mp.*`、`wx.miniapp.*`（微信配置，本地开发可使用自己的测试号）
- `justauth.type.*`（钉钉、企业微信、小程序、公众号的 OAuth 配置）
- `yshop.pay.order-notify-url`、`refund-notify-url`（支付回调地址，需公网可访问，本地通常使用内网穿透）
- `yshop.file-path`（文件与二维码保存目录；**`application-local.yaml` 中为 macOS 路径**，Windows 环境需修改）
- `yshop.tencent-lbs-key`（地图服务密钥，属于第三方密钥）

> ⚠ 以上配置文件中已存在数据库、微信、钉钉、企业微信、地图等密钥。**不要把真实密钥提交到版本库**，建议通过本地未跟踪的配置或环境变量覆盖。

### 2.3 编译（已确认）

在工程根目录 `yshop-crm/` 执行（README 给出的命令）：


### 2.5 Docker 部署（文件已确认，未实际执行）

`yshop-crm/script/docker/docker-compose.yml` 包含以下服务：

| 服务 | 镜像 / 构建 | 说明 |
|------|-------------|------|
| `mysql` | `mysql:8`，容器名 `yshop-mysql` | 数据库，数据卷 `mysql` |
| `redis` | `redis:6-alpine`，容器名 `yshop-redis` | 缓存，数据卷 `redis` |
| `server` | 由 `yshop-crm/yshop-server/Dockerfile` 构建，镜像 `yshop-server` | 后端，依赖 mysql、redis |
| `admin` | 前端构建（构建参数 `NODE_ENV`），镜像 `yshop-admin` | 前端，依赖 server |

> ⚠ Dockerfile 暴露端口为 `48080`，而 `application-local.yaml` 中端口为 `48081`，映射关系需确认（**待验证**）。
> ⚠ `docker.env` 包含环境变量与敏感信息，**不要外传**。

部署脚本 `yshop-crm/script/shell/deploy.sh` 使用 `nohup java -server ... -jar` 启动，日志输出到 `nohup.out`，并通过 `--spring.profiles.active=$PROFILES_ACTIVE` 指定环境（**本次未执行该脚本**）。

## 3. 前端启动

### 3.1 安装依赖（已确认）

在 `yshop-crm-vue/` 目录执行：

```bash
pnpm install
```

> `package.json` 中的 `i` 脚本即 `pnpm install`。

### 3.2 配置 API 地址（已确认）

环境变量文件：`yshop-crm-vue/.env`（全局）、`.env.local`、`.env.dev`、`.env.test`、`.env.stage`、`.env.prod`。

| 变量 | 说明 | 当前值 |
|------|------|--------|
| `VITE_BASE_URL` | 后端域名与端口 | `.env.local` 中为 `http://localhost:48081`；`.env.dev` 指向外部域名 |
| `VITE_API_URL` | API 前缀 | `/admin-api` |
| `VITE_PORT` | Vite 开发端口 | `.env` 中为 `80` |
| `VITE_APP_TENANT_ENABLE` | 是否启用多租户 | `.env` 中为 `false` |
| `VITE_APP_CAPTCHA_ENABLE` | 是否启用验证码 | `.env` 与 `.env.local` 中均为 `true` |

> `axios` 的 `baseURL` 由 `VITE_BASE_URL + VITE_API_URL` 拼接（`src/config/axios/config.ts`）。

## 4. 启动后验证

1. 后端：访问 `http://localhost:48081/swagger-ui`，能看到接口文档即说明后端启动成功（`application-local.yaml` 端口 `48081`）。
2. 前端：访问 `http://localhost:80`（`.env` 中 `VITE_PORT=80`），登录后能加载菜单即说明前后端连通。
3. 验证码：`application-local.yaml` 中 `yshop.captcha.enable: false`（本地关闭），而前端 `.env.local` 中 `VITE_APP_CAPTCHA_ENABLE=true`。两者是否影响登录流程，**待验证**（以实际登录页为准）。

## 5. 常见问题（基于配置推断，待验证）

| 现象 | 可能原因 | 排查方向 |
|------|----------|----------|
| 后端启动报数据源错误 | MySQL 地址、账号、库名与实际不一致 | 核对 `spring.datasource.dynamic.datasource.master` |
| Redis 连接失败 | `application-dev.yaml` 配置了 Redis `password`，本地若未设置会失败 | 核对 `spring.data.redis.password` |
| 前端登录后无菜单 | 菜单数据未导入，或 `system_menu.component` 与 `src/views/` 路径不一致 | 检查 `system_menu` 表与 `src/views/` |
| 支付二维码无法生成 | `yshop.file-path` 目录不存在或无写入权限 | 确认目录存在，Windows 下改为有效路径 |
| 定时任务未执行 | `spring.quartz.auto-startup` 在 dev 为 `false` | 本地开发按需开启 |
| 启动后数据库表缺失 | 仅导入了全量脚本，未执行增量索引脚本 | 参考第 2.1 节第 3 点 |

## 6. 注意事项

1. 配置文件中包含多处明文密钥与账号，启动前不要将修改后的本地配置提交到版本库。
2. 前端 `package.json` 中 `repository`、`bugs`、`homepage` 指向 `gitee.com/guchengwuyue/yshop-drink`（历史项目地址），与 `yshop-crm` 不一致，仅作参考。
3. `application-dev.yaml` 中 `spring.quartz.auto-startup: false`，`application-local.yaml` 中为 `true`。如果本地开启定时任务，`customerAutoJob` 会按配置自动放入公海，请注意测试数据影响。
4. 详细的测试执行方式见 [../testing/existing-tests.md](../testing/existing-tests.md)。CRM 模块当前没有单元测试。

> 本地开发应使用 `.env.local`，**不要直接修改 `.env.dev` 指向线上地址**。

### 3.3 启动（已确认）

```bash
npm run dev
```

等价于 `package.json` 中的 `"dev": "vite"`。`npm run dev-server` 对应 `vite --mode dev`。

### 3.4 构建与检查（已确认）

| 命令 | 说明 |
|------|------|
| `pnpm build:local` | 本地构建（`--max_old_space_size=8192`） |
| `pnpm build:dev` / `build:test` / `build:stage` / `build:prod` | 对应环境构建 |
| `pnpm preview` | 先执行 `build:local`，再 `vite preview` |
| `pnpm ts:check` | `vue-tsc --noEmit` 类型检查 |
| `pnpm lint:eslint` | `eslint --fix --ext .js,.ts,.vue ./src` |

```bash
mvn clean install package '-Dmaven.test.skip=true'
```

- 父 POM 使用 `flatten-maven-plugin` 统一 `revision`（当前 `2.1.3`），`install` 会将各子模块安装到本地仓库，启动 `yshop-server` 前需保证依赖模块已安装。
- 父 POM 配置了 `huaweicloud`、`aliyunmaven` 两个 Maven 镜像仓库（见 `yshop-crm/pom.xml`）。
- 注解处理器 `spring-boot-configuration-processor`、`lombok`、`mapstruct-processor` 已在父 POM 的 `annotationProcessorPaths` 中配置。

### 2.4 启动（已确认）

入口类：`co.yixiang.yshop.server.YshopServerApplication`（`yshop-crm/yshop-server/src/main/java/co/yixiang/yshop/server/`）。

- IDEA：直接运行 `YshopServerApplication`，并确认 Active profiles 为 `local` 或 `dev`。
- 命令行：在 `yshop-crm/yshop-server/target/` 下执行 `java -jar yshop-server.jar --spring.profiles.active=local`（jar 名称由 `<finalName>yshop-server</finalName>` 决定）。
- 启动后访问（路径均来自配置）：
  - 接口文档（Swagger / Knife4j）：`/swagger-ui`，OpenAPI JSON 为 `/v3/api-docs`。
  - Druid 监控：`/druid/*`（`stat-view-servlet` 已启用）。
  - Actuator：`/actuator`（`application-dev.yaml` 中开放了全部端点，**生产环境需收紧**）。
  - Spring Boot Admin：`/admin`（`spring.boot.admin.context-path`）。

> 注：管理端接口前缀为 `/admin-api`（前端 `VITE_API_URL` 与 CRM 接口清单一致）；`/app-api` 仅在回调放行规则中出现过，其它用户端接口是否统一使用该前缀，**待验证**。
