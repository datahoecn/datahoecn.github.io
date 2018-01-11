## TiSpark 开发示例

#### 构建编译环境

./bin/build-client.sh

在这里下载或编译 tispark ，并确保它在本地 maven 仓库中可见（这意味着您可能需要 mvn clean install 在您的 tispark 目录中执行以安装 tispark 到本地 .m2 仓库）。

#### java 测试
编写自己的应用程序代码。在这个例子中，我们创建一个 com.pingcap.spark.App 像这样的 Java 文件：

```
package com.pingcap.spark;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.TiContext;

public class App {
  public static void main(String[] args) {
    SparkSession spark = SparkSession
            .builder()
            .appName("TiSpark Application")
            .getOrCreate();

    TiContext ti = new TiContext(spark);
    ti.tidbMapDatabase("tpch", false);
    Dataset dataset = spark.sql("select * from customer");
    dataset.show(); // Displays the content of the DataFrame to stdout
  }
}
```
我们使用它从表中取出所有行 tpch.customer 并将其打印出来。

运行 mvn clean package 在您的控制台来建立 your-application.jar

cd 到你的 spark 配置目录，并在你的 spark-default.conf 根据您的实际部署环境更改您的 pd 地址

spark.tispark.pd.addresses 127.0.0.1:2379

./bin/spark-submit --class <main class> --jars /where-ever-it-is/tispark-0.1.0-SNAPSHOT-jar-with-dependencies.jar your-application.jar

/home/tidb/deploy/spark/bin/spark-submit --master spark://10.122.48.109:7077 --num-executors 12 --executor-memory 6G  --executor-cores 2  --total-executor-cores 24 --conf spark.tispark.pd.addresses=10.122.33.238:2379,10.122.33.252:2379,10.122.33.243:2379 --class App /root/yukuo/target/yukuo.spark-1.0-SNAPSHOT-jar-with-dependencies.jar

在这种情况下，我们运行

./bin/spark-submit --class com.pingcap.spark.App --jars /home/novemser/Documents/Code/PingCAP/tispark/target/tispark-0.1.0-SNAPSHOT-jar-with-dependencies.jar /home/novemser/Documents/Code/Java/tisparksample/target/tispark-sample-0.1.0-SNAPSHOT.jar
--class指定应用程序的入口点
--jars指定TiSpark库

#### Scala 测试
#### python 测试

#### R 语言测试

