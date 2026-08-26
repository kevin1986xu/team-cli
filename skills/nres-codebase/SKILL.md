---
name: nres-codebase
description: 在 nres Java/Spring Boot 业务仓库中定位模块、追踪接口和数据流、评估改动影响、处理配置与 Jasper/JRXML 模板并选择可靠的构建验证方式。当任务明确针对 nres 项目的分析、实现、排障或测试时使用，不用于其他仓库。
---

# nres 代码库工作指南

使用本技能快速缩小搜索范围，但始终以当前工作树中的代码为事实来源。本技能的统计基线来自提交 `fea164b206d433cf6857ee175e7bc15807d1a4f3`；当 `git rev-parse HEAD` 不同时，重新验证与任务有关的结论。

## 按需读取

- 需要定位业务模块、入口、数据访问或集成边界时，读取 [references/architecture.md](references/architecture.md)。
- 需要编译、测试、打包或评估验证强度时，读取 [references/build-and-verification.md](references/build-and-verification.md)。
- 需要处理运行配置、凭据、Jasper/JRXML 或 PDF 模板时，读取 [references/config-and-templates.md](references/config-and-templates.md)。

## 工作原则

1. 先确认仓库根目录包含 `pom.xml` 和 `src/main/java/com/geostar/nres/bussiness/NresApplication.java`。不要依赖当前 `README.md`，它仍是 GitLab 初始模板。
2. 根据任务先选定 `bussiness` 下的顶层模块，再从 Controller 追踪到 Service/Impl、`DbBuilder` 查询或外部集成。保留代码中已存在的 `bussiness` 拼写，不要为了纠正单词扩大改动。
3. 对路由、SQL、调度任务和外部服务使用 `rg` 做精确搜索，避免一次加载超大的 Service 或 DAO 文件。
4. 不展示、复制或提交配置文件中的密码、app secret、私钥、token 或内网端点值。需要说明配置时只引用键名。
5. 不把 Java 8 目标、Spring Boot 2.7.10、WAR 部署或私有依赖当作可顺手现代化的内容。升级任务需要独立的兼容性范围和测试计划。
6. 尊重用户给定的写入边界。只读分析、诊断或文档生成不授权修改业务代码。

## 结果标准

- 结论引用相对文件路径和类/方法符号；对只由包名或命名推断的业务含义明确标注为推断。
- 先运行与变更面对应的最小验证，再决定是否需要完整 Maven 构建。
- 如果没有自动化测试覆盖目标路径，明确报告这一点，不把“编译通过”表述为“功能已验证”。
