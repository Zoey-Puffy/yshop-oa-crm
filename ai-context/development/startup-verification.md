# 启动验证报告

> 验证日期：2026-11-15
> 验证范围：MySQL 数据库、Redis 缓存、配置一致性
> 验证方式：实际操作验证

---

## 1. MySQL 验证

### 1.1 版本与服务

| 项目 | 实际值 | 预期值 | 状态 |
|------|--------|--------|------|
| MySQL 版本 | 8.0.45 Community Server | 8.0.x | ✅ |
| MySQL 服务 | MySQL80 - Running | Running | ✅ |
| 连接地址 | 127.0.0.1:3306 | 127.0.0.1:3306 | ✅ |
| Root 密码 | mysql1234（与 application-local.yaml 一致） | 与实际配置一致 | ✅ 已修改 |

### 1.2 数据库配置

| 项目 | 实际值 | 预期值 | 状态 |
|------|--------|--------|------|
| 字符集（服务器） | utf8mb4 | utf8mb4 | ✅ |
| 排序规则（服务器） | utf8mb4_0900_ai_ci | utf8mb4_unicode_ci | ⚠️ 差异可接受 |
| 排序规则（表级） | utf8mb4_general_ci | utf8mb4_unicode_ci | ⚠️ 差异可接受 |
| 时区 | SYSTEM（中国标准时间） | Asia/Shanghai | ✅ |
| SQL mode | ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,... | 默认 | ✅ |

### 1.3 数据库 `yshop-crm-open-source`

| 项目 | 实际值 | 预期值 | 状态 |
|------|--------|--------|------|
| 数据库名称 | yshop-crm-open-source | yshop-crm-open-source | ✅ |
| 数据库存在 | 是 | 是 | ✅ |
| 总表数量 | 90 | — | ✅ |
| CRM 表数量 | 16（yshop_crm_*） | 16 | ✅ |
| 引擎 | InnoDB + Dynamic 行格式 | InnoDB | ✅ |
| 增量索引 | 已全部执行（7 张表共 13 个索引） | 已执行 | ✅ |

### 1.4 核心表与数据

| 表名 | 数据行数 | 状态 |
|------|----------|------|
| system_users | 包含示例数据（admin 等） | ✅ |
| system_menu | 包含菜单数据 | ✅ |
| yshop_crm_customer | 已创建 | ✅ |
| 其余 yshop_crm_* 表 | 14 张已创建 | ✅ |

### 1.5 配置一致性

| 配置项 | application-local.yaml | 实际环境 | 状态 |
|--------|----------------------|----------|------|
| URL | jdbc:mysql://127.0.0.1:3306/yshop-crm-open-source | 127.0.0.1:3306 | ✅ |
| Username | root | root | ✅ |
| Password | mysql1234 | mysql1234 | ✅ 已修改 |
| serverTimezone | Asia/Shanghai | SYSTEM（中国标准时间） | ✅ |

## 2. Redis 验证

### 2.1 运行状态

| 项目 | 实际值 | 预期值 | 状态 |
|------|--------|--------|------|
| 运行方式 | Docker 容器 `yshop-redis` | Docker | ✅ |
| 镜像版本 | redis:6-alpine → 6.2.24 | redis:6-alpine | ✅ |
| 端口映射 | 0.0.0.0:6379->6379/tcp | 6379 | ✅ |
| PING | PONG | PONG | ✅ |
| requirepass | 空（无密码） | 无密码 | ✅ |
| SELECT 0 | OK | OK | ✅ |
| uptime | 正常运行中 | — | ✅ |

### 2.2 配置一致性

| 配置项 | application-local.yaml | 实际环境 | 状态 |
|--------|----------------------|----------|------|
| host | 127.0.0.1 | 127.0.0.1 | ✅ |
| port | 6379 | 6379 | ✅ |
| database | 0 | 0 | ✅ |
| password | 未配置（注释掉） | 无密码 | ✅ |

## 3. 已验证的数据库连接参数汇总

| 参数 | 值 | 来源 |
|------|-----|------|
| MySQL 主机 | 127.0.0.1 | `mysql -h` 验证 |
| MySQL 端口 | 3306 | 默认，`netstat` 确认 |
| MySQL 用户 | root | `SELECT USER()` 验证 |
| MySQL 数据库 | yshop-crm-open-source | `SHOW DATABASES` 验证 |
| MySQL charset | utf8mb4 | `SHOW VARIABLES` 验证 |
| Redis 主机 | 127.0.0.1 | `docker inspect` 验证 |
| Redis 端口 | 6379 | `docker ps` 验证 |
| Redis 无密码 | 确认 | `CONFIG GET requirepass` 验证 |

> 所有连接参数均通过实际操作验证，非仅基于配置推断。

## 4. 待验证（运行时）

以下项需要在后端启动后进一步验证：
1. 数据库连接字符串中 `allowPublicKeyRetrieval=true&nullCatalogMeansCurrent=true` 参数在 MySQL 8.0.45 上的实际表现
2. Druid 连接池初始化是否成功
3. Redisson 客户端连接 Redis 是否正常
4. Quartz JDBC 表（QRTZ_*）是否能正常使用
5. 多数据源主从切换是否正常