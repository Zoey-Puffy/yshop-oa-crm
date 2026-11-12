# 技术环境上下文

## 本地开发环境（已验证）

| 组件 | 版本 | 路径 / 说明 |
|------|------|-------------|
| JDK | 17.0.12 | `C:\Program Files\Java\jdk-17`；`JAVA_HOME` 已配置 |
| Maven | 3.9.11 | IntelliJ IDEA 2026.1 内置，已添加到系统 PATH |
| MySQL | 8.0.45 | Community Server，本地服务运行中 |
| Redis | — | 通过 Docker 运行 `redis:6-alpine` 容器 |
| Node.js | 24.16.0 | `C:\Program Files\nodejs` |
| npm | 11.13.0 | 随 Node.js 自带 |
| pnpm | 12.18.0 | 全局安装（通过 npm） |
| Docker Desktop | 29.9.3 | 已安装，可用于运行 Redis 容器 |
| Git | — | 已安装并在 PATH 中 |
| IntelliJ IDEA | 2026.1 | `C:\Program Files\JetBrains\IntelliJ IDEA 2026.1` |

## 环境变量配置

| 变量 | 值 | 配置位置 | 状态 |
|------|-----|----------|------|
| `JAVA_HOME` | `C:\Program Files\Java\jdk-17` | 系统环境变量 | ✅ 已配置 |
| `PATH` (Maven) | `C:\Program Files\JetBrains\IntelliJ IDEA 2026.1\plugins\maven\lib\maven3\bin` | 追加到用户 PATH | ✅ 已配置 |

## 项目技术栈

- **后端**：Spring Boot 3.2.2、MyBatis-Plus 3.5.5、Lombok 1.18.30、MapStruct 1.5.5
- **前端**：Vue 3 + Vite、axios
- **数据库**：MySQL 8（本地）、Redis 6（Docker）
- **构建**：Maven 3.9.11、pnpm 12.18.0

## 架构决策记录

| 决策 | 选择 | 说明 |
|------|------|------|
| Redis 启动方式 | `docker run` | 简单直接，无需 docker-compose |
| 验证码开关 | `false` | 本地开发不启用验证码 |
| 前端端口冲突 | 改端口 | 80 被占用时调整 `VITE_PORT` |
| Redis 数据持久化 | 容器停止即清空 | 开发环境无需持久化 |
| 后续全量 Docker Compose | MySQL 本地 + Redis Docker | 保持现状 |