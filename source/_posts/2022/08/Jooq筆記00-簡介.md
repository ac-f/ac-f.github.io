---
title: Jooq筆記00-廢話與簡介
categories:
  - Java
  - Jooq
date: 2022-08-09 14:31:30
tags:
  - Java
  - Jooq筆記
cover: https://user-images.githubusercontent.com/12562305/183546328-06586ed3-2c12-44bf-a9fc-6108cf37c4a7.png
---
![image](https://user-images.githubusercontent.com/12562305/183546328-06586ed3-2c12-44bf-a9fc-6108cf37c4a7.png)

## 前言

最近因為工作的關係開始寫Java，並且開始認識這個厲害的工具，但因為網路上比較少資源，想說可以來貢獻一下。

## 解決了什麼問題?

每個工具、技術的誕生通常是為了解決現階段無法解決的問題，或是讓人們更便於達到某種效益，那Jooq到底幫助了我們解決了什麼問題呢?

1. 有Code Generator可以幫助我們完成日常繁瑣的CRUD，提升開發效率
2. Code Generator產生的不只是常見的Dao物件，甚至一併產生了POJO/Record/Table物件，讓你享用數不盡的Intelli Sense 減少Typo Error的窘境

### AC你這樣講反而製造了我很多問題... 什麼是...

#### Code Generator

可以幫你產生常用程式碼的工具，例如我們在寫CRUD時常常會用到findById, update, create...等方法，這些Jooq都會幫我們做好，我們只要負責引入並使用就好了。

#### Intelli Sense

就是當我們使用IDE/Editor在開發時，輸入特定內容會自動提示我們的工具，如下圖
![image](https://user-images.githubusercontent.com/12562305/183549763-ed059b3a-305c-4341-9942-cbe3f298c318.png)

#### DAO/POJO物件

這邊強烈建議可以拜讀李啟嘉大大的[筆記](https://hackmd.io/@MonsterLee/HJyAdgRBB)，如果還是看不懂沒關係，我們會透過之後的文章來讓你更了解創建這些物件的優勢。
