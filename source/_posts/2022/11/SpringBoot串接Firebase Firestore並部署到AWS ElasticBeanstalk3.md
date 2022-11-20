---
title: SpringBoot串接Firebase Firestore並部署到AWS ElasticBeanstalk(3)-部署
categories:
  - Java
  - Spring Boot
date: 2022-11-20 09:55:30
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
在[上一篇文章](/2022/11/SpringBoot串接Firebase%20Firestore並部署到AWS%20ElasticBeanstalk2/)中，我們已經完成了一個基礎的CRUD程式，成為了一個無情的CRUD工具。

這篇文章假定大家已經有AWS的帳號並且已經綁定信用卡，如果沒有的話可以去參考別的文章，網路上有蠻多大神提供相關資源的。

接下來我們就一起來把程式部署到AWS的ElasticBeanstalk上吧。

## 打包程式

首先我們要先將寫好的程式打包，點擊頁面側邊的 「Gradle」 頁籤，展開 「Build」 ，接著把bootjar ~按到爛掉~

![image](https://user-images.githubusercontent.com/12562305/202877940-6a19befe-ff42-4386-95c3-fe7272642856.png)

接著你就會在 `/build/libs` 中看到打包好的jar檔，等一下我們就是要把它上傳到 ElasticBeanstalk。

![CleanShot 2022-11-20 at 09 13 23@2x](https://user-images.githubusercontent.com/12562305/202878045-b3540e66-37f5-4d13-a39c-d074adbbe53a.png)

## 部署程式

打開AWS主控台介面，在搜尋欄上搜尋 ElasticBeanstalk，並點擊第一個。

![image](https://user-images.githubusercontent.com/12562305/202877835-514a9fdd-4397-4b17-b1cd-d962ba65de5d.png)

點擊 「建立新的環境」，選擇 「Web 伺服器環境」 ，再點 「從顯示的新下拉式清單選取」。
![image](https://user-images.githubusercontent.com/12562305/202878273-0bfda118-2ebb-4676-a183-99f4baec5dce.png)

接著填入 「應用程式名稱」
![image](https://user-images.githubusercontent.com/12562305/202878401-13d9028d-884d-49df-a705-9acaaedf2b29.png)

平台選擇「Java」，分支選擇JDK版本 「17」，下面把 「上傳您的程式碼」 選起來。
![CleanShot 2022-11-20 at 09 35 25@2x](https://user-images.githubusercontent.com/12562305/202878605-2ae0988e-eac4-41d8-8da8-0c5968436bb2.png)

選擇 「本機檔案」 ，接著點擊 「選擇檔案」 ，並選擇剛剛打包的jar檔，接著點擊 「建立環境」。
![image](https://user-images.githubusercontent.com/12562305/202878824-77df37ee-4c5a-4031-90a1-d08e8a663604.png)

接著就讓他跑一下，去喝杯咖啡、吃個早餐
![image](https://user-images.githubusercontent.com/12562305/202878932-75173919-2cd3-4dd6-9118-3993a1750aac.png)

Holy ...
![image](https://user-images.githubusercontent.com/12562305/202879957-e4a05579-aa56-407f-b298-0ef23bd16308.png)

對不起這個Bug我解不出來，沒啦開玩笑，是因爲ElasticBeanstalk自帶的nginx會主動去監聽`5000port`的內容。

因此我們只要在`src/main/resources/application.properties`(or yaml)中改port的設定值就可以囉！
```properties
server.port=5000
```

然後再重Build一次程式並上傳就可以囉！

API 的Base Url就是下圖第二點的網址哦～

雖然他的運作狀態看起來紅紅的很嚇人，但實際上去打每一個Endpoint都是通的哦！

![image](https://user-images.githubusercontent.com/12562305/202880582-d5c7d4e0-d3c0-4a56-949e-8e9cade8e438.png)

## 結語

這個系列的文章就到這邊啦～

其實在部署的時候也可以搭配各種自動化工具或是官方提供的CLI工具。

之後有機會再跟大家介紹啦😂

下次見掰掰～
