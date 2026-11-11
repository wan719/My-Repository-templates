# 后端技术栈与开发约定

本文是 `backend/` 的初始化参考，保留可复用的技术与开发约定。项目专属模块、接口、账号及设备规则由实际项目另行定义。当前仓库尚未创建后端工程，以下命令和路径是目标结构，不代表已经可运行。

## 定位与边界

- 后端向前端提供业务 API，负责身份与权限校验、业务流程和数据持久化。
- 如果项目接入物联网平台，设备接入、物模型和规则联动等职责应与业务后端明确分界；平台凭据由后端管理，浏览器不直接持有。
- 先写清业务数据与外部平台数据的权威来源，再设计同步和缓存策略。

## 建议技术栈

| 类别 | 初始选型 | 用途 |
| --- | --- | --- |
| 语言 | Java 21 | 编译与运行基线 |
| 框架 | Spring Boot | Web API 与应用配置 |
| 数据访问 | MyBatis-Plus | MySQL 数据访问 |
| 数据库 | MySQL 8 | 业务数据持久化，字符集使用 `utf8mb4` |
| 构建 | Gradle（Groovy DSL） | 通过项目内 Gradle Wrapper 构建 |
| 认证 | 无状态 JWT | 按项目权限模型签发与校验令牌 |
| 接口文档 | SpringDoc OpenAPI | 维护可查阅的接口定义 |
| 接口风格 | REST + JSON | 提供版本化业务 API |

依赖的具体版本与兼容组合应在工程初始化时验证，并以构建文件和锁定机制为准；不要把参考版本号当作已安装或已验证的版本。

## 建议项目结构

采用单模块 Gradle 工程。根包名应替换为项目自己的命名空间，确定一个业务模块后再建立对应目录。

```text
backend/
├── build.gradle
├── settings.gradle
├── gradle/
├── gradlew
├── gradlew.bat
├── .env.example
└── src/
    ├── main/
    │   ├── java/<项目根包>/
    │   │   ├── common/
    │   │   ├── config/
    │   │   └── <业务模块>/
    │   │       ├── controller/
    │   │       ├── service/impl/
    │   │       ├── mapper/
    │   │       ├── entity/
    │   │       ├── dto/
    │   │       └── vo/
    │   └── resources/
    │       ├── application.yaml
    │       ├── db/
    │       └── mapper/
    └── test/java/<项目根包>/
```

- 模块内按 `controller → service → mapper` 调用；Controller 不直接调用 Mapper。
- 跨模块业务调用通过对方 Service，避免直接依赖其 Mapper、实体和接口对象。
- `common/` 与 `config/` 仅放跨模块共享组件及全局配置。

## 配置与数据

- 默认配置集中在 `application.yaml`。数据库地址、密码和 JWT 密钥通过环境变量注入，不提交真实凭据。
- 可提交不含秘密的 `.env.example`；实际 `.env` 应被忽略。环境差异按需通过 Spring Profiles 管理。
- 表名和字段名使用 `snake_case`；主键、时间戳和软删除字段按项目数据模型统一约定。
- 数据库结构变更记录在 `src/main/resources/db/` 的有序 SQL 脚本中；项目需明确这些脚本由什么工具或流程执行。
- MyBatis-Plus 实体、分页插件和自动填充配置应与选定版本及数据库一致。简单查询可使用 Wrapper，复杂查询放在按模块组织的 XML 映射文件中。

## API 与认证

- 使用统一响应结构，并为业务失败定义明确的错误码和可读消息。例如：`{"code":0,"message":"success","data":{}}`。
- 全局异常处理统一转换已知业务异常；意外服务端错误应记录日志并返回恰当的 HTTP 状态，避免泄露内部细节。
- API 使用 `/api/v1` 等版本前缀；URL 使用小写、连字符和复数资源名。具体路径以项目接口设计为准。
- 如采用 JWT，请求通过 `Authorization: Bearer <token>` 携带；令牌只放必要声明，过期时间、角色和撤销策略由项目确定。
- 密码使用可靠哈希算法保存；认证与权限校验集中实现，不分散在各业务接口中。

## 构建、测试与维护

工程初始化后，补充经实际验证的命令。使用 Gradle Wrapper 的项目通常从 `backend/` 运行 `./gradlew build`、`./gradlew test` 和 `./gradlew bootRun`；Windows 可使用 `gradlew.bat`。测试选择及结果报告遵循 [软件测试规范](test.md)。

项目落地时更新本文的实际包名、模块职责、依赖版本、数据库初始化方式、运行命令和 CI 门禁。
