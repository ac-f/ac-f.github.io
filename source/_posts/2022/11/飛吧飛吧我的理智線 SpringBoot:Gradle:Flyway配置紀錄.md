---
title: 飛吧飛吧我的理智線 SpringBoot/Gradle/Flyway配置紀錄
categories:
  - Java
  - Spring Boot
  - Flyway
date: 2022-11-15 00:36:30
tags:
  - SpringBoot
  - Gradle
  - Flyway
cover: https://user-images.githubusercontent.com/12562305/201612381-0156c79a-89ac-4c7e-9925-472e444fa5b1.png
---
![image](https://user-images.githubusercontent.com/12562305/201612381-0156c79a-89ac-4c7e-9925-472e444fa5b1.png)

## 前言

最近公司要導入新的東東啦，也就是今天要介紹的主題`Flyway`，

在服用之前呢，請確定一件事！非常重要，就是你的JDK版本，目前的時間是**2022年11月**，若你目前是使用`JDK12`以上的版本（有夠怪）是不支援Flyway的。

因此希望各位朋朋可以先確定好你的版號～避免像我一樣，因為奇怪的事情而沒辦法順利使用。

## 專案配置

- JDK **11**
- Spring Boot **2.7.5**
- Gradle **7**
- DBMS MariaDB **10.9.4**

## 引入專案

當然，第一步我們需要將相關套件引入到我們的打包文件(`build.gradle`)中

```groovy
buildscript {
    ...
    dependencies {
        ...
        classpath 'org.flywaydb:flyway-core'
        classpath "org.flywaydb:flyway-mysql:9.2.0"
    }
}

plugins {
    ...
    id "org.flywaydb.flyway" version "8.5.13"
}

dependencies {
  compileOnly 'org.projectlombok:lombok'
  runtimeOnly 'org.mariadb.jdbc:mariadb-java-client'
  annotationProcessor 'org.projectlombok:lombok'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

```

## 兩種配置方式

Flyway共有兩種配置方式，一種是透過在resources中的profile配置，另一種是直接在打包工具中配置。以下兩種我們都會介紹，就讓我們繼續看下去啦～

### 在 Configuration File 中配置 Flyway

`application.yml`

```yml
spring:
  flyway:
    enabled: true #是否啟用
    schema: 'DATABASE_SCHEMA' #資料庫schema
    driver: 'com.microsoft.sqlserver.jdbc.SQLServerDriver' #使用的資料庫driver
    url: 'jdbc:{DB_TYPE}://{IP}:{PORT}/{TABLE}' #資料庫路徑
    user: 'DATABASE_USERNAME' #資料庫使用者帳號
    password: 'DATABASE_PASSWORD' #資料庫使用者密碼
    locations: #指定位置下所有合規範的sql檔，都會migrate
      - classpath:db/migration 
      - filesystem:db/migration
      - s3:<bucket>(/optionalfolder/subfolder)
      - gcs:<bucket>(/optionalfolder/subfolder) #付費版才支援
    target: 'latest' #目標版號，若為latest, Flyway會將所有基準版號以上且合規範的sql檔都一並執行
    baselineVersion: '0.0.1.1' #基準版號，
    baselineOnMigrate: true #是否也migrate基準版號的sql檔
    validateOnMigrate: true #migrate前是否先驗證
    outOfOrder: true #版號是否不須照順序
    placeholderReplacement: false #是否將資料庫中placeholder的地方替換
    table: flyway_schema_history #用於紀錄migration的狀態

```
### 在 build.gradle 中配置 Flyway

這也是筆者比較建議的方式，原因是若寫在`Configuration File`中，每次運行程式的時候都會去執行一次，

以下將會使用建立一個獨立task的方式來配置Flyway

`build.gradle`

```groovy
task migrateDB(type: org.flywaydb.gradle.task.FlywayMigrateTask) {
    driver = 'com.microsoft.sqlserver.jdbc.SQLServerDriver'
    url = 'jdbc:{DB_TYPE}://{IP}:{PORT}/{TABLE}'
    user = 'DATABASE_USERNAME'
    password = 'DATABASE_PASSWORD'
    locations = ['classpath:db/migration']
    target = 'latest'
    baselineVersion = '0.0.1.1'
    baselineOnMigrate = true
    validateOnMigrate = true
    outOfOrder = true
    placeholderReplacement = false
    table = 'flyway_schema_history'
}

```

若想了解更多配置，也可以參考[官方文件](https://flywaydb.org/documentation/configuration/parameters/)

## 第一個migration
[官方文件](https://flywaydb.org/documentation/concepts/migrations#naming-1)提到的命名方式，簡單來說就是以下格式

`[Prefix]`  `[Version]`  `[Sperator]`  `[Description]`

- prefix: 該migration的類型，分為三種 `V`標記版本用 `U`重做用 以及`R`重複使用 （一般情境只會用到V)
- version: 版本號碼，為數字與小數點的組合。
- sperator: 分割版本與描述，為**兩個**底線_ (這真的要特別注意!)
- description: 該migration的描述文字

### 範例

`db/migrations/V0.0.0.1__demo.sql`

```sql
CREATE TABLE demo_migration (
  value VARCHAR(25) NOT NULL, PRIMARY KEY(value)
);


INSERT INTO demo_migration (value) VALUES ('AC demo');
```