---
title: 積木堆高高之使用Gradle開發多模組Spring Boot程式
categories:
  - Java
  - Spring Boot
date: 2022-11-17 14:40:30
tags:
  - Spring Boot
  - Gradle
  - IntelliJ Idea
  - 良葛格
cover: https://user-images.githubusercontent.com/12562305/202376114-2845c709-07da-49bd-9bcf-3ebed5ef7c41.png
---
![image](https://user-images.githubusercontent.com/12562305/202376114-2845c709-07da-49bd-9bcf-3ebed5ef7c41.png)

## 前言
雖然說目前在Java生態圈中，打包工具還是以Maven為大宗，但Gradle作為後起之秀其實也有他值得一用的優點，例如語法精簡，~可以逃離XML的悲慘世界~，甚至可以使用groovy的特性實現高度自動化的流程等等。

今天希望透過簡單的範例來示範如何使用Gradle來實作多模組的Spring Boot程式

如果你很懶得看文章也可以直接[看程式](https://github.com/ac-f/demo-gradle-multi-module)

## 認識多模組
當程式邏輯越寫越複雜的時候，我們會希望把不一樣的業務拆分成不一樣的模組。

![image](https://user-images.githubusercontent.com/12562305/202380150-024b8f10-7e90-49b2-852a-d425b5e9fe41.png)

例如與產品相關的程式就歸類為產品模組(Product Module)，與統計相關的程式就歸類為產品模組等等，而今天的範例，我們會透過Gradle+Spring Boot實作以下的架構。

![image](https://user-images.githubusercontent.com/12562305/202380934-254792e2-467f-491f-bdd6-881222c7b9af.png)

## 建立新專案
首先我們用IntelliJ 透過Spring Initializr新增一個Spring Boot的專案，這邊使用JDK17，並按下[Next]
![image](https://user-images.githubusercontent.com/12562305/202379185-9184289a-0cbe-4369-98c7-b3d9f64bf6b6.png)

Spring Boot的版本就用目前最新的2.7.5，然後可以直接下一步～點擊[Create]

![image](https://user-images.githubusercontent.com/12562305/202378850-8247df78-ffff-46f4-9f81-c45deaad4c43.png)

建立完之後我們看一下資料夾結構，我們可以知道Spring Initializr幫我們建立好一個有根模組(Root Module)的程式架構，而這個根模組就是`/src`。

![image](https://user-images.githubusercontent.com/12562305/202381473-add5993b-09ba-435d-b534-fcdd7a972afa.png)

接下來我們要建立兩個模組，分別是`product-module` 和 `statistic-module`。

使用 **右鍵** 點擊 **專案資料夾** (multi-module)，然後 **New** > **Module**

![image](https://user-images.githubusercontent.com/12562305/202382172-3ac073ed-c2a7-48d3-b849-3c3b43153b70.png)

接下來要**特別注意！** ， 因為我們已經建立了一個Spring Boot程式，因此我們這邊 **千萬不要** 再建立一個，而是要選擇 **New Module**， `name` 即為模組名稱，我們填入`product-module` 和 `statistic-module`，接著按下 [Create]

![image](https://user-images.githubusercontent.com/12562305/202383137-26ea0d02-9748-47c3-b40a-603668779066.png)

![image](https://user-images.githubusercontent.com/12562305/202383348-f3aa7f7c-f541-4d04-83e8-6514cd5d8aa2.png)

建立完後看一下我們的專案資料夾，你會發現已經新增了兩個模組

![image](https://user-images.githubusercontent.com/12562305/202383743-47430580-5035-4bd5-a677-39ee5329c08f.png)

並且IntelliJ 也很貼心的幫我們在 `settings.gradle` 中加入以下設定

```groovy
rootProject.name = 'multi-module'
include 'product-module'   //新加入的內容
include 'statistic-module' //新加入的內容
```
但目前其實我們只是把兩個模組加入 `gradle` 的打包清單中，還沒有引入到我們 `Spring Boot` 的程式。

因此我們需要在最外層的 `build.gradle` 中，再加入以下設定， 記得要加 `:` !

```groovy
...

dependencies {
    ...
    implementation project(':product-module')
    implementation project(':statistic-module')
}
...

```

並且在每一個module，都新增一個跟 `根模組(Root Module)` 一模一樣的package。

例如目前可以看到，我們的 `根模組(Root Module)` 底下有一個 `com.acf.multimodule` 的package

![image](https://user-images.githubusercontent.com/12562305/202385292-b61f06a9-04ff-4601-8df5-53efbfba5ae3.png)

所以我們就要在其他兩個模組內，也新增一樣的package，像這樣

![image](https://user-images.githubusercontent.com/12562305/202385926-fbf63510-8ac9-4f70-a8c4-b8ccf57cb298.png)

接著我們還需要把每個模組需要的dependency加入到各自模組的`build.gradle`中，

也就是`product-module/build.gradle` 和 `statistic-module/build.gradle`，

以下範例我們簡單做個把每個模組各加入一個 GET 的 endpoint，而在加入之前，要先在 **兩個模組中都引入**  `spring-boot-starter-web` 

或是用 [allprojects](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:allprojects(groovy.lang.Closure)) 或 [subproject](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:subprojects(groovy.lang.Closure))，但那又是更進階的玩法了。

`product-module/build.gradle` ＆ `statistic-module/build.gradle`

```groovy
...
dependencies {
  ...
  implementation 'org.springframework.boot:spring-boot-starter-web:2.7.5'
}
...
```

然後在兩個模組內各新增一個controller

`product-module/src/main/java/com/acf/multimodule/ProductController.java`

```java
package com.acf.multimodule;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("product")
public class Controller {
  @GetMapping("")
  public String index() {
    return "product";
  }
}

```


`statistic-module/src/main/java/com/acf/multimodule/StatisticController.java`

```java
package com.acf.multimodule;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("statistic")
public class Controller {
  @GetMapping("")
  public String index() {
    return "statistic";
  }
}

```

接著啟動`Spring Boot`服務，就可以看到endpoint被成功加上去啦～

![image](https://user-images.githubusercontent.com/12562305/202389604-a42a1bcf-2bfa-4c3d-b3e6-240b48a15e4d.png)
