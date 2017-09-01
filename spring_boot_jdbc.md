# spring boot jdbc

## pom.xmlの記述
spring-bootとjdbcの設定を入れましょう。

今回ハンズオンでは、RDBはインストール不要なインメモリで動作するH2を使用しますので、その設定を入れましょう。

pom.xmlを開いてjunitの設定を削除して、\<!-- ここから記述 --\>から\<!-- ここまで記述 --\>までの行を記述しましょう。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>jp.javado.springboot</groupId>
    <artifactId>springboot-jdbc</artifactId>
    <packaging>jar</packaging>
    <version>1.0</version>
    <name>springboot-jdbc</name>
    <url>http://maven.apache.org</url>

    <!-- ここから記述 -->
    <parent>
        <!-- spring-bootの設定 -->
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.6.RELEASE</version>
    </parent>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <!-- springbootのjdbcの設定 -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <!-- H2(RDB)の設定 -->
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <!-- mavenでspringbootをサポートする設定 -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    <!-- ここまでを記述 -->

</project>
```

## mvnでコンパイル
pom.xmlに書いた記述が合っているか確認するためにmvnでコンパイルしましょう。

```
$ mvn compile
```

下記のようなログが出力されるので、[INFO] BUILD SUCCESS、が確認できたらOKです。
spring-bootやh2のjarファイルがダウンロードされることがわかります。
（ここに記載があるのはspring-bootはすでにダウンロード済みだったため出力されていません）

```
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building springboot-jdbc 1.0
[INFO] ------------------------------------------------------------------------
Downloading: https://repo.maven.apache.org/maven2/com/h2database/h2/1.4.196/h2-1.4.196.pom
Downloaded: https://repo.maven.apache.org/maven2/com/h2database/h2/1.4.196/h2-1.4.196.pom (960 B at 0.3 KB/sec)
Downloading: https://repo.maven.apache.org/maven2/com/h2database/h2/1.4.196/h2-1.4.196.jar
Downloaded: https://repo.maven.apache.org/maven2/com/h2database/h2/1.4.196/h2-1.4.196.jar (1780 KB at 211.2 KB/sec)
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ springboot-jdbc ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /Users/haruki/javado/maven/20170903/springboot/springboot-jdbc/src/main/resources
[INFO] skip non existing resourceDirectory /Users/haruki/javado/maven/20170903/springboot/springboot-jdbc/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ springboot-jdbc ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /Users/haruki/javado/maven/20170903/springboot/springboot-jdbc/target/classes
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 14.334 s
[INFO] Finished at: 2017-09-01T16:31:48+09:00
[INFO] Final Memory: 22M/188M
[INFO] ------------------------------------------------------------------------
```

## DBのテーブルと対応するクラスを作成
DBに作成するテーブルを以下としましょう。

| id | first_name | last_name |
|:---|-----------:|:---------:|
| 1  | John       | Woo       |
| 2  | John       | Titor     |
| 3  | Josh       | Long      |


このテーブルと対応するクラスを作成します。

作る場所はsrc/main/java/jp/javado/springbootで、クラス名はCustomer.javaを作ります。

```
package jp.javado.springboot;

public class Customer {
    private long id;
    private String firstName, lastName;
    
    public Customer() {
    }

    public Customer(long id, String firstName, String lastName) {
        this.id = id;
        this.firstName = firstName;
        this.lastName = lastName;
    }

    @Override
    public String toString() {
        return String.format(
                "Customer[id=%d, firstName='%s', lastName='%s']",
                id, firstName, lastName);
    }

    public long getId() {
        return id;
    }
    
    public void setId(long id) {
        this.id = id;
    }
    
    public String getFirstName() {
        return this.firstName;
    }
    
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
    
    public String getLastName() {
        return this.lastName;
    }
    
    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}
```

## SpringBootのアプリケーションクラスを作成
src/main/java/jp/javado/springboot/App.javaのクラスが雛形で用意されているので、それを書き換えて作成しましょう。

このクラスがSpringBootのアプリケーションとして最初に動くクラスであることを示すために@SpringBootApplicationを使います。

このアプリケーションはコマンドラインで動作するため、CommandLineRunnerインターフェースを実装します。

Springが提供するRDBにアクセスするためのクラスが、JdbcTemplateクラスです。
JdbcTemplateクラスを使えるようにするために、フィールド変数として書き、中身はRDBに接続するための所々の設定をSpringが自動的に行ったものを、@Autowiredをつけることで扱うことができるようになります。
@AutowiredはSpringコンテナが管理するクラスの実装をDIするためのアノテーションです。

```
package jp.javado.springboot;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.jdbc.core.JdbcTemplate;

@SpringBootApplication
public class App implements CommandLineRunner {

    private static final Logger log = LoggerFactory.getLogger(App.class);

    public static void main(String args[]) {
        SpringApplication.run(App.class, args);
    }

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Override
    public void run(String... strings) throws Exception {

    }
}
```

## アプリケーション実行時に、RDBにテーブルを作成
今回は、サンプルに従ってアプリケーション実行時にテーブルを作成しましょう。
開発時はこういったケースはあまりないとは思いますが、今回は練習用です。

run()メソッドの中に下記を書いて、アプリケーション実行時にテーブルを作成します。なお、最初の「DROP TABLE customers IF EXSITS」は、今回は何度もアプリケーションを動かすため、customersテーブルが存在していたら消して作り直すためにあります。

サンプルではテーブルの設計は以下です

- idカラムは、SERIAL型（<- データ挿入時に番号が自動でふられる）
- first_nameカラムは、VARCHAR(255)型（文字列を格納する）
- last_nameカラムは、VARCHAR(255)型（文字列を格納する）

```
        log.info("Creating tables");

        jdbcTemplate.execute("DROP TABLE customers IF EXISTS");
        jdbcTemplate.execute("CREATE TABLE customers(" +
                "id SERIAL, first_name VARCHAR(255), last_name VARCHAR(255))");

```

これで、実行時にH2（RDB）にさきほどのテーブルができます。
まだ中身はないです。

| id | first_name | last_name |
|:---|-----------:|:---------:|
|    |            |           |
|    |            |           |
|    |            |           |


## テーブルにデータを挿入

できあがったテーブルにデータを挿入して以下となるようにするために、プログラム中でINSERT文を実行できるようにします。

| id | first_name | last_name |
|:---|-----------:|:---------:|
| 1  | John       | Woo       |
| 2  | John       | Titor     |
| 3  | Josh       | Long      |

最初に挿入するためのデータを用意し、そのデータをDBに一件ずつ挿入するためにjdbcTemplate.update()メソッドを書きます。

```
        List<Object[]> insertDataList = new ArrayList<>();
        insertDataList.add(new Object[]{ "John", "Woo"});
        insertDataList.add(new Object[]{ "John", "Titor"});
        insertDataList.add(new Object[]{ "Josh", "Long"});

        String insertSql = "INSERT INTO customers(first_name, last_name) VALUES (?,?)";

        for (Object[] insertData : insertDataList) {
            jdbcTemplate.update(insertSql, insertData);
        }
```


## 格納したデータを取り出して表示

RDBのテーブルのデータを取得するSELECT文を実行するためのメソッドとして, jdbcTemplate.query()メソッドが用意されています。

query( "SQL文" , "SQLの条件に設定したい値", "SQL実行結果を格納する")
となっています。

"SQL実行結果を格納する"方法としてサンプルでは、
ラムダ式を使ってDBから値をrs.getString()で値を取得して、用意していたCustomerクラスにセットするができます。

DBから値をSELECTして実行できるプログラムを書きましょう。

```
        log.info("Querying for customer records where first_name = 'Josh':");
        
        String selectSql = "SELECT id, first_name, last_name FROM customers WHERE first_name = ?";
        List<Customer> customerList = jdbcTemplate.query(selectSql, new Object[] { "Josh" },
                (rs, rowNum) -> new Customer(rs.getLong("id"), rs.getString("first_name"), rs.getString("last_name")));
        
        // SQL実行して取得した値を表示
        customerList.stream().forEach(customer -> log.info(customer.toString()));
```

## SpringBootアプリケーションの実行

以下のコマンドを実行すると、

- DBにテーブルを作成
- テーブルに値を挿入
- テーブルから値を取得
- データを表示

ができます。

```
$ mvn spring-boot:run
```

実行すると、以下のログが表示されます。

```
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building springboot-jdbc 1.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] >>> spring-boot-maven-plugin:1.5.6.RELEASE:run (default-cli) > test-compile @ springboot-jdbc >>>
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ springboot-jdbc ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /Users/haruki/javado/maven/20170903/springboot/springboot-jdbc/src/main/resources
[INFO] skip non existing resourceDirectory /Users/haruki/javado/maven/20170903/springboot/springboot-jdbc/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ springboot-jdbc ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to /Users/haruki/javado/maven/20170903/springboot/springboot-jdbc/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ springboot-jdbc ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /Users/haruki/javado/maven/20170903/springboot/springboot-jdbc/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ springboot-jdbc ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] <<< spring-boot-maven-plugin:1.5.6.RELEASE:run (default-cli) < test-compile @ springboot-jdbc <<<
[INFO] 
[INFO] --- spring-boot-maven-plugin:1.5.6.RELEASE:run (default-cli) @ springboot-jdbc ---

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.6.RELEASE)

2017-09-01 19:27:44.409  INFO 19776 --- [           main] jp.javado.springboot.App                 : Starting App on dhcp4.kk.pilot.chitose.ac.jp with PID 19776 (/Users/haruki/javado/maven/20170903/springboot/springboot-jdbc/target/classes started by haruki in /Users/haruki/javado/maven/20170903/springboot/springboot-jdbc)
2017-09-01 19:27:44.413  INFO 19776 --- [           main] jp.javado.springboot.App                 : No active profile set, falling back to default profiles: default
2017-09-01 19:27:44.509  INFO 19776 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@55247ea9: startup date [Fri Sep 01 19:27:44 JST 2017]; root of context hierarchy
2017-09-01 19:27:45.615  INFO 19776 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2017-09-01 19:27:45.629  INFO 19776 --- [           main] jp.javado.springboot.App                 : Creating tables
2017-09-01 19:27:46.033  INFO 19776 --- [           main] jp.javado.springboot.App                 : Querying for customer records where first_name = 'Josh':
2017-09-01 19:27:46.051  INFO 19776 --- [           main] jp.javado.springboot.App                 : Customer[id=3, firstName='Josh', lastName='Long']
2017-09-01 19:27:46.053  INFO 19776 --- [           main] jp.javado.springboot.App                 : Started App in 2.1 seconds (JVM running for 6.727)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 5.461 s
[INFO] Finished at: 2017-09-01T19:27:46+09:00
[INFO] Final Memory: 30M/394M
[INFO] ------------------------------------------------------------------------
2017-09-01 19:27:46.267  INFO 19776 --- [       Thread-2] s.c.a.AnnotationConfigApplicationContext : Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@55247ea9: startup date [Fri Sep 01 19:27:44 JST 2017]; root of context hierarchy
2017-09-01 19:27:46.268  INFO 19776 --- [       Thread-2] o.s.j.e.a.AnnotationMBeanExporter        : Unregistering JMX-exposed beans on shutdown

```

この中で下記の行が表示されたデータとなります。

```
2017-09-01 19:27:46.051  INFO 19776 --- [           main] jp.javado.springboot.App                 : Customer[id=3, firstName='Josh', lastName='Long']

```

## 自動でクラスにDBの値を設定
さきほどはラムダ式を使って値を取得してCustomerクラスに設定していましたが、もっと便利に書く方法があります。

BeanPropertyRowMapperクラスを使うと、指定したクラス（ここではCustomerクラス）に取得した値を自動的に設定してくれます。

BeanPropertyRowMapper<>("テーブルと対応したクラス")に書き換えてみましょう。

まず、import文を記載する箇所（プログラムのかなり上部）に以下を付け加えます。

```
import org.springframework.jdbc.core.BeanPropertyRowMapper;
```

jdbcTemplate.query()の箇所を以下に書き換えます。

```
        List<Customer> customerList = jdbcTemplate.query(selectSql, new Object[] { "Josh" }, new BeanPropertyRowMapper<>(Customer.class) );

```

コマンドで実行してみると、

```
$ mvn spring-boot:run
```

出力されたログに同じ値が出ていることが確認できます。

```
2017-09-01 19:41:55.011  INFO 19954 --- [           main] jp.javado.springboot.App                 : Customer[id=3, firstName='Josh', lastName='Long']
```

