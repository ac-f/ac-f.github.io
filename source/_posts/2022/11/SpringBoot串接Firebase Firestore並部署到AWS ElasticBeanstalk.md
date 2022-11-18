---
title: SpringBoot串接Firebase Firestore並部署到AWS ElasticBeanstalk(1)-啟用Firebase服務
categories:
  - Java
  - Spring Boot
date: 2022-11-18 17:00:30
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

之前在幫以前大學的老師做一個線上測驗的Side Project，

因為小弟實在是太窮了，老師也想節省經費，因此就在想怎麼用最省錢的方式來構建一個系統。

過往是從前端慢慢摸到後端，因此對Firebase不算陌生，因此就把歪腦筋動到了Firebase，

所幸Firebase 也有提供Java的SDK，但是網路上的中文教學範例不是太多，因此今天就想說可以來貢獻下

ps. 這個圖是我自己拉的，總覺得把Spring Boot 的Logo加個帽子(Firebase的Logo)好可愛XD

## 必要條件

首先你必須滿足以下條件才能跟著教學一步一步完成哦！

1. 一個[Google](https://www.google.com.tw/)帳號
2. 一個[AWS](https://aws.amazon.com/tw/)帳號
3. ~一台可以摳頂並能連上網路的電腦~

## 使用Firebase服務

Firebase這個服務在2014年被Google收購，因此如果您已經有Google帳號就可以直接開始囉！

首先我們先進入[Firebase控制台](https://console.firebase.google.com/)頁面，

若您之前沒有使用過，映入眼簾的就是這個非常乾淨好看的頁面，這時候我們點一下「建立專案」

![image](https://user-images.githubusercontent.com/12562305/202651854-45006d52-c02f-4a84-9693-b032a6a47b98.png)

這邊就隨便取個喜歡的名字吧，Google Analytic 不在本次教學的範疇內，所以這邊就先不打開囉～（你想開也是可以）
![image](https://user-images.githubusercontent.com/12562305/202650215-56d48686-e2ad-431b-b7e5-534c7906d47d.png)

接著就會跳轉到構建專案的畫面，稍微等他跑一下，點擊「繼續」
![image](https://user-images.githubusercontent.com/12562305/202650626-5315ad7c-1299-4d5c-8e7d-6aac07443e04.png)

噠啷～ 不得不說他的介面真的做的非常好看
![image](https://user-images.githubusercontent.com/12562305/202650903-80bea411-738a-451a-af1a-9f2a98fe68b2.png)

### Cloud Firestore & Realtime Database的差異

在Firebase中有提供兩個服務 `Cloud Firestore` 與 `Realtime Database`，這兩個都是NoSQL的Database，並且都有提供**實時資料更新**的功能（只有前端用得到XD）

他們的差異主要在於Realtime Database更快，但卻不支援**複雜的查詢** ~(其實簡單的就不太好查了)~，因此如果你要做的服務需要有稍微複雜的查詢，就請選擇`Cloud Firestore`

本次範例會使用`Cloud Firestore`來做示範。

更多內容可以參考[官方文件](https://firebase.google.com/docs/database/rtdb-vs-firestore)

### 新增Android應用程式

接著我們需要新增Android應用程式

![image](https://user-images.githubusercontent.com/12562305/202658071-24428168-8ade-449a-a708-fb51167dca48.png)

然後輸入應用程式package的名稱，這邊我輸入我常用的`com.acf.demo`，點擊「註冊應用程式」，接著點擊「下載google-service.json」，接著就一直按「繼續」就行了。

這個json檔非常重要，務必一定要下載下來，我們要透過這個json file才可以存取資料庫的相關資源。

![image](https://user-images.githubusercontent.com/12562305/202659161-b5910968-0f34-4959-93a5-7ed4e7a33540.png)

接下來我們要啟用Firebase的`Cloud Firestore`服務，展開控制台左邊導覽列的 「建構」 並點擊 「Firestore Database」

![image](https://user-images.githubusercontent.com/12562305/202661200-bac34b90-d3bb-48d1-bf04-e10e0369f736.png)

點擊「建立資料庫」

![image](https://user-images.githubusercontent.com/12562305/202660710-41aedfec-6642-4400-88de-bb1d7685f717.png)

點擊 「以測試模式啟動」 並按 「繼續」。

![image](https://user-images.githubusercontent.com/12562305/202661438-09ce6653-1569-4a53-988b-84b77a18c4a5.png)

接著選擇地區的部分可以選擇離自己近的區域，這邊筆者選擇目前所在區域「臺灣」，並按下 「啟用」。

![image](https://user-images.githubusercontent.com/12562305/202661777-21ce169e-729f-4226-a185-3281e640282b.png)

## 結語

這樣就完成Firebase的設定啦～

我們下一篇文章會教大家如何引入Firebase服務到你的Spring Boot程式當中，並且完成基礎的CRUD功能。

敬請期待～

