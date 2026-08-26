# 构建与验证

## 构建契约

- 使用 Maven，仓库中没有 `mvnw`。
- `pom.xml` 目标 Java 8，继承 Spring Boot `2.7.10`，产物为 `target/nres.war`。
- `vendor-repo/` 是项目级 Maven 仓库，用于 `com.geostar`、`com.geostar.nres` 等私有依赖。不要随意删除、升级或改为公网坐标。
- `src/main/resources` 启用 Maven filtering。修改资源处理时检查二进制文件和 `${...}` 占位符是否被意外处理。
- Spring Boot Maven 插件开启 `fork` 和 `includeSystemScope`，打包行为与普通纯 jar 项目不同。
- 当前基线需要 JDK 8 才能编译：`QlrDto` 和 `PreRegisterDto` 引用 `jdk.nashorn.internal.ir.annotations.Ignore`。实测 JDK 18 因 Nashorn 内部包已移除而失败，Zulu JDK `1.8.0_462` 下成功。除非这两处引用已被正式替换，不要使用高版本 JDK 声称基线可构建。
- JDK 8 编译会对 `AssistOfficeServiceImpl` 和 `wangban/utils/FileUtils` 的 `sun.misc.BASE64Encoder/BASE64Decoder` 引用报内部 API 警告。这不是基线构建失败，但是 JDK 升级时必须处理的兼容性项。

## 验证命令

从最小证据逐步增加强度：

```bash
# macOS 已安装 JDK 8 时
export JAVA_HOME="$(/usr/libexec/java_home -v 1.8)"
export PATH="$JAVA_HOME/bin:$PATH"
export MAVEN_OPTS="${MAVEN_OPTS:+$MAVEN_OPTS }-Dfile.encoding=UTF-8"

# 编译主代码并执行已存测试
mvn test

# 验证 WAR 打包和资源处理
mvn package

# 查看产物，不启动外部服务
ls -lh target/nres.war
```

不要仅为了“测一下”盲目启动应用。完整运行需要数据库、Redis、ActiveMQ、Nacos/Dubbo 和多个内外部端点，而且启动路径会禁用 SSL 证书校验。

## 对测试结果的解读

- 基线没有 `src/test`，因此 `mvn test` 成功主要证明主代码能编译和测试生命周期能运行，不证明业务行为正确。
- 修改纯 Java 逻辑时，应为可隔离的逻辑添加针对性测试；如果无法隔离私有基础设施，报告具体缺口。
- Controller/Service/SQL 路径变更需要至少对目标分支做参数、返回值和副作用核对，不应仅依赖 Maven 成功。
- JRXML/Jasper/PDF 变更需要产物级渲染或编译检查，只做 XML 语法检查不足以证明布局和字段正确。

## 环境检查

```bash
mvn -version
java -version
git status --short
```

如果 Maven 使用的 JDK 与命令行 `java` 不同，以 `mvn -version` 报告的 Java 为实际构建环境。当新 JDK 与旧依赖或 Java 8 目标不兼容时，优先切换到团队验证过的 JDK，不要通过随意改 `pom.xml` 来掩盖环境问题。
当工作区路径包含中文时，JDK 8 默认 US-ASCII 会让 Maven 无法读取 `target/maven-status` 中的增量状态文件；使用上述 `MAVEN_OPTS` 并执行一次 `mvn clean ...` 重建状态。

验证后检查 `git status --short`。`target/` 应被忽略；不要把本地配置、日志、密钥或构建产物加入提交。
