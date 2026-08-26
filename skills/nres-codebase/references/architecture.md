# 代码库导航与架构边界

## 基线快照

以 Git 提交 `fea164b206d433cf6857ee175e7bc15807d1a4f3` 为基线：

- 单 Maven 工程，坐标 `com.geostar:nres:3.1.1`，打包类型为 WAR，最终名称为 `nres`。
- Spring Boot `2.7.10`，`pom.xml` 目标 Java `1.8`。
- 约 2,148 个 Java 源文件、441,927 行 Java，没有 `src/test`。这些数字只用于判断搜索和验证策略，不是持久不变的指标。
- 主入口为 `src/main/java/com/geostar/nres/bussiness/NresApplication.java`。
- `README.md` 仍是 GitLab 默认模板，不能作为构建、部署或业务事实依据。

## 启动边界

`NresApplication` 同时支持 `main` 方法和外部 Servlet 容器的 WAR 启动，并有以下非默认行为：

- 排除 `DataSourceAutoConfiguration` 和 `HibernateJpaAutoConfiguration`，数据源不能按普通 Spring Boot 默认自动配置来推断。
- 组件扫描包含 `com.geostar.nres` 和 `com.geostar.geoios`，后者来自私有依赖。
- 启用异步方法。
- `main` 与 WAR `configure` 路径都调用 `SslVerification.disableSslVerification()`。这是已存安全边界；不要在新代码中扩散该模式。
- 注册 UTF-8 `CharacterEncodingFilter`。

## 顶层业务模块

Java 代码集中在 `src/main/java/com/geostar/nres/bussiness/`。下表的业务含义部分来自包名，必须通过目标 Controller/Service 再次验证。

| 模块 | 基线文件数 | 导航线索 |
| --- | ---: | --- |
| `bdc` | 618 | 不动产登记核心，Controller、流程、证书、打印和规则相关服务 |
| `wangban` | 463 | 网办接口、申请/查询和离线查询；`dao/InternetDao.java` 是高频数据入口 |
| `manage` | 301 | 管理、同步、调度、文档与 PDF 模板 |
| `jiaoyi` | 265 | 交易、合同、测绘和资金监管 |
| `extend` | 168 | 地区化和项目扩展，包括 `hubei`、`zhejiang`、`shandong`、`shennongjia`、`jiangxi` |
| `escrow` | 143 | 资金托管/监管 |
| `share` | 118 | 外部集成，包括电子签章、CA、腾讯云等 |
| `reap` | 71 | 受理和附件创建相关路径 |

## 交互与数据访问形态

- 基线中有 104 个 `@RestController` 类，是定位 HTTP 行为的首选入口。
- 路由以 POST 为主：基线约有 1,189 个 `@PostMapping`、88 个 `@GetMapping` 和 105 个类/方法级 `@RequestMapping`。
- 数据访问的主要可见模式是私有 `DbBuilder` API，基线中约 153 个 Java 文件引用它。不要假定项目使用 MyBatis、JPA 或 `JdbcTemplate`。
- 项目还有 JMS (`@JmsListener`)、Dubbo、`RestTemplate`、CXF/WebService 和多种云存储/签章集成。改动 Service 时检查是否跨越这些边界。
- 存在 6 个含 `@Scheduled` 的类，部分 Controller 也直接挂调度方法。查找对应开关和 cron 键后才能判断运行时影响。

## 定位流程

1. 用业务词和路由同时搜索 Controller：`rg -n '@(Get|Post|Request)Mapping|<business-term>' src/main/java/.../controller`。
2. 从注入字段或构造参数找到 Service 接口和实现，留意同时存在 `common`、`v2`、`workflow` 和 `rulebase` 变体。
3. 搜索目标方法中的 `DbBuilder`、外部 URL 配置键、JMS/Dubbo 调用和文档/模板路径。
4. 检查同名类、V2 实现、地区扩展和调度入口，防止只修改一条并行路径。
5. 超大类应按方法或符号读取；多个 Service/DAO 超过 5,000 行，全文加载会淹没实际依赖。
