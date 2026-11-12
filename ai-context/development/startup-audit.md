# YShop CRM 启动环境全面审计报告

> 审计日期：2026-11-12
> 审计范围：后端 Maven 模块、前端 Vue 工程、数据库脚本、Docker 部署配置、部署脚本
> 审计方式：仅基于源码与配置文件的静态分析，未执行任何运行时验证

---

## 1. 项目实际启动架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        本地开发环境                              │
│                                                                 │
│  ┌──────────────┐     HTTP:48081/admin-api/...   ┌───────────┐ │
│  │ Vue 3 + Vite │  ◀──────────────────────────▶  │ Spring    │ │
│  │ :80          │     axios Bearer Token          │ Boot:48081│ │
│  └──────────────┘                                 └─────┬─────┘ │
│                                                          │       │
│               ┌──────────────────────────────────────────┼───┐   │
│               │ MySQL:3306/yshop-crm-open-source         │   │   │
│               │ Redis:6379/db=0                          │   │   │
│               └──────────────────────────────────────────┘   │   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        Docker 部署环境                          │
│                                                                 │
│  ┌──────────────┐     HTTP:48080/admin-api/...   ┌───────────┐ │
│  │ Nginx        │  ◀──────────────────────────▶  │ Spring    │ │
│  │ :8080        │                                │ Boot:48080│ │
│  └──────────────┘                                └─────┬─────┘ │
│                                                          │       │
│               ┌──────────────────────────────────────────┼───┐   │
│               │ yshop-mysql:3306/yixiang-drink           │   │   │
│               │ yshop-redis:6379                         │   │   │
│               └──────────────────────────────────────────┘   │   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. 各组件软件版本

### 2.1 后端（Java）

| 组件 | 版本 | 源码依据 | 状态 |
|------|------|----------|------|
| JDK 编译版本 | **17** | `yshop-crm/pom.xml` L32: `<java.version>17</java.version>` | [已确认] |
| Maven 编译器 source/target | 17 | `pom.xml` L33-34: `${java.version}` | [已确认] |
| Maven Wrapper | **不存在** | `yshop-crm/` 下无 `.mvn/` 目录 | [已确认] |
| Spring Boot | **3.2.2** | `pom.xml` L40: `<spring.boot.version>3.2.2</spring.boot.version>` | [已确认] |
| 项目版本 | **2.1.3** | `pom.xml` L30: `<revision>2.1.3</revision>` | [已确认] |
| MyBatis-Plus | 3.5.5 | `yshop-dependencies/pom.xml` L26 | [已确认] |
| Lombok | 1.18.30 | `pom.xml` L39 | [已确认] |
| MapStruct | 1.5.5.Final | `pom.xml` L41 | [已确认] |
| Maven 打包 finalName | `yshop-server.jar` | `yshop-server/pom.xml` L95: `<finalName>${project.artifactId}</finalName>` | [已确认] |

### 2.2 数据库

| 项 | 值 | 源码依据 | 状态 |
|----|-----|----------|------|
| SQL 导出源 MySQL 版本 | **9.6.0** (90600) | `sql/yshop-crm-open-source.sql` L6 | [已确认] |
| Docker Compose MySQL 镜像 | **mysql:8** | `script/docker/docker-compose.yml` L8 | [已确认] |
| 字符集 | utf8mb4 | SQL 文件 `SET NAMES utf8mb4` | [已确认] |
| 排序规则 | utf8mb4_unicode_ci | 各表 DDL | [已确认] |
| 时区 | Asia/Shanghai | JDBC URL | [已确认] |
| SQL 文件 Schema | **yshop-crm-open-source** | SQL 文件 L8 | [已确认] |
| 全量 SQL 文件 | 含表结构、菜单、字典、示例数据 | `sql/yshop-crm-open-source.sql` | [已确认] |
| 增量脚本 | CRM 表组合索引 | `sql/migration/20261008_p0_crm_business_indexes.sql` | [已确认] |

### 2.3 Redis

| 项 | 值 | 源码依据 | 状态 |
|----|-----|----------|------|
| Docker 镜像 | **redis:6-alpine** | `docker-compose.yml` L21 | [已确认] |
| 本地配置 | host=127.0.0.1, port=6379, database=0, 无密码 | `application-local.yaml` L61-63 | [已确认] |
| 开发配置 | host=127.0.0.1, port=6379, database=2, 有密码 | `application-dev.yaml` L55-58 | [已确认] |
| Docker 环境地址 | yshop-redis | `docker-compose.yml` L53 | [已确认] |

### 2.4 Docker

| 服务 | 镜像/构建 | 端口映射 | 依赖 |
|------|-----------|----------|------|
| mysql | `mysql:8` | 3306:3306 | — |
| redis | `redis:6-alpine` | 6379:6379 | — |
| server | 由 `Dockerfile` 构建 | **48080:48080** | mysql, redis |
| admin | 由 `./yshop-ui-admin/` 构建 | 8080:80 | server |

### 2.5 前端

| 组件 | 版本/要求 | 源码依据 | 状态 |
|------|-----------|----------|------|
| Node.js | >= 16.0.0 | `package.json` | [已确认] |
| pnpm | >= 8.6.0 | `package.json` | [已确认] |
| pnpm lock | 存在 `pnpm-lock.yaml` | 根目录 | [已确认] |
| Vue | 3.4.21 | `package.json` | [已确认] |
| Vite | 5.1.4 | `package.json` | [已确认] |
| Element Plus | 2.6.1 | `package.json` | [已确认] |
| 本地端口 | **80** | `.env` VITE_PORT=80 | [已确认] |
| 本地 API | `http://localhost:48081` + `/admin-api` | `.env.local` | [已确认] |
| axios baseURL | `VITE_BASE_URL + VITE_API_URL` | `src/config/axios/config.ts` | [已确认] |

---

## 3. 配置文件的作用与关系

### 3.1 后端配置文件优先级

```
application.yaml（基础）
    ↓ 被覆盖
application-local.yaml（默认激活）
    ↓ 或
application-dev.yaml（--spring.profiles.active=dev）
    ↓ 或
Docker 环境变量（docker-compose.yml ARGS 传入）
```

### 3.2 前端环境变量优先级（Vite）

```
.env（基础）
    ↓ 被覆盖
.env.local（本地覆盖）
    ↓ 被覆盖
.env.[mode]（--mode 参数指定）
```

---

## 4. 已发现的配置冲突

### [存在冲突] 冲突 1：Docker Compose 中 server 构建上下文路径错误

- **文件**：`yshop-crm/script/docker/docker-compose.yml` L32-33
- **问题**：`context: ./yshop-server/` 相对于 `docker-compose.yml` 所在目录（`script/docker/`）解析，实际路径为 `script/docker/yshop-server/`，但 Dockerfile 实际在 `yshop-crm/yshop-server/Dockerfile`。
- **影响**：Docker Compose 无法找到 Dockerfile，构建失败。
- **修复方向**：context 应改为 `../../yshop-server/`。

### [存在冲突] 冲突 2：Docker Compose 中 admin（前端）构建上下文路径不存在

- **文件**：`yshop-crm/script/docker/docker-compose.yml` L61
- **问题**：`context: ./yshop-ui-admin/` 指向 `script/docker/yshop-ui-admin/`，但实际前端工程在 `yshop-crm-vue/`。环境变量使用 `VUE_APP_*`（Vue 2），项目实际使用 `VITE_*`（Vue 3）。
- **影响**：admin 服务无法构建，参数也不匹配。
- **修复方向**：context 改为 `../../yshop-crm-vue/`，环境变量改为 `VITE_*`。

### [存在冲突] 冲突 3：Dockerfile 暴露端口与 Spring 配置端口不一致

| 来源 | 端口 |
|------|------|
| `Dockerfile` L19: `EXPOSE 48080` | 48080 |
| `docker-compose.yml` L36: `"48080:48080"` | 48080 |
| `application-local.yaml` L2: `server.port: 48081` | **48081** |
| `application-dev.yaml` L2: `server.port: 48081` | **48081** |
| `deploy.sh` L14: health check URL | **48080** |

- **影响**：Docker 环境中 Spring 使用 48081 端口，但 Docker 映射 48080。Docker Compose 未覆盖 `server.port`。
- **修复方向**：统一端口，或在 Docker 环境变量中添加 `--server.port=48080`。

### [存在冲突] 冲突 4：JDK 版本不一致

| 来源 | JDK |
|------|-----|
| `pom.xml` `<java.version>17</java.version>` | **JDK 17** |
| `Dockerfile` `eclipse-temurin:21-jre` | **JRE 21** |

- **影响**：Java 21 理论上兼容 JDK 17 编译的字节码，但未经实际测试。
- **状态**：[待验证]

### [存在冲突] 冲突 5：数据库名称不统一

| 来源 | 数据库名 |
|------|----------|
| `application-local.yaml` | **yshop-crm-open-source** |
| `application-dev.yaml` | **yshop_crm** |
| `docker-compose.yml` 默认值 | **yixiang-drink** |
| `docker.env` | **yixiang-drink** |
| SQL 文件头 Source Schema | **yshop-crm-open-source** |

- **影响**：不同环境使用不同的数据库名，Docker 默认 `yixiang-drink` 与 SQL 文件不匹配。
- **修复方向**：Docker 环境应明确指定 `MYSQL_DATABASE=yshop-crm-open-source`。

### [存在冲突] 冲突 6：Docker 环境 MySQL URL 中数据库名不匹配

- **文件**：`docker-compose.yml` L47-48
- **问题**：`jdbc:mysql://yshop-mysql:3306/yixiang-drink?...`，与 SQL 文件目标库 `yshop-crm-open-source` 不一致。
- **修复方向**：统一数据库名。

### [存在冲突] 冲突 7：Docker Compose SQL 初始化脚本路径不存在

- **文件**：`docker-compose.yml` L18
- **问题**：`./sql/mysql/yshop-crm.sql` 相对于 `script/docker/` 解析，实际 SQL 文件在 `sql/yshop-crm-open-source.sql`。
- **影响**：MySQL 容器无法自动初始化数据。
- **修复方向**：挂载 `../../sql/yshop-crm-open-source.sql`。

### [存在冲突] 冲突 8：Deploy 脚本健康检查端口与实际端口不一致

- **文件**：`deploy.sh` L14
- **问题**：健康检查 URL 使用 48080，但 Spring 配置使用 48081。
- **影响**：健康检查失败。
- **修复方向**：修改端口为 48081 或确保部署 profile 使用 48080。

### [存在冲突] 冲突 9：本地验证码配置前后端不一致

| 来源 | 验证码 |
|------|--------|
| `application-local.yaml` | **关闭** (false) |
| `.env.local` | **开启** (true) |

- **影响**：前端显示验证码输入框，后端不验证。登录可能正常，但体验不一致。
- **修复方向**：统一设置为 false（本地开发）。

### [存在冲突] 冲突 10：文件路径为 macOS 格式，不适用于 Windows

- **文件**：`application-local.yaml` L212: `yshop.file-path: /Users/hupeng/drink/file/`
- **影响**：Windows 环境无法使用，文件上传和二维码生成会失败。
- **修复方向**：修改为 Windows 有效路径，如 `D:/yshop/file/`。

### [需要决策] 冲突 11：MySQL 版本差异

- SQL 文件导出自 MySQL 9.6.0，Docker Compose 使用 mysql:8 镜像。
- **影响**：当前 SQL 语法在 MySQL 8 上通常兼容。
- **状态**：[待验证] 需实际导入验证。

---

## 5. 缺失的信息及无法确认的事项

### 5.1 无法确认的配置项

| 事项 | 说明 |
|------|------|
| `yshop.info.base-package` | 启动类注解使用该占位符，需在配置中定义 |
| `application-prod.yaml` | 生产环境 profile 文件是否存在 |
| Maven 本地仓库状态 | 是否已缓存所有依赖 |
| 微信/钉钉/企业微信密钥有效性 | 配置中的密钥可能已过期 |
| `application-my.yaml` | `.gitignore` 中排除，用户可能已创建但未跟踪 |
| 前端 Dockerfile | 项目中未找到，Docker Compose admin 服务需要 |

### 5.2 项目中不存在的组件

| 组件 | 结论 | 依据 |
|------|------|------|
| Nacos | **不存在** | 无相关配置或依赖 |
| RabbitMQ | **不存在** | 消息使用 Redis Stream |
| Elasticsearch | **不存在** | 无相关配置或依赖 |
| MinIO | **依赖存在但非必需** | 文件上传可通过 server 模式 |
| SkyWalking | **可选** | deploy.sh 中已注释，本地可忽略 |


---

## 6. startup-guide.md 需要修正的内容

| 编号 | 当前内容 | 问题 | 建议修正 |
|------|----------|------|----------|
| 1 | 第 2.3 节 Maven 编译命令后直接跳转到第 2.5 节 Docker 部署 | 缺少第 2.4 节「启动」的锚点（实际内容在文档末尾第 152 行） | 将第 2.4 节内容移到第 2.3 节之后 |
| 2 | 第 2.4 节启动说明中 jar 名称描述不完整 | 只说了 `yshop-server.jar` 但不清楚路径 | 明确路径 `yshop-server/target/yshop-server.jar` |
| 3 | 第 2.5 节 Docker 部署未指出构建上下文路径问题 | `./yshop-server/` 和 `./yshop-ui-admin/` 路径错误 | 增加警告说明 Docker Compose 存在路径问题 |
| 4 | 未说明 Maven Wrapper 不存在 | 用户可能需要手动安装 Maven | 增加提示：需在系统安装 Maven 3.8+ |
| 5 | 未说明 MySQL 版本差异 | SQL 导出自 MySQL 9.6.0，Docker 使用 mysql:8 | 增加说明 |
| 6 | 前端启动建议使用 `npm run dev` | 项目要求 pnpm | 统一使用 `pnpm dev` |
| 7 | Windows 环境注意事项 | 未提及文件路径和端口 80 可能被占用 | 增加 Windows 专属说明 |

---

## 7. 建议的启动方式及其理由

### 7.1 推荐方案：本地直接启动（非 Docker）

鉴于 Docker Compose 配置文件存在多处路径错误和环境变量不匹配，**首次启动强烈建议使用本地直接启动方式**。

#### 步骤概览

```
1. 准备环境（JDK 17 + Maven 3.8+ + MySQL 8 + Redis 6+ + Node 16+ + pnpm 8.6+）
2. 创建数据库 yshop-crm-open-source
3. 导入全量 SQL → 执行增量索引脚本
4. 修改 application-local.yaml（数据库密码、Redis、文件路径）
5. Maven 编译：mvn clean install package -Dmaven.test.skip=true
6. 启动后端：java -jar yshop-server/target/yshop-server.jar --spring.profiles.active=local
7. 前端安装依赖：pnpm install
8. 前端启动：pnpm dev
9. 验证：后端 http://localhost:48081/swagger-ui，前端 http://localhost:80
```

#### 理由
- Docker Compose 存在多处路径错误，需要修复后才能使用
- 本地方式便于调试和观察日志
- 无需处理 Docker 网络和端口映射问题

### 7.2 Docker 部署（待修复后使用）

Docker 部署需要先修复以下问题：
1. docker-compose.yml 中 server 和 admin 的 build context 路径
2. docker-compose.yml 中 SQL 初始化脚本挂载路径
3. 统一数据库名为 `yshop-crm-open-source`
4. 统一端口为 48080
5. 解决 admin 服务环境变量前缀问题（VUE_APP_* → VITE_*）
6. 创建前端 Dockerfile（当前不存在）

---

## 8. 后续任务执行顺序建议

| 优先级 | 任务 | 说明 |
|--------|------|------|
| P0 | **安装/确认本地依赖** | JDK 17、Maven 3.8+、MySQL 8、Redis 6+、Node 16+、pnpm 8.6+ |
| P0 | **创建数据库并导入 SQL** | 创建 `yshop-crm-open-source` 库，导入全量 SQL，执行增量索引 |
| P0 | **修改本地配置** | 数据库密码、Redis、文件路径（Windows）、微信等密钥 |
| P1 | **Maven 编译** | `mvn clean install package -Dmaven.test.skip=true` |
| P1 | **启动后端** | 验证 Swagger 文档可访问 |
| P1 | **前端安装依赖并启动** | `pnpm install && pnpm dev` |
| P2 | **登录验证** | 验证前后端连通性 |
| P2 | **Docker Compose 修复** | 修复路径和配置冲突（如需 Docker 部署） |
| P3 | **补充缺失的配置文件** | 如 `application-my.yaml` 本地覆盖配置 |
| P3 | **更新 startup-guide.md** | 基于审计结果修正 |

---

## 9. 综合状态统计

| 状态 | 数量 |
|------|------|
| [已确认] | 30 项 |
| [存在冲突] | 11 项 |
| [待验证] | 4 项 |
| [需要决策] | 1 项（MySQL 版本选择） |

---

## 附录 A：关键文件路径速查

| 文件 | 路径 |
|------|------|
| 父 POM | `yshop-crm/pom.xml` |
| 依赖 BOM | `yshop-crm/yshop-dependencies/pom.xml` |
| 启动类 | `yshop-crm/yshop-server/src/main/java/.../YshopServerApplication.java` |
| 公共配置 | `yshop-crm/yshop-server/src/main/resources/application.yaml` |
| 本地配置 | `yshop-crm/yshop-server/src/main/resources/application-local.yaml` |
| 开发配置 | `yshop-crm/yshop-server/src/main/resources/application-dev.yaml` |
| 日志配置 | `yshop-crm/yshop-server/src/main/resources/logback-spring.xml` |
| 后端 Dockerfile | `yshop-crm/yshop-server/Dockerfile` |
| Docker Compose | `yshop-crm/script/docker/docker-compose.yml` |
| Docker 环境变量 | `yshop-crm/script/docker/docker.env` |
| 部署脚本 | `yshop-crm/script/shell/deploy.sh` |
| 全量 SQL | `yshop-crm/sql/yshop-crm-open-source.sql` |
| 增量 SQL | `yshop-crm/sql/migration/20261008_p0_crm_business_indexes.sql` |
| 前端 package.json | `yshop-crm-vue/package.json` |
| 前端 Vite 配置 | `yshop-crm-vue/vite.config.ts` |
| 前端全局环境 | `yshop-crm-vue/.env` |
| 前端本地环境 | `yshop-crm-vue/.env.local` |
| 前端 axios 配置 | `yshop-crm-vue/src/config/axios/config.ts` |
| 根 .gitignore | `.gitignore` |
| 后端 .gitignore | `yshop-crm/.gitignore` |

---

## 附录 B：配置文件中已发现的明文敏感信息位置

> 以下配置文件包含密钥、密码、Token 等敏感信息，**切勿提交到版本库**。

| 文件 | 敏感信息类型 |
|------|-------------|
| `application-local.yaml` | MySQL 密码、微信 MP/小程序密钥、钉钉/企微 client-secret、腾讯地图密钥 |
| `application-dev.yaml` | MySQL 密码、Redis 密码、微信 MP/小程序密钥、钉钉/企微 client-secret、腾讯地图密钥 |
| `application.yaml` | MyBatis-Plus 加密密钥、快递鸟/快递 100 API Key |
| `docker.env` | MySQL 密码 |

