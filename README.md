# 20170903_seminar_springboot
for The Spring Boot Session using JdbcTemplate


## 環境セットアップ

### Mavenプロジェクト作成

#### 以下のコマンドを実行

うまく行かない場合は、\改行 を削除して、1行でやってみてください。

```
mvn -B archetype:generate \
-DgroupId=jp.javado.springboot \
-DartifactId=springboot-jdbc \
-Dversion=1.0 \
-DarchetypeArtifactId=maven-archetype-quickstart
```

#### 以下のフォルダ・ファイルができていることを確認
 - springboot-jdbc
 - springboot-jdbc/src
 - springboot-jdbc/pom.xml
 
ファイル・フォルダ構成は以下になっています。

```
springboot-jdbc/ 
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── jp
    │           └── javado
    │               └── springboot
    │                   └── App.java
    └── test
        └── java
            └── jp
                └── javado
                    └── springboot
                        └── AppTest.java
```


### 作成したプロジェクトを統合開発環境に取り込んでみよう

#### IntelliJ

- Import Project
 - springboot-jdbc/pom.xml を選択
 - 上から3番目のImport Maven projects automatically にチェック
 - あとは指示に従ってNextしていく。（JDKの設定ができていることだけ注意）
 
#### NetBeans

- ファイル > プロジェクトを開く
  - springboot-jdbc を選択

#### Eclipse

- File > Import...（日本語化していれば ファイル > インポート）
  - Maven > Existing Maven Project  
（日本語化していれば、 Maven > 既存のMavenプロジェクト）
  - Browse > springboot-jdbc を選択
