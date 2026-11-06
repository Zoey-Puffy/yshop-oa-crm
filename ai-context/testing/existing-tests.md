# 已有测试情况

> 统计时间基准：当前工作区源码。统计口径：仅计 `src/test/` 下的 Java 文件与 `@Test` 注解，已排除 `target/`。

## 1. 结论（CRM 模块）

| 项 | 结果 | 依据 |
|----|------|------|
| `yshop-module-crm-biz` 测试目录 | **不存在**（无 `src/test`） | `Test-Path` 检查 |
| `yshop-module-crm-api` 测试目录 | **不存在** | 同上 |
| CRM 模块 `@Test` 数量 | **0** | 全模块检索 |
| CRM 模块测试依赖 | 已声明 `yshop-spring-boot-starter-test`（scope=test） | `yshop-module-crm-biz/pom.xml` |

> 即：CRM 模块**具备测试依赖，但没有任何测试代码**，后续新增单元测试可直接复用项目已有的测试基类（见第 3 节）。

## 2. 全仓库测试分布

| 模块 | 测试 Java 文件 | `@Test` 数量 | 备注 |
|------|----------------|--------------|------|
| `yshop-framework`（含 `yshop-common` 与若干 starter） | 14 个文件 | 58 | `biz-data-permission`（7 类）、`biz-ip`、`excel`、`web`、`yshop-common` 等 |
| `yshop-module-system` | 36 个文件 | 436 | 覆盖 service、oauth2、sms、dict、user、tenant、permission 等，是项目中测试最完整的模块 |
| `yshop-module-mall` | 1 个文件（`yshop-module-shop-biz`，`MaterialServiceImplTest`） | 7 | 商城素材服务 |
| `yshop-server` | 1 个文件（`ProjectReactor.java`） | 0 | **不是单元测试**，是项目重命名工具（含 `main` 方法） |
| `yshop-module-crm` | 0 | 0 | 见第 1 节 |
| `yshop-module-infra`、`yshop-module-pay`、`yshop-module-mp`、`yshop-module-message`、`yshop-module-product` | 未检索到 `src/test` | — | 待验证 |

> 注：`yshop-module-system` 的测试文件数字为 `src/test/java` 下的 `*.java` 数量。

## 3. 测试框架与基类

### 3.1 依赖（已确认）

- JUnit 5（由 `spring-boot-starter-test` 提供）。
- Mockito：`mockito-inline 5.2.0`（支持 final 类与静态方法 mock），版本定义在 `yshop-dependencies/pom.xml`。
- H2 内存数据库（`h2`）、`jedis-mock`（Redis 模拟）、`podam`（随机数据生成，`RandomUtils.randomPojo` 使用）。
- surefire 插件版本 `3.2.2`（根 `pom.xml` 中 `maven-surefire-plugin.version`）。

依赖声明位置：
- `yshop-crm/yshop-dependencies/pom.xml`（版本管理）
- `yshop-crm/yshop-framework/yshop-spring-boot-starter-test/pom.xml`（测试 starter 的实际依赖）

### 3.2 测试基类（`yshop-spring-boot-starter-test`，已确认）

路径：`yshop-crm/yshop-framework/yshop-spring-boot-starter-test/src/main/java/co/yixiang/yshop/framework/test/core/ut/`

| 基类 | 用途 | 关键注解/机制 |
|------|------|----------------|
| `BaseDbUnitTest` | 依赖 H2 内存库的单元测试（Mapper / Service） | `@SpringBootTest(webEnvironment = NONE)`、`@ActiveProfiles("unit-test")`、`@Sql(scripts = "/sql/clean.sql", AFTER_TEST_METHOD)` 清理数据；导入 `YshopDataSourceAutoConfiguration`、`YshopMybatisAutoConfiguration`、MyBatis-Plus 与 Join 配置 |
| `BaseMockitoUnitTest` | 纯 Mockito 单元测试 | `@ExtendWith(MockitoExtension.class)` |
| `BaseRedisUnitTest` | 依赖 Redis 的单元测试 | 使用 `jedis-mock` / 测试 Redis 配置（`RedisTestConfiguration`） |
| `BaseDbAndRedisUnitTest` | 同时依赖 DB 与 Redis | 组合上述两者 |

工具类（`yshop-spring-boot-starter-test/.../core/util/`）：
- `RandomUtils`：`randomPojo(Class, Consumer)`、`randomCommonStatus()` 等，用于生成随机测试数据。
- `AssertUtils`：`assertPojoEquals(expected, actual, ignoreFields...)`、`assertServiceException(() -> ..., ErrorCode)` 等。

### 3.3 测试资源（`yshop-module-system-biz/src/test/resources/`）

| 文件 | 作用 |
|------|------|
| `application-unit-test.yaml` | 单元测试配置：H2（`jdbc:h2:mem:testdb;MODE=MYSQL`）、Redis 端口 16379、MyBatis 懒加载、关闭 Banner |
| `sql/create_tables.sql` | H2 建表脚本（由 `spring.sql.init.schema-locations` 加载） |
| `sql/clean.sql` | 每个用例结束后清理数据 |
| `logback.xml` | 日志配置 |

> 注意：`application-unit-test.yaml` 中的 Redis 配置为 `127.0.0.1:16379`，`BaseDbUnitTest` 依赖的 `unit-test` profile 位于 `yshop-module-system-biz`，**其它模块若要使用 `BaseDbUnitTest`，需要在本模块的 `src/test/resources` 下提供同名的 `application-unit-test.yaml` 与 `sql/create_tables.sql`**（待验证：`BaseDbUnitTest` 的 `@ActiveProfiles` 是否在其它模块生效，需要实际运行确认）。

## 4. 现有测试写法（以 `DeptServiceImplTest` 为例）

来源：`yshop-crm/yshop-module-system/yshop-module-system-biz/src/test/java/co/yixiang/yshop/module/system/service/dept/DeptServiceImplTest.java`

- 类注释：`{@link DeptServiceImpl} 的单元测试类`，并附 `@author`。
- 注解：`@Import(DeptServiceImpl.class)`，类继承 `BaseDbUnitTest`。
- 注入：`@Resource` 注入被测 Service 与 Mapper（真实 H2 数据）。
- 方法命名：`testCreateDept`、`testUpdateDept`……（`test + 方法名`）。
- 数据准备：`randomPojo(DeptSaveReqVO.class, o -> {...})`，关键字段覆盖（如 `setId(null)`、`setStatus(randomCommonStatus())`）。
- 断言：`assertNotNull`、`assertPojoEquals(reqVO, deptDO, "id")`、`assertServiceException(() -> ..., ERROR_CODE)`。
- 测试前置数据：`deptMapper.insert(...)`（代码注释中注明 `@Sql` 思路，实际为直接 insert）。

## 5. 测试规范（从现有代码归纳，已确认）

1. **测试类位置**：与被测类同包路径，放在 `src/test/java` 下，类名为 `<被测类名>Test`（如 `DeptServiceImplTest`）。
2. **测试基类**：Service 层、涉及 Mapper 的测试继承 `BaseDbUnitTest`；纯逻辑使用 `BaseMockitoUnitTest`。
3. **随机数据**：优先使用 `RandomUtils.randomPojo`，避免硬编码。
4. **断言**：使用 `AssertUtils` 的 `assertPojoEquals` 与 `assertServiceException`，异常码使用 `ErrorCodeConstants` 中的常量。
5. **清理**：依赖 `BaseDbUnitTest` 的 `@Sql` 每次结束清理，不需要手动删除。

## 6. 如何为 CRM 模块补充测试（建议，待实施）

以下为基于现有规范的建议步骤，**本次任务未创建任何测试文件**：

1. 在 `yshop-crm/yshop-module-crm/yshop-module-crm-biz/src/test/resources/` 下复制 `application-unit-test.yaml`、`sql/create_tables.sql`、`sql/clean.sql`（需确认 CRM 相关表是否已在 `create_tables.sql` 中；当前 system-biz 的 `create_tables.sql` 仅覆盖 system 模块表，**CRM 表需要补充到 H2 建表脚本中，待验证**）。
2. 在 `yshop-crm-biz` 的 `pom.xml` 已有 `yshop-spring-boot-starter-test`（test scope），无需新增依赖。
3. 优先为以下高价值 Service 编写测试：
   - `CrmCluesServiceImpl#openClues` / `receiveClues`（状态流转）
   - `CrmCustomerServiceImpl#deleteCustomer`（关联约束）
   - `CrmFlowServiceImpl#sendNextStep`（审批推进）
   - `CrmInvoiceServiceImpl#preCheck`（开票金额约束）
   - `CrmContractServiceImpl#createContract` 与 `deleteContract`（状态约束）
4. 注意 `CrmContractServiceImpl` 使用 `RedissonClient`，需 mock 或使用 `BaseDbAndRedisUnitTest`。

## 7. 其它测试相关事项（待验证）

- 根 `pom.xml` 未显式配置 surefire 的 include/exclude，默认只执行 `*Test.java`、`Test*.java`、`*Tests.java`、`*TestCase.java`。`ProjectReactor` 不符合默认规则，**不会被当作测试执行**（该类本身也无 `@Test`）。
- 前端 `yshop-crm-vue` **无任何单元测试**：`package.json` 中未声明 vitest、jest、cypress 等测试依赖，`scripts` 中也无 `test` 命令（已确认）。
- 前端仅有 lint（eslint、stylelint、prettier）与 `ts:check`（`vue-tsc --noEmit`）。
