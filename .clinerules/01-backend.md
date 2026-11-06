---
paths:
  - "**/*.java"
  - "**/*.xml"
  - "**/pom.xml"
---

# 后端开发规则

1. 优先沿用项目已有的分层结构。
2. 遵循 Controller、Service、Mapper 分层。
3. 优先使用项目已有的 MyBatis-Plus 实现。
4. 不要擅自将 MyBatis-Plus 改为 JPA。
5. 新增业务逻辑时考虑参数校验和异常处理。
6. 优先补充 JUnit 测试。
7. 修改接口时检查前端调用关系。