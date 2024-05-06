---
title: 编写StarRocks的自定义函数
slug: StarRocks01
categories:
  - 大数据技术
tags:
  - StarRocks
halo:
  site: http://156.224.24.61:8090/
  name: 391f235e-5bd3-491d-aaee-d7e108f816d2
  publish: true
---
## 前提条件

StarRocks使用udf函数需要满足以下条件:

* 安装jdk1.8

* 开启udf功能，在FE的配置文件**fe/conf/fe.conf**中设置配置项`enable_udf`为`true`，并且重启FE节点使配置生效

## 开发使用UDF函数

创建maven项目，并且用java实现udf函数

1. **创建maven项目并且添加以下的pom依赖:**

    ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <project xmlns="http://maven.apache.org/POM/4.0.0"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
            <modelVersion>4.0.0</modelVersion>

            <groupId>org.example</groupId>
            <artifactId>udf</artifactId>
            <version>1.0-SNAPSHOT</version>

            <properties>
                <maven.compiler.source>8</maven.compiler.source>
                <maven.compiler.target>8</maven.compiler.target>
            </properties>

            <dependencies>
                <dependency>
                    <groupId>com.alibaba</groupId>
                    <artifactId>fastjson</artifactId>
                    <version>1.2.76</version>
                </dependency>
            </dependencies>

            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-dependency-plugin</artifactId>
                        <version>2.10</version>
                        <executions>
                            <execution>
                                <id>copy-dependencies</id>
                                <phase>package</phase>
                                <goals>
                                    <goal>copy-dependencies</goal>
                                </goals>
                                <configuration>
                                    <outputDirectory>${project.build.directory}/lib</outputDirectory>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-assembly-plugin</artifactId>
                        <version>3.3.0</version>
                        <executions>
                            <execution>
                                <id>make-assembly</id>
                                <phase>package</phase>
                                <goals>
                                    <goal>single</goal>
                                </goals>
                            </execution>
                        </executions>
                        <configuration>
                            <descriptorRefs>
                                <descriptorRef>jar-with-dependencies</descriptorRef>
                            </descriptorRefs>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </project>
    ```

2. **开发UDF:**

    * **Scalar UDF:**  

        Scalar udf也就是即用户自定义标量函数，可以对单行数据进行操作，输出单行结果。当您在查询时使用 Scalar UDF，每行数据最终都会按行出现在结果集中。典型的标量函数包括 `UPPER`、`LOWER`、`ROUND`、`ABS`。
        \
        其中主要是通过写一个`evaluate`函数即可。比如现在创建一个获取json全部key的函数`string _get_json_key_as_str(string)`，那么代码就如下：

        ```java
        public class GetJsonKeyAsString {
            public final String evaluate(String text) {
                if (text == null || text.isEmpty()) {
                    return null;
                }

                JSONObject jsonObject = JSON.parseObject(text);

                if (!jsonObject.keySet().isEmpty()) {
                    return jsonObject.keySet().toString();
                } else {
                    return null;
                }

            }
        }
        ```

        udf函数不需要继承或者实现什么接口，只能给定规范的函数名即可，不过要注意输出和输出的数据类型。

    * **UDAF函数:**

    * **UDWF函数:**

    * **UDTF函数:**

3. **打包和上传自定义函数jar包:**

    使用package命令打包会生成两个文件：**udf-1.0-SNAPSHOT.jar**和**udf-1.0-SNAPSHOT-jar-with-dependencies.jar**。需要将**udf-1.0-SNAPSHOT-jar-with-dependencies.jar**包上传到服务器上，这个服务器可以是基于nginx的，也可以是python搭建的简单服务器，前提是FE和BE都可以进行HTTP访问。

4. **在StarRocks中创建udf函数:**

    StarRocks 内提供了两种 Namespace 的 UDF：一种是 Database 级 Namespace，一种是 Global 级 Namespace。

    * 如果没有特殊的 UDF 可见性隔离需求，您可以直接选择创建 Global UDF。在引用 Global UDF 时，直接调用Function Name 即可，无需任何 Catalog 和 Database 作为前缀，访问更加便捷

    * 如果有特殊的 UDF 可见性隔离需求，或者需要在不同 Database下创建同名 UDF，那么你可以选择在 Database 内创建 UDF。此时，如果您的会话在某个 Database 内，您可以直接调用 Function Name 即可；如果您的会话在其他 Catalog 和 Database 下，那么您需要带上 Catalog 和 Database 前缀，例如: `catalog.database.function`

    > **注意：**
    创建 Global UDF 需要有 System 级的 CREATE GLOBAL FUNCTION 权限；创建数据库级别的 UDF 需要有数据库级的 CREATE FUNCTION 权限；使用 UDF 需要有对应 UDF 的 USAGE 权限

    **语法**

    ```sql
    CREATE [GLOBAL] FUNCTION [DB]._get_json_key_as_str(string) 
    RETURNS string
    PROPERTIES (
        "symbol" = "com.starrocks.udf.sample.GetJsonKeyAsStr", 
        "type" = "StarrocksJar",
        "file" = "http://http_host:http_port/udf-1.0-SNAPSHOT-jar-with-dependencies.jar"
    );
    ```

    |参数|描述|
    |:-:|:-:|
    |symbol|UDF所在项目的类名。格式为`<package_name>.<class_name>`|
    |type|用于标记所创建的UDF类型。取值为`StarrocksJar`，表示基于Java的UDF|
    |file|UDF所在Jar包的HTTP路径。格式为`http://<http_server_ip>:<http_server_port>/<jar_package_name>`|

    > **注意：**
    在使用`show functions`的时候，如果创建的是global的函数，那么就会出现`unknow databases DB`的错误，创建的是DB的函数那么就没问题

## 查看UDF信息

执行以下的命令

```sql
SHOW [GLOBAL] FUNCTIONS;
```

## 删除UDF

执行以下的命令

```sql
DROP [GLOBAL] FUNCTION <function_name>(arg_type [, ...]);
```

例如：删除上面的函数代码如下：

```sql
drop function db._get_json_key_as_str(string);
```
