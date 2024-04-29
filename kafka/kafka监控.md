---
title: kafka监控
categories:
  - 大数据技术
  - 消息队列
tags:
  - kafka
  - 运维
  - 指标监控
halo:
  site: http://156.224.24.61:8090
  name: d853bf7a-b003-4cce-a62f-dd62177e885f
  publish: true
---
## kafka使用Prometheus、Grafana和kafka_exporter来构建kafka指标监控

### 问题背景

在实时场景下，对于数据积压是很常见的，我们更希望如何去快速知道有没有数据积压，目前消费了多少，速度怎么样，趋势如何。可以使用原生命令`kafka-consumer-groups.sh --bootstrap-server node01:9092,node02:9092,node03:9092 --group test --describe`来查看当前group所消费的topic的进度如何，如果group很多，topic很多，这样使用命令的效率反而会更低

### 解决方案

将Kafka的度量指标暴露给监控系统，以便进行数据收集、分析和可视化展示。使用**kafka-exporter、Prometheus**和**Grafana**这一组合来进行Kafka监控的原理和流程大致如下：

#### 原理概述

1. **kafka-exporter:** kafka-exporter是一个代理或者中间件，它的主要任务是将Kafka通过**JMX**暴露的指标转换成Prometheus能够理解的格式。Kafka内部使用**Yammer Metrics**来收集各种性能和状态指标，并通过**JMX**暴露出来。kafka-exporter通过连接到**Kafka Broker**的**JMX**端口，读取这些指标，并以**HTTP**端点的形式提供给Prometheus拉取

2. **Promethues:** Promethues是一个开源的监控系统，可以定期地从目标拉取指标数据，Prometheus负责收集这些数据，并存储在本地数据库中，支持查询语言 PromQL 进行灵活的数据检索和聚合运算

3. **Grafana:** Grafana是一个强大的可视化平台，可以连接到Prometheus作为数据源，用来展示和分析监控数据。用户可以在Grafana中创建仪表板，通过各种图表和面板直观地展示Kafka的性能指标，如消息吞吐量、延迟、消费者滞后等

### 监控部署

#### kafka-exporter部署

1. **github上下载[官网安装包](https://github.com/danielqsj/kafka_exporter)**

    ![kafka_github](http://156.224.24.61:8090/upload/kafka_exporter.png)

2. **解压安装包**

    ```shell
    tar -zxvf kafka_exporter-1.7.0.linux-amd64.tar.gz -C kafka_exporter-1.7.0
    ```

3. **前台启动程序**
  
    ```shell
    ./kafka_exporter --kafka.server=kafka:9092 [--kafka.server=another-server ...]
    ```

4. **去web上查看是否成功生成metrics数据**

   如果生成如下的结果，那就说明metrics数据已经采集到了
   ![kafka_metrics](http://156.224.24.61:8090/upload/kafka_metrics.png)

#### Prometheus采集metrics数据

1. **配置数据采集地址**

    在配置好监控数据源以后，现在就要告诉Prometheus应该去哪里采取数据给存放到本地中。打开Promethues的配置文件，增加kafka-exporter暴露的地址即可
    执行以下代码，代开配置文件

    ```shell
    vim promethues.yml
    ```

    在文件末尾添加以下代码

    ```shell
    - job_name: kafka-pro
        static_configs:
            - targets: ['10.10.1.27:9308']
                labels:
                instance: kafka-pro
    ```

    这段代码的含义就是告诉Prometheus去targets采取数据来

    `job_name`: 指定监控任务的名称
    `target`: 监控的目标地址
    `lables`: 为监控目标添加额外的标签信息

2. **重启Premothues服务**

    ```shell
    sudo systemctl restart premothues
    ```

3. **查看数据是否被采集到**

    在web中打开网页<http://dmcloud27:9090/targets>，查看是否出现我们的job，成功的情形如下：
    ![prometheus_kafka](http://156.224.24.61:8090/upload/Prometheus_kafka.png)

    `status`: 服务状态，如果为**up**，表示服务可用

#### 数据导入到Grafana形成看板

1. **创建数据源**

    选择使用Prometheus的datasource
    ![datasource](http://156.224.24.61:8090/upload/datasource.png)

    指定地址
    ![address](http://156.224.24.61:8090/upload/address.png)

2. **下载看板模板json**

    在[Grafana官网](https://grafana.com/grafana/dashboards/7589-kafka-exporter-overview/)下载看板模板
    ![download](http://156.224.24.61:8090/upload/download.png)

3. **配置看板**

    创建Prometheus的数据源
    ![import](http://156.224.24.61:8090/upload/import.png)

    配置数据源
    ![import](http://156.224.24.61:8090/upload/import-azjb.png)

    导入json看板配置文件
    ![jsonFile](http://156.224.24.61:8090/upload/jsonFile.png)

    选择Prometheus的数据源
    ![choosesource](http://156.224.24.61:8090/upload/choosesource.png)

    现实看板
    ![final](http://156.224.24.61:8090/upload/final.png)
