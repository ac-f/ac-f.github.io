---
title: SpringBoot串接Firebase Firestore並部署到AWS ElasticBeanstalk(2)-程式串接
categories:
  - Java
  - Spring Boot
date: 2022-11-19 18:55:30
tags:
  - Spring Boot
  - Firebase
  - AWS
  - ElasticBeanstalk
  - Gradle
  - 良葛格
cover: https://user-images.githubusercontent.com/12562305/202645794-d7922c72-a3e7-42b3-af16-1066432d709c.png
---
![image](https://user-images.githubusercontent.com/12562305/202645794-d7922c72-a3e7-42b3-af16-1066432d709c.png)

## 前言
在[上一篇文章](/2022/11/SpringBoot串接Firebase%20Firestore並部署到AWS%20ElasticBeanstalk1/)中，我們已經完成了Firebase的基礎設定，接下來就一起來讓Spring Boot串接Firebase的服務吧～

## 創建專案

首先我們打開 **IntelliJ Idea**，並且創建一個新的專案，JDK這邊我們選擇`JDK17`，設定完後按 **「Next」**

![image](https://user-images.githubusercontent.com/12562305/202840734-d614aa5c-1487-4532-9b92-13736442cdbc.png)

接著會出現選擇Dependency的畫面，這邊我只需要勾偷懶神器 `Lombok` 以及 構建API服務必備的 `Spring Web`，然後就可以按下 「Next」 來創建新專案囉。

![image](https://user-images.githubusercontent.com/12562305/202840841-839d454b-8137-4f15-8356-756c9c097dac.png)

## 設定firebase服務

### 引入依賴 (Dependency)

接下來我們只要在`/build.gradle`中引入 `Firebase` 的sdk，並reload gradle的設定就可以囉
``` groovy
...
dependencies {
  ...
    implementation 'com.google.firebase:firebase-admin:9.1.1'
}
...
```

![image](https://user-images.githubusercontent.com/12562305/202841090-84aca4b8-1eaa-4ced-8af8-57b0fcf82113.png)

### 設定Bean

首先要先把上一篇文章我們拿到的 `firebase.json` 放到 `src/main/resources`

建立 `src/main/java/com/acf/firebasedemo/config/DatabaseConfiguration.java`

```java
package com.acf.firebasedemo.config;

import com.google.auth.oauth2.GoogleCredentials;
import com.google.cloud.firestore.Firestore;
import com.google.firebase.FirebaseApp;
import com.google.firebase.FirebaseOptions;
import com.google.firebase.cloud.FirestoreClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.ResourceLoader;

import java.io.IOException;
import java.io.InputStream;

@Configuration
public class DatabaseConfiguration {
  ResourceLoader resourceLoader;

  public DatabaseConfiguration(ResourceLoader resourceLoader) {
    this.resourceLoader = resourceLoader;
  }

  @Bean
  public Firestore db() throws IOException {
    ClassPathResource resource = new ClassPathResource("firebase.json");
    var firebaseApps = FirebaseApp.getApps();
    boolean hasBeenInitialized = false;
    for (FirebaseApp app : firebaseApps) {
      if (app.getName().equals(FirebaseApp.DEFAULT_APP_NAME)) {
        hasBeenInitialized = true;
      }
    }
    if (!hasBeenInitialized) {
      InputStream inputStream = resource.getInputStream();
      FirebaseOptions options = FirebaseOptions.builder()
              .setCredentials(GoogleCredentials.fromStream(inputStream))
              .build();
      FirebaseApp.initializeApp(options);
    }

    return FirestoreClient.getFirestore();
  }
}
```

## 撰寫基本的CRUD程式

這次範例我們就來簡單做一個Item的物件，並且透過Firebase來實現新增、讀取、更新、刪除吧。

### 定義物件
我們在Spring Boot程式所在的的Package中新增一個Item的物件，並且透過Lombok提供的Annotation幫我們省去做Getter/Setter 等等的工作。
`src/main/java/com/acf/firebasedemo/pojo/Item`
```java
package com.acf.firebasedemo.pojo;

import lombok.Data;

@Data
public class Item {
  private String id;
  private String name;
}
```

### 建立 CRUD API

在新增API之前呢，我們需要先新增 `Controller` / `Service` / `DAO` 的 class，根據`SRP (Single Responsibility Principle)`原則，一個功能只專心做好一件事，跟人生一樣（？

`Controller` 負責將 `request` 的內容導向到需要處理的 `service`。

`Service` 負責業務邏輯。

`DAO` 負責跟資料庫溝通。

`src/main/java/com/acf/firebasedemo/dao/ItemDao.java`

```java
package com.acf.firebasedemo.dao;

import com.acf.firebasedemo.pojo.Item;
import com.google.cloud.firestore.*;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Map;
import java.util.concurrent.ExecutionException;
@Repository
public class ItemDao {
  private Firestore db;
  private CollectionReference collection;
  public ItemDao (Firestore firestore) {
    this.db = firestore;
    this.collection = this.db.collection("Item");
  }

  public List<Item> fetchAllItems() throws ExecutionException, InterruptedException {
    return this.collection.get().get().getDocuments().stream().map(data->data.toObject(Item.class)).toList();
  }

  public void insertItem(String id, Item item) {
    this.collection.document(id).create(item);
  }

  public void updateItem(String id, Map map) {
    this.collection.document(id).update(map);
  }
  public void deleteItem(String id) {
    this.collection.document(id).delete();
  }
}
```

`src/main/java/com/acf/firebasedemo/service/ItemService.java`

```java
package com.acf.firebasedemo.service;

import com.acf.firebasedemo.dao.ItemDao;
import com.acf.firebasedemo.pojo.Item;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ExecutionException;

@Service
@RequiredArgsConstructor
public class ItemService {
  private final ItemDao itemDao;

  public ResponseEntity createItem (Item item) {
    String id = UUID.randomUUID().toString();
    item.setId(id);
    this.itemDao.insertItem(id, item);
    return ResponseEntity.ok().body("建立成功");
  }
  public ResponseEntity getItemList () throws ExecutionException, InterruptedException {
    var list =  this.itemDao.fetchAllItems();
    return ResponseEntity.ok().body(list);
  }

  public ResponseEntity updateItem(String id, Item item) {
    Map map = Map.of("id", id, "name", item.getName());
    this.itemDao.updateItem(id, map);
    return ResponseEntity.ok().body("更新成功");
  }

  public ResponseEntity deleteItem (String id) {
    this.itemDao.deleteItem(id);
    return ResponseEntity.ok().body("刪除成功");
  }

}
```

`src/main/java/com/acf/firebasedemo/controller/ItemController.java`

```java
package com.acf.firebasedemo.controller;

import com.acf.firebasedemo.pojo.Item;
import com.acf.firebasedemo.service.ItemService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.concurrent.ExecutionException;

@RestController
@RequestMapping("item")
@RequiredArgsConstructor
public class ItemController {
  private final ItemService itemService;

  @PutMapping("")
  public ResponseEntity createItem (@RequestBody Item item) {
    return this.itemService.createItem(item);
  }
  @GetMapping("")
  public ResponseEntity getItemList () throws ExecutionException, InterruptedException {
    return this.itemService.getItemList();
  }

  @PostMapping("{id}")
  public ResponseEntity updateItem(@PathVariable("id") String id, @RequestBody Item item) {
    return this.itemService.updateItem(id, item);
  }

  @DeleteMapping("{id}")
  public ResponseEntity deleteItem (@PathVariable("id") String id) {
    return this.itemService.deleteItem(id);
  }
}
```

## 測試API

完成了API的開發後我們終於可以來測試看看我們的API啦，這邊附上我Postman設定的相關內容。

![image](https://user-images.githubusercontent.com/12562305/202846913-c80ec31c-df52-4197-bea6-54b1cb55551c.png)

![image](https://user-images.githubusercontent.com/12562305/202846928-fe4dcda9-7856-46bc-8a54-45553110eb68.png)

![image](https://user-images.githubusercontent.com/12562305/202846980-6b9f30ca-a31e-46fa-86d9-144fc0888913.png)

![image](https://user-images.githubusercontent.com/12562305/202846991-446e395e-6ea9-4340-b506-7aedaed22f5c.png)

## 結語

經過了一番波折我們終於把API服務完成啦～[下一篇](/2022/11/SpringBoot串接Firebase%20Firestore並部署到AWS%20ElasticBeanstalk3/)我們再來看如何將做好的服務部署到 `AWS Elastic Beanstalk` 上吧！

我們下次見～