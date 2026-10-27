# 3.x 客户端迁移后的源码构建

当前 `easy-es-common` 已依赖 `elasticsearch-java` 和 Jackson。遗留的 `FastJsonUtils`、`RestHighLevelClientBuilder`、`RestHighLevelClientUtils` 仍引用未声明的 Fastjson 1 和旧高级客户端；其中旧客户端参数还与现有 `EEVersionUtils` 接口不匹配，导致源码构建失败。仓库其他 Java 源码没有调用这三个类。

本次移除这三个遗留源文件，使用已经存在的 `EsClientUtils`、`ElasticsearchClient` 和 Jackson 工具链，不重新引入旧客户端或 Fastjson 依赖，也不在编译器中隐藏排除文件。

## 对下游使用者的影响

这三个类原本是 public。显式导入它们的下游代码需要迁移，因此不能将移除描述为二进制兼容变更：

- 客户端创建与获取使用现有 `EsClientUtils` / `ElasticsearchClient` API；旧高级客户端类型不能直接传入新客户端 API。
- 依赖 `SimplePropertyPreFilter` 的代码需要按业务需求迁移至 Jackson 的字段过滤或自定义 ObjectMapper 配置；两套过滤器没有直接类型兼容性。

如果仍需提供旧 API，应在维护者确认后设计带有明确依赖的独立兼容层。合并前应确认项目 3.x 的兼容范围。

## 构建

在 JDK 17 与 Maven 3.9 下，从仓库根目录执行：

```shell
mvn -B -ntp -P!release -Dproject.build.sourceEncoding=UTF-8 -Dmaven.compiler.encoding=UTF-8 -DskipTests package
```

该命令编译框架、示例及测试源码并打包全部模块，但明确不执行测试，不等于所有集群集成测试通过。`!release` 关闭默认发布附加步骤；UTF-8 参数避免 Windows 平台默认编码影响中文源码。实际功能测试应另行选择目标测试类并配置专用测试服务。
