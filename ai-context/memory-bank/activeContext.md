# 当前开发状态

AI 知识库已建立，12 份文档已就绪。本地开发环境已准备完毕。

## 当前模块

- 知识库位于 `ai-context/` 目录
- 入口文件：`ai-context/README.md`

## 已完成

### 第一阶段：AI 知识库创建（已完）
1. AI 知识库 12 份文档全部创建完成
2. 接口清单章节顺序已修复
3. README 路径错误已修正

### 第二阶段：本地开发环境准备（已完成）
| 软件 | 版本 | 状态 |
|------|------|------|
| JDK | 17.0.12 | ✅ 已存在 |
| Maven | 3.9.11 | ✅ 通过 IntelliJ IDEA 内置 Maven 配置到 PATH |
| MySQL | 8.0.45 | ✅ 已存在 |
| Node.js | 24.16.0 | ✅ 已存在 |
| npm | 11.13.0 | ✅ 已存在 |
| pnpm | 12.18.0 | ✅ 已存在 |
| Docker Desktop | 29.9.3 | ✅ 已存在 |
| IntelliJ IDEA | 2026.1 | ✅ 已存在 |

### 环境变量配置
- `JAVA_HOME = C:\Program Files\Java\jdk-17` ✅
- Maven bin 目录已追加到用户 PATH ✅

## 正在进行

等待后续任务：
- 创建数据库并导入 SQL
- 修改本地配置（数据库密码、Redis、文件路径等）
- Maven 编译
- 启动后端
- 前端安装依赖并启动
- 登录验证

## 下一步

按 startup-plan.md 执行：
1. 创建数据库 `yshop-crm-open-source` 并导入全量 SQL + 增量索引
2. 修改 `application-local.yaml` 配置
3. Maven 编译：`mvn clean install package -Dmaven.test.skip=true`
4. 启动后端
5. 前端启动：`pnpm dev`

## 注意事项

- 知识库不包含任何密钥、密码、Token，仅记录配置位置
- `application-local.yaml` 中 `yshop.file-path` 为 macOS 路径，Windows 环境需修改
- Dockerfile 使用 JRE 21，本地开发为 JDK 17，需留意差异
- 打开新终端窗口后，`mvn` 命令自动可用（已持久化到用户 PATH）

## 更新频率

每次任务结束
尽量控制在一页以内