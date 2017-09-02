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
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <!-- ここまでを記述 -->

</project>
```
