# 配置与模板资源

## 配置文件

主要运行配置位于：

- `src/main/resources/application.properties`
- `src/main/resources/application.yml`
- `src/main/resources/application.ymlxc`
- `src/main/resources/logback-spring.xml`

`application.ymlxc` 的加载方式从文件名无法确认，只有找到加载器或部署流程证据后才能声称它生效。

配置键覆盖以下敏感或外部边界：

- 多数据源的 URL、用户名和密码。
- Redis、ActiveMQ、Elasticsearch、Nacos/Dubbo 和对象存储。
- 微信 app secret、签章/CA 凭据、app key、传输加密公私钥。
- IOS/IGS/登记/受理/共享等内网服务端点。
- 大量调度任务和监控功能开关。

安全查看配置时只列出键名，例如：

```bash
awk -F'[=:]' '/^[[:space:]]*[^#![:space:]][^=:]*[=:]/{print $1}' \
  src/main/resources/application.properties
rg -n '^[[:space:]]*[A-Za-z0-9_.-]+:' src/main/resources/application.yml
```

不要在回复、测试日志、skill 或中央资源仓库中包含任何配置值。如果必须修复泄露的凭据，先旋转凭据，再做代码或历史处理。

## Jasper/JRXML 和 PDF

基线快照中，仓库根目录附近有 11 个 `.jrxml` 和 8 个 `.jasper` 文件，主要在：

- `template-fixes/`
- `修改后的表单/`

这两个目录不在 `src/main/resources`。除非代码、构建脚本或部署流程给出证据，不要假定它们会自动进入 WAR。

- `.jrxml` 是可审查的 XML 源文件，应优先从它追踪字段、参数、表达式和布局。
- `.jasper` 是编译后的二进制产物，不要用文本方式编辑。
- `pom.xml` 使用 JasperReports `6.0.0`。重新编译模板时保持与目标运行时兼容，并比对生成 PDF 或打印结果。
- Java 生成式 PDF/文档模板大量位于 `src/main/java/com/geostar/nres/bussiness/manage/template/pdf/`，不要只搜索 `.jrxml`。

## 模板变更的最小核对

1. 搜索同名 `.jrxml`、`.jasper` 和 Java 调用方，确定真正的运行时来源。
2. 检查字段名、参数名和表达式类型是否与数据生成代码一致。
3. 对 JRXML 做 XML 解析检查，并使用兼容的 JasperReports 版本重新编译。
4. 使用可控的脱敏样例数据渲染，人工检查字段、分页、中文字体和签章位置。
5. 报告未验证的运行环境、数据来源或打印设备差异。
