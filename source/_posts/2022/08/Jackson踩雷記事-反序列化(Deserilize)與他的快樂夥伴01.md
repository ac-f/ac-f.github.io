---
title: Jackson踩雷記事-反序列化(Deserilize)與他的快樂夥伴01
categories:
  - Java
  - Jackson
date: 2022-08-12 14:31:30
tags:
  - Java
  - Jackson
  - HttpClient
  - HttpRequest
  - Spring Boot
  - 踩雷
cover: https://user-images.githubusercontent.com/12562305/184280610-8ee68f69-e835-438d-a897-9b3c7eadc688.png
---
![image](https://user-images.githubusercontent.com/12562305/184280610-8ee68f69-e835-438d-a897-9b3c7eadc688.png)

## 前言

最近遇到了一個我職業生涯中很重大的瓶頸(並沒有)，需求是需要去call特定路徑的api，並且解析Response的JSON，最後把相對應的內容塞進資料庫。

## 處理連線

首要當然要先處理連線，`HttpClient` 是在 **Java9** 新增的API，相較於以往的處理方式除了提供了對HTTP/2的支援，
甚至效能更甚，因此我們透過它來發送HttpRequest。

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.Duration;


public String fetchData () {
    String url = "https://test.com";
    HttpClient client = HttpClient
        .newBuilder()
        .connectTimeout(Duration.ofMinutes(3))
        .followRedirects(HttpClient.Redirect.NORMAL)
        .build();
    HttpRequest request = HttpRequest
        .newBuilder()
        .uri(URI.create(url))
        .version(HttpClient.Version.HTTP_2)
        .GET()
        .build();

    try {
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        return response.body();
    } catch (Exception e) {
        e.getStackTrace();
    }

    return "";
}
```

## 解析JSON

一開始拿到的資料是長這個樣子的

```json
{
  "count": 1,
  "data": [
    {
      "id": 1,
      "cmsdbUrl": "https://test.com",
      "imageAuthority": {
        "type": 12
      },
      "name": "AC-f",
      "gender": 1,
      "alias": "AC",
      "citizenship": " ",
      "bornStr": "1877",
      "bornPlace": "臺北市信義區",
      "bornCity": "臺北市",
      "bornAddress": " ",
      "bornLng": 120.2079953,
      "bornLat": 23.008515,
      "deathStr": "1934",
      "deathPlace": "",
      "parent": "父：老AC",
      "job": "工程師",
      "organization": "Github",
      "creator": "AC",
      "categories": [
        {
          "category1": "AC",
          "category2": "ABC",
          "category3": "ABCD",
          "category4": "ABCDE"
        }
      ],
      "expertise": [
        "Java"
      ],
      "content": "<p>一些內容</p>",
      "timelines": [
        "中華民國"
      ],
      "keywords": [
        "AC",
        "Java"
      ],
      "language": "中文",
      "sameAs": "",
      "dataSource": "本人提供",
      "openType": false,
      "openStatus": true,
      "openStart": "2022-11-11T11:11:11.233+08:00",
      "apiInterface": false,
      "audit": 1,
      "createDate": "2022-11-11T11:11:11.233+08:00",
      "modifyDate": "2022-11-11T11:11:11.233+08:00",
      "auditDate": "2022-11-11T11:11:11.233+08:00",
      "createOrgName": "管理單位",
      "check": 0
    }
  ]
}
```

於是乎我理所當然的去google一些內容，有許多網友推薦使用Jackson，我心想Jackson是SpringBoot Web預設使用的套件，應該穩穩的，於是就開始引入到我的Project內。

```gradle
implementation 'com.fasterxml.jackson.core:jackson-databind:2.0.1'
```

引入完之後想當然爾就是要劈哩啪啦來一段code才能對得起老闆的薪水，於是我參考了網路上的範例寫了一個測試的內容

```java
public void runJackson () {
  String data ="{\n" +
        "        \"count\": 12\n" +
        "    }";
    System.out.println(data);
    ObjectMapper mapper = new ObjectMapper();
    try {
        Test test = mapper.readValue(data, Test.class);
        System.out.println(test.toString());
    } catch (Exception exception) {
        exception.printStackTrace();
    }
}

class Test {
    int count;
    public String toString () {
        return "count: " + count;
    }
}

```

嘿嘿嘿，這時候來猜一猜我兩段print到底會輸出什麼東西吧!
答案是...

**啥都沒有印!**

但是它很貼心的(並沒有)給我們一段錯誤訊息
`Error "java.lang.NoSuchFieldError: WRITE_DURATIONS_AS_TIMESTAMPS"`

後來稍微爬了一下文，才發現這個是因為`Jackson`的相依套件`joda`已經無法相容新版本的專案，此時我們需要將`Jackson`升級到比較新的版本。

```gradle
implementation 'com.fasterxml.jackson.core:jackson-databind:2.13.3'
```

這時候我們再來一次不要命的compile，看看是不是會如我們所願印出東西呢!
以下是輸出內容...

```json
// 第一個輸出
{
    "count":12
}

// 第二個輸出
count: 0
```

![image](https://user-images.githubusercontent.com/12562305/184292017-ee23d492-d6ee-47cf-ac80-1594681948b5.png)
WTF???
第二個輸出不應該是`count: 0`嗎?

於是我就開始瘋狂的問Java社群的朋友問題啦XD，後來才被提醒如果要對物件賦值，要有`getter` & `setter`，接著我把程式改成了這樣

```java
public void runJackson () {
  String data ="{\n" +
        "        \"count\": 12\n" +
        "    }";
    System.out.println(data);
    ObjectMapper mapper = new ObjectMapper();
    try {
        Test test = mapper.readValue(data, Test.class);
        System.out.println(test.toString());
    } catch (Exception exception) {
        exception.printStackTrace();
    }
}

class Test {
    int count;

    public String getCount() {
        return category1;
    }

    public void setCount(String count) {
        this.count = count;
    }

    public String toString () {
        return "count: " + count;
    }
}

```

搭拉，如此一來就可以正常輸出了

```json
// 第一個輸出
{
    "count":12
}

// 第二個輸出
count: 12
```

## 參考資源

- [java 11 httpClient 簡介與爬蟲實作](https://www.tpisoftware.com/tpu/articleDetails/2341)
- [Error "java.lang.NoSuchFieldError: WRITE_DURATIONS_AS_TIMESTAMPS"](https://stackoverflow.com/questions/34329418/error-java-lang-nosuchfielderror-write-durations-as-timestamps)
