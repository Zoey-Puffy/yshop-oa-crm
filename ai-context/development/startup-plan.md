# YShop CRM 本地启动环境配置方案

> 版本：1.0
> 创建日期：2026-11-12
> 状态：方案制定（未执行）

---

## 目录

1. [环境现状总览](#1-环境现状总览)
2. [软件版本推荐与选择依据](#2-软件版本推荐与选择依据)
3. [架构决策说明](#3-架构决策说明)
4. [配置修改方案](#4-配置修改方案)
5. [数据库创建与初始化](#5-数据库创建与初始化)
6. [后端启动步骤](#6-后端启动步骤)
7. [前端启动步骤](#7-前端启动步骤)
8. [本地环境与 Docker 环境的切换](#8-本地环境与-docker-环境的切换)
9. [执行顺序总表](#9-执行顺序总表)
10. [需要你确认的决策](#10-需要你确认的决策)
11. [后续任务规划（Task 2～6）](#11-后续任务规划task-26)

---

## 1. 环境现状总览

### 1.1 已安装软件（Task 2 确认）

| 软件 | 版本 | 状态 |
|------|------|------|
| Java (JDK) | 17.0.12 | ✅ 已安装，符合源码要求 |
| Maven | 3.9.11 | ✅ IntelliJ IDEA 2026.1 内置 Maven，已添加到 PATH |
| MySQL | 8.0.45 (Community Server) | ✅ 已安装，符合源码要求 |
| Redis | — | ⏳ 待通过 Docker Desktop 启动容器 |
| Node.js | 24.16.0 | ✅ 已安装，远高于最低要求（>=16） |
| npm | 11.13.0 | ✅ 已安装（随 Node.js 自带） |
| pnpm | 12.18.0 | ✅ 已安装，远高于最低要求（>=8.6） |
| Docker Desktop | 29.9.3 | ✅ 已安装 |
| IntelliJ IDEA | 2026.1 | ✅ 已安装 |
| Git | — | ✅ 已安装 |

### 1.2 安装优先级（已全部满足）

```
所有软件已就绪，无需进一步安装。
```

---

## 2. 软件版本推荐与选择依据

### 2.1 Java (JDK)

| 项目 | 推荐值 |
|------|--------|
| **版本** | **17.0.x**（已安装 17.0.12） |
| **选择依据** | 源码强制要求：`pom.xml` `<java.version>17</java.version>` |
| **类型** | **源码明确要求** |
| **当前状态** | ✅ 已满足，无需变更 |

> **如未来需要升级**：源码 `pom.xml` L32 同时控制 `<maven.compiler.source>` 和 `<maven.compiler.target>`，升级 JDK 只需修改此处一处。但当前 JDK 17 完全可用，不建议升级。

### 2.2 Maven

| 项目 | 推荐值 |
|------|--------|
| **版本** | **3.9+**（兼容 3.8+） |
| **选择依据** | `README.md` 要求 3.8+；`pom.xml` 使用了 `maven-compiler-plugin 3.11.0` 和 `flatten-maven-plugin 1.5.0`，这些插件的最新特性需要 Maven 3.8+ 支持。推荐 3.9.x 以获得更好的稳定性和 Spring Boot 3.x 兼容性 |
| **类型** | **源码要求 + 依赖兼容性推导** |
| **当前状态** | ❌ 需要安装 |

#### 安装方式

**方案 A：手动安装（推荐）**

1. 下载地址：https://maven.apache.org/download.cgi
2. 选择：`Binary zip` (Windows `.zip`)
3. 具体版本：**3.9.9**（2024 年最新稳定版，已验证与 Spring Boot 3.2.x 兼容）
4. 安装步骤：
   - 解压到 `C:\tools\maven`（或其他无空格路径）
   - 配置环境变量：
     - `MAVEN_HOME = C:\tools\maven`
     - `PATH` 追加 `;%MAVEN_HOME%\bin`
5. 验证：打开新 CMD/PowerShell，执行 `mvn -v`，应显示 Maven 3.9.x

**方案 B：通过包管理器（Scoop/Chocolatey）**

```powershell
# Scoop
scoop install maven
# Chocolatey
choco install maven
```

**方案 A（手动安装）的优点**：路径可控，不依赖第三方包管理器，便于多版本切换。

> **本项目不使用 Maven Wrapper**（`yshop-crm/` 下无 `.mvn/` 目录）。如果你希望后续统一构建版本，可以在未来通过 `mvn wrapper` 命令生成，本次暂不引入。

---

## 3. 架构决策说明

### 3.1 架构总览

```
                               ┌──────────────────────────────────────┐
                               │           本地开发环境               │
                               │                                      │
                               │  ┌──────────────┐   ┌─────────────┐ │
                               │  │  Vue 3 + Vite│   │ Spring Boot │ │
                               │  │  :80          │   │ :48081      │ │
                               │  └──────┬───────┘   └──────┬──────┘ │
                               │         │                    │       │
                               │         └── HTTP ───────────┘       │
                               │         axios /admin-api             │
                               │                                      │
                               │  ┌──────────────────────────────┐    │
                               │  │  MySQL 8.0.45 (本地安装)     │    │
                               │  │  127.0.0.1:3306              │    │
                               │  │  database: yshop-crm-open-source│  │
                               │  └──────────────────────────────┘    │
                               │                                      │
                               │  ┌──────────────────────────────┐    │
                               │  │  Redis 6 (Docker Desktop)    │    │
                               │  │  127.0.0.1:6379              │    │
                               │  │  db=0                        │    │
                               │  └──────────────────────────────┘    │
                               └──────────────────────────────────────┘
```

### 3.2 Redis Docker 启动方案

由于你在 Windows 上无法直接安装 Redis，使用 Docker Desktop 运行 Redis 容器是最简洁的方案。

#### 3.2.1 方案 A：docker run 命令（推荐，简单直接）

```powershell
# 拉取并启动 Redis 6 容器
docker run -d ^
  --name yshop-redis ^
  -p 6379:6379 ^
  --restart unless-stopped ^
  redis:6-alpine
```

**优点**：无需编写 YAML 文件，执行一条命令即可。

**管理命令**：

```powershell
# 查看 Redis 运行状态
docker ps -a --filter name=yshop-redis

# 停止 Redis
docker stop yshop-redis

# 重新启动
docker start yshop-redis

# 查看 Redis 日志
docker logs yshop-redis

# 进入 Redis CLI
docker exec -it yshop-redis redis-cli
```

#### 3.2.2 方案 B：独立 docker-compose-redis.yml

在 `yshop-crm/script/docker/compose-redis.yml` 中创建：

```yaml
version: "3.4"
name: yshop-redis
services:
  redis:
    container_name: yshop-redis
    image: redis:6-alpine
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
volumes:
  redis-data:
    driver: local
```

启动命令：
```powershell
docker compose -f compose-redis.yml up -d
```

**推荐**：初期使用方案 A，需要更复杂配置时再切换方案 B。

#### 3.2.3 数据持久化

Redis 容器默认数据存储在容器内 `/data`。方案 A 未挂载卷，容器停止后数据丢失（本地开发通常没问题）。

如需保留数据，加 `-v redis-data:/data` 参数。

### 3.3 不使用全量 Docker Compose 的原因

现有 `docker-compose.yml` 存在 6 个问题（详见 startup-audit.md）：

| 问题 | 影响 |
|------|------|
| server build.context 路径错误 | 无法构建 backend |
| admin build.context 路径错误 | 无法构建 frontend |
| SQL 初始化挂载路径错误 | MySQL 不会自动初始化 |
| 数据库名 `yixiang-drink` 不统一 | 混淆 |
| 环境变量前缀 `VUE_APP_*` 与 `VITE_*` 不一致 | env 无效 |
| 前端无 Dockerfile | 无法构建 |

**结论**：修复后才能全量 Docker Compose 部署。个人学习开发推荐「MySQL 本地 + Redis Docker」方案。

| 项目 | 推荐值 |
|------|--------|
| **Node.js 版本** | **>= 16.0.0**（当前已安装 **24.16.0**） |
| **选择依据** | `yshop-crm-vue/package.json` `engines.node >= 16.0.0` |
| **类型** | **源码明确要求** |
| **当前状态** | ✅ Node.js 已满足 |

| 项目 | 推荐值 |
---

## 4. 配置修改方案

### 4.1 修改清单总览

| # | 文件 | 修改内容 | 是否必须 | 依赖 |
|---|------|----------|---------|------|
| 1 | `application-local.yaml` | MySQL 密码 → 你的本地 MySQL root 密码 | ✅ 必须 | MySQL 已安装 |
| 2 | `application-local.yaml` | `yshop.file-path` → Windows 有效路径 | ✅ 必须 | 无 |
| 3 | `application-local.yaml` | `yshop.captcha.enable` → `true`（可选） | ⚠️ 建议 | 无 |
| 4 | 无需修改 | `application.yaml` 公共配置 | — | — |
| 5 | 无需修改 | `.env.local` 前端配置 | — | — |

### 4.2 修改 `application-local.yaml`

**文件**：`d:\workspace\yshop-oa-crm\yshop-crm\yshop-server\src\main\resources\application-local.yaml`

#### 修改 1：MySQL 密码

**位置**：第 49～56 行

**修改前（默认值）**：

```yaml
        master:
          url: jdbc:mysql://127.0.0.1:3306/yshop-crm-open-source?...  # L49
          username: root                                                # L50
          password: root                                                # L51 ← 默认密码 "root"
        slave:
          url: jdbc:mysql://127.0.0.1:3306/yshop-crm-open-source?...   # L53
          username: root                                                # L54
          password: root                                                # L55 ← 默认密码 "root"
```

**修改后示例**（将 `root` 替换为你的实际 MySQL 密码）：

```yaml
        master:
          url: jdbc:mysql://127.0.0.1:3306/yshop-crm-open-source?...
          username: root
          password: 你的MySQL密码       # ← 修改此值
        slave:
          url: jdbc:mysql://127.0.0.1:3306/yshop-crm-open-source?...
          username: root
          password: 你的MySQL密码       # ← 修改此值
```

**影响**：不修改则无法连接 MySQL，启动时报 `Access denied for user 'root'@'localhost'`。

**如何确认你的密码**：PowerShell 中执行 `mysql -u root -p`，输入的就是你的 MySQL root 密码。

#### 修改 2：文件存储路径（Windows 路径适配）

**位置**：第 212 行 `yshop.file-path`

**修改前（macOS 路径，不可用于 Windows）**：

```yaml
yshop:
  file-path: /Users/hupeng/drink/file/
```

**修改后示例**（Windows 路径，用正斜杠 `/`）：

```yaml
yshop:
  file-path: D:/workspace/yshop-oa-crm/yshop-data/file/
```

**影响**：
- 用于支付二维码图片、文件上传的保存目录
- 目录不存在会导致二维码生成失败、文件上传报错
- **需手动创建目录并加入 `.gitignore`**

创建目录命令：

```powershell
mkdir "d:\workspace\yshop-oa-crm\yshop-data\file"
```

#### 修改 3（可选）：验证码开关
---

## 5. 数据库创建与初始化

### 5.1 创建数据库

```powershell
# 连接 MySQL
mysql -u root -p

# 在 MySQL 命令行中执行：
mysql> CREATE DATABASE IF NOT EXISTS `yshop-crm-open-source`
         CHARACTER SET utf8mb4
         COLLATE utf8mb4_unicode_ci;
mysql> SHOW DATABASES;
mysql> EXIT;
```

### 5.2 导入全量 SQL

```powershell
mysql -u root -p yshop-crm-open-source ^
  < "d:\workspace\yshop-oa-crm\yshop-crm\sql\yshop-crm-open-source.sql"
```

> 约 11.8 MB，耗时几十秒到几分钟。

### 5.3 执行增量索引脚本

```powershell
mysql -u root -p yshop-crm-open-source ^
  < "d:\workspace\yshop-oa-crm\yshop-crm\sql\migration\20261008_p0_crm_business_indexes.sql"
```

> ⚠️ 增量脚本不在全量 SQL 中，必须单独执行，否则 CRM 查询性能受影响。

### 5.4 验证初始化

```sql
USE `yshop-crm-open-source`;
SHOW TABLES;                          -- 应显示 130+ 张表
SELECT COUNT(*) FROM `system_users`;  -- 应 > 0（有 admin 等示例用户）
```

---

## 6. 后端启动步骤

### 6.1 编译

```powershell
cd d:\workspace\yshop-oa-crm\yshop-crm
mvn clean install package "-Dmaven.test.skip=true"
```

**说明**：
- `clean` → 清理上次构建
- `install` → 安装各模块到本地 Maven 仓库（yshop-server 依赖其它模块）
- `package` → 打包 `yshop-server/target/yshop-server.jar`
- `-Dmaven.test.skip=true` → 跳过测试（CRM 无单元测试）

**首次编译耗时**：约 5～15 分钟（视网络速度，首次下载依赖）。

### 6.2 启动后端

```powershell
cd d:\workspace\yshop-oa-crm\yshop-crm\yshop-server\target
java -jar yshop-server.jar --spring.profiles.active=local
```

> `--spring.profiles.active=local` 可以省略（`application.yaml` 默认即为 `local`），但显式指定更清晰。

### 6.3 启动前 Checklist

- [ ] Docker Redis 已启动（`docker ps` 查看）
- [ ] MySQL 服务正在运行
- [ ] 数据库 `yshop-crm-open-source` 已创建且已导入
- [ ] `application-local.yaml` 中 MySQL 密码已修改
- [ ] `yshop.file-path` 目录已创建
- [ ] Maven 编译已完成
---

## 8. 本地环境与 Docker 环境的切换

### 8.1 当前方案

```
本地 → MySQL 8.0.45（Windows 进程）
Docker → Redis 6（yshop-redis 容器）
```

切换回纯本地（非 Windows）只需停止 Redis 容器，在本地启动 Redis 进程。

### 8.2 若未来将 MySQL 也移入 Docker

| 变更 | 说明 |
|------|------|
| `master.url` | `127.0.0.1:3306` → `yshop-mysql:3306` |
| Docker Compose | 需加入 MySQL 服务 |
| 数据持久化 | 需挂载数据卷，防止数据丢失 |
| SQL 初始化 | 需挂载到 `/docker-entrypoint-initdb.d/` |

### 8.3 若未来需要全量 Docker Compose

参考 [3.3 节](#33-不使用全量-docker-compose-的原因)，先修复现有 `docker-compose.yml` 的 6 个问题。**建议个人学习阶段保持当前方案**。

---

## 9. 执行顺序总表

| 步 | 任务 | 预计时间 | 备注 |
|----|------|---------|------|
| 1 | **安装 Maven 3.9.x** | 10 min | 手动下载 → 配置环境变量 → 验证 |
| 2 | **安装 Docker Desktop** | 20 min | 下载安装包 → 安装 → 启动 Docker Engine |
| 3 | **安装 pnpm** | 2 min | `npm install -g pnpm` → 验证 |
| 4 | **启动 Docker Redis** | 2 min | `docker run -d --name yshop-redis -p 6379:6379 redis:6-alpine` |
| 5 | **创建数据库并导入** | 5 min | CREATE DATABASE → 导入全量 SQL → 导入增量索引 |
| 6 | **修改本地配置** | 5 min | application-local.yaml 密码 + 文件路径 |
| 7 | **创建文件存储目录** | 1 min | `mkdir yshop-data/file` |
| 8 | **Maven 编译** | 5～15 min | `mvn clean install package -Dmaven.test.skip=true` |
| 9 | **启动后端** | 30 sec | `java -jar yshop-server.jar --spring.profiles.active=local` |
| 10 | **验证后端** | 1 min | 访问 Swagger UI |
| 11 | **前端 pnpm install** | 1～3 min | `pnpm install` |
| 12 | **启动前端** | 10 sec | `pnpm dev` |
| 13 | **验证前端** | 1 min | 登录页 → 登录 → 检查菜单 |

> **总计初次启动时间**：约 30～60 分钟（Maven 首次编译依赖网络速度）。

---

## 10. 需要你确认的决策

### 决策 1：Docker Redis 启动方式

- **方案 A（推荐）**：`docker run` 单命令，简单直接
- **方案 B**：独立 `compose-redis.yml` 文件管理

### 决策 2：验证码开关

- **保持 `false`（推荐）**：本地开发无需输入验证码
- **改为 `true`**：需配置验证码密钥，完整模拟生产

### 决策 3：前端端口冲突处理

如果端口 80 被占用：
- **方案 A**：修改 `.env` 中 `VITE_PORT` 为其他端口
- **方案 B**：查找并关闭占用 80 的进程

### 决策 4：Redis 数据持久化

- **不挂载卷（推荐）**：容器停止数据清空
- **挂载卷**：`-v redis-data:/data`

### 决策 5：后续是否需要 Docker Compose 全量部署

- **当前阶段（推荐）**：保持「MySQL 本地 + Redis Docker」
- **未来可选**：修复 `docker-compose.yml` 后切换

请就以上决策给出你的选择，以便调整方案细节。

---

## 11. 后续任务规划（Task 2～6）

### Task 2：安装缺失软件并配置环境

1. 安装 Maven 3.9.x（手动下载或包管理器）
2. 配置 `MAVEN_HOME` 和 `PATH` 环境变量
3. 安装 pnpm（`npm install -g pnpm`）
4. 安装 Docker Desktop for Windows
5. 验证所有软件版本
6. 更新 `memory-bank/decisions.md`

### Task 3：Docker Redis 启动与数据库初始化

1. 启动 Docker Desktop
2. 执行 `docker run` 启动 Redis 6 容器
3. 登录 MySQL 创建 `yshop-crm-open-source` 数据库
4. 导入全量 SQL 脚本
5. 执行增量索引脚本
6. 验证初始化结果

### Task 4：修改本地配置文件

1. 修改 `application-local.yaml` 中 MySQL 密码和 `yshop.file-path`
2. 创建文件存储目录
3. 更新 `.gitignore` 确保 `yshop-data/` 不被提交

### Task 5：编译后端并启动

1. 执行 `mvn clean install package -Dmaven.test.skip=true`
2. 解决编译错误（如有）
3. 启动 Spring Boot 后端
4. 验证 Swagger 文档和 Redis 连通性

### Task 6：启动前端并验证全流程

1. 执行 `pnpm install`
2. 启动前端 `pnpm dev`
3. 验证登录页面、菜单、CRM 页面
4. 验证前后端连通性
5. 更新 `memory-bank/activeContext.md` 和 `memory-bank/progress.md`

---

## 附录：软件下载链接

| 软件 | 下载地址 | 推荐版本 |
|------|----------|----------|
| Maven | https://maven.apache.org/download.cgi | 3.9.9 (Binary zip) |
| Docker Desktop | https://www.docker.com/products/docker-desktop/ | Latest (Windows x86_64) |
| pnpm | `npm install -g pnpm` | Latest (>= 8.6.0) |
| Redis (Docker) | `docker pull redis:6-alpine` | 6-alpine |
| JDK | 已安装 17.0.12 | — |
| MySQL | 已安装 8.0.45 | — |
| Node.js | 已安装 24.16.0 | — |

### 6.4 启动后验证

| URL | 预期结果 |
|-----|----------|
| http://localhost:48081/swagger-ui | Knife4j 接口文档页面 |
| http://localhost:48081/v3/api-docs | OpenAPI JSON |
| http://localhost:48081/actuator/health | `{"status":"UP"}` |

---

## 7. 前端启动步骤

### 7.1 安装依赖

```powershell
cd d:\workspace\yshop-oa-crm\yshop-crm-vue
pnpm install
```

### 7.2 启动

```powershell
pnpm dev
```

### 7.3 验证

| URL | 预期结果 |
|-----|----------|
| http://localhost:80 | 登录页面 |
| 登录 admin / admin123 | 进入主菜单 |

> 默认管理员账号密码来自 SQL 脚本示例数据。

**位置**：第 197 行

**修改前**：

```yaml
yshop:
  captcha:
    enable: false         # 本地环境暂时关闭
```

**建议**：保持 `false`（推荐）—— 本地开发无需每次输入验证码，简化登录流程。如需测试验证码功能再改为 `true`。

### 4.3 无需修改的配置

| 配置 | 说明 | 原因 |
|------|------|------|
| `spring.data.redis.host: 127.0.0.1` | Redis 连接地址 | Docker 映射 6379 到宿主机，`127.0.0.1:6379` 可达 |
| `spring.data.redis.database: 0` | Redis 数据库编号 | 与 Docker Redis 默认 db0 一致 |
| `VITE_BASE_URL: http://localhost:48081` | 前端后端地址 | 后端端口 48081 正确 |
| `VITE_API_URL: /admin-api` | API 前缀 | 与后端路径一致 |
| `VITE_PORT: 80` | 前端端口 | 如 80 被占用再调整 |

### 4.4 前端端口被占用

如果 Windows 上端口 80 已被占用：

**方案 A**：修改 `yshop-crm-vue/.env` 中 `VITE_PORT=8080` 或其它可用端口。

**方案 B**：查找并关闭占用进程：
```powershell
netstat -ano | findstr ":80 "
```
|------|--------|
| **pnpm 版本** | **>= 8.6.0** |
| **选择依据** | `yshop-crm-vue/package.json` `engines.pnpm >= 8.6.0` |
| **类型** | **源码明确要求** |
| **当前状态** | ❌ 需要安装 |

#### pnpm 安装方式

```powershell
# 通过 npm 全局安装（推荐，最简单）
npm install -g pnpm

# 验证
pnpm -v
# 应输出 >= 8.6.0
```

> npm 11.13.0（当前已安装）完全支持全局安装 pnpm。无需单独下载 pnpm 安装包。

### 2.4 MySQL

| 项目 | 推荐值 |
|------|--------|
| **版本** | **8.0.x**（当前已安装 **8.0.45**） |
| **选择依据** | Docker Compose 使用 `mysql:8` 镜像；SQL 文件目标版本为 `9.6.0`，但导出的 SQL 语法在 MySQL 8.0.x 上兼容 |
| **类型** | **源码要求（非强制版本号）** |
| **当前状态** | ✅ 已满足，但需注意以下两点 |

**⚠️ 需要注意的兼容性问题：**

1. **SQL 导出来源为 MySQL 9.6.0**：全量 SQL 文件 `yshop-crm-open-source.sql` 使用 Navicat 从 MySQL 9.6.0 导出。MySQL 8.0.45 导入时，需要注意是否有 MySQL 9.x 特有的语法。经静态检查，该 SQL 文件主要使用标准 DDL/DML（`CREATE TABLE`、`INSERT INTO`、`ALTER TABLE`），暂未发现 MySQL 9.x 特有语法。
2. **`sql_mode` 差异**：MySQL 8.0+ 默认 `sql_mode` 包含 `ONLY_FULL_GROUP_BY`，可能会影响部分查询。你可以保持默认值，运行时如果出现 `GROUP BY` 相关错误再调整。

#### 验证 MySQL 服务状态

```powershell
# 检查 MySQL 服务是否在运行
net start | findstr MySQL

# 或通过 MySQL 客户端连接
mysql -u root -p -e "SELECT VERSION();"
```

### 2.5 Redis

| 项目 | 推荐值 |
|------|--------|
| **版本** | **6.x**（对应 `redis:6-alpine`） |
| **选择依据** | Docker Compose 使用 `redis:6-alpine` 镜像 |
| **类型** | **源码明确要求** |
| **当前状态** | ❌ 需要安装 Docker Desktop 并通过 Docker 运行 |

Redis 通过 Docker 运行（详见 [第 3.2 节](#32-redis-docker-启动方案)）。

### 2.6 Docker Desktop

| 项目 | 推荐值 |
|------|--------|
| **版本** | **最新稳定版** |
| **选择依据** | 需运行 Redis 容器（`redis:6-alpine`） |
| **类型** | **推荐使用** |
| **当前状态** | ❌ 需要安装 |

#### 安装方式

1. 下载地址：https://www.docker.com/products/docker-desktop/
2. 选择：**Docker Desktop for Windows**（x86_64）
3. 安装后注意事项：
   - 安装完成后启动 Docker Desktop，等待 Docker Engine 就绪
   - 建议将 Docker 的 `Settings → Resources → Memory` 设置为至少 2GB（运行 Redis 足够）
   - 验证：打开 PowerShell，执行 `docker --version`