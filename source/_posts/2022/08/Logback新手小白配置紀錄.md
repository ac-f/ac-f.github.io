---
title: Logback新手小白配置紀錄
categories:
  - Java
  - Spring Boot
  - LogBack
date: 2022-08-09 19:31:30
tags:
  - Java
  - Logback
cover: https://user-images.githubusercontent.com/12562305/183610738-493008ff-414b-44df-9016-f6c71181275c.png
---
![image](https://user-images.githubusercontent.com/12562305/183610738-493008ff-414b-44df-9016-f6c71181275c.png)

## 前言

每次在開新專案的時候總會去翻一下過去的專案是怎麼寫Logger配置文件的，今天就把相關的內容整理起來方便以後回憶。

## Logging Level

基本上Log有分幾種等級，由輕微到嚴重可依序分為

- Trace
- Debug
- Info
- Warn
- Error

如果沒有特別設置，Spring Boot 預設只處理 **Info** 以上的Log(避免Console或log檔)炸開。

## Logback

Logback是Spring Boot 預設的Logging工具，有兩種方式可以啟用Logback

1. 直接使用`application.properties` 或 `application.yml`來配置
  => 最簡單，但彈性低
2. 使用xml檔來配置
  => 比較複雜，但彈性高

## 使用application.yml來進行配置

`src/main/resources/application.yml`

```yml
logging:
  level:
    com.acf.logbacktest.Log1Service: trace
    com.acf.logbacktest.Log2Service: info
```

如果想看看 `application.yml` 有哪些更多的設定，可以參考[Common Application Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties.core)

## 使用xml檔來配置

`src/main/resources/logback.yml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- 這邊可以設定相對應的變數，讓整個xml的結構更簡潔 -->
    <property name="ProjectName" value="Logback Test" />
    <property name="LOGS" value="./logs" />
    <property name="ConsolePattern"
      value="%cyan(%d{ISO8601}) %highlight(%-5level) [%blue(%t)] %yellow(%C{1.}): %msg%n%throwable" />

    <appender name="Console" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>${ConsolePattern}</Pattern>
        </layout>
    </appender>

    <appender name="RollingFile" class="ch.qos.logback.core.rolling.RollingFileAppender">

        <file>${LOGS}/${ProjectName}.log</file>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <Pattern>%d %p %C{1.} [%t] %m%n</Pattern>
        </encoder>

        <rollingPolicy
                class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 當檔案大於10MB時就會轉存到以下這個檔案 -->
            <fileNamePattern>${LOGS}/archived/logger-%d{yyyy-MM-dd}.%i.log
            </fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender>

    <!-- Log紀錄大於 INFO level 的log -->
    <root level="info">
        <appender-ref ref="RollingFile" />
        <appender-ref ref="Console" />
    </root>

    <!-- 在"com.acf.logbacktest.Log1Service*"這個位置的程式會記錄大於 TRACE level 以上的Log(即所有)-->
    <logger name="com.acf.logbacktest.Log1Service" level="trace" additivity="false">
        <appender-ref ref="RollingFile" />
        <appender-ref ref="Console" />
    </logger>

</configuration>
```

## 建立兩個Service來測試Log

`com/acf/logbacktest/Log1Service.java`

```java
package com.acf.logbacktest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

@Component
public class Log1Service {
        private final static Logger logger = LoggerFactory.getLogger(Log1Service.class);

    @Bean
    public void log() {

        logger.trace("Trace log");
        logger.debug("Debug log");
        logger.info("Info Log");
        logger.warn("Warn Log");
        logger.error("Error Log");
    }
}

```

`com/acf/logbacktest/Log2Service.java`

```java
package com.acf.logbacktest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

@Component
public class Log2Service {
        private final static Logger logger = LoggerFactory.getLogger(Log2Service.class);

    @Bean
    public void log() {

        logger.trace("Trace log");
        logger.debug("Debug log");
        logger.info("Info Log");
        logger.warn("Warn Log");
        logger.error("Error Log");
    }
}

```

接著就可以看到log被記錄下來囉!
![image](https://user-images.githubusercontent.com/12562305/183618425-eeeeee5a-5a25-4d75-9815-fcb1b929a2b9.png)
![image](https://user-images.githubusercontent.com/12562305/183618730-f04e4b9f-cc98-423f-ba50-b5d2f4509423.png)

## 參考資料

- [Logging in Spring Boot](https://www.baeldung.com/spring-boot-logging)
- [Common Application Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties.core)
- [Logback](https://logback.qos.ch/)
