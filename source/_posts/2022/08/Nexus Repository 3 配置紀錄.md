---
title: Nexus Repository 3 配置紀錄
categories:
  - DevOps
  - Nexus
date: 2022-08-09 18:31:30
tags:
  - DevOps
  - Nexus
cover: https://user-images.githubusercontent.com/12562305/183628699-88260965-460a-4767-aa89-d526da8fc568.png
---
![image](https://user-images.githubusercontent.com/12562305/183628699-88260965-460a-4767-aa89-d526da8fc568.png)

首先我們需要先從docker hub 將nexus的image載下來，並且在docker上執行

```bash
# 從docker hub 將nexus的image載下來
docker pull sonatype/nexus3
# 在本地端的8080 port執行 nexus
docker run -d -p 8081:8081 --name nexus sonatype/nexus3
```

第一次開啟會需要比較久的時間，可以先去沖杯咖啡XD，接著在瀏覽器輸入
`http://localhost:8081`， 如果有出現以下畫面代表成功了。
![image](https://user-images.githubusercontent.com/12562305/183630091-f2b2dcd0-27f5-4763-9ab3-f6dd78141644.png)

接著我們需要進去container裡面拿到預設的密碼


```bash
docker exec -it nexus bash
cat /opt/sonatype/sonatype-work/nexus3/admin.password
#退出
exit
```

![image](https://user-images.githubusercontent.com/12562305/183631359-1d043a10-00fb-4a12-80b2-e0a73651f62e.png)

接著按畫面右上角的Sign In，並且輸入帳號密碼，開始初始設置
![image](https://user-images.githubusercontent.com/12562305/183632088-fae10a5d-7b32-461e-bcb3-477afdbebadd.png)

帳號: admin
密碼: 剛剛透過bash拿到的密碼

接著就會彈出以下視窗
![image](https://user-images.githubusercontent.com/12562305/183632386-87d8cf00-932a-456c-892a-1e7dc417ffcb.png)

設定新密碼
![image](https://user-images.githubusercontent.com/12562305/183632554-141d0f42-f499-45e4-b65a-580a6d265cbb.png)

需要使用者帳號密碼才可以存取nexus repo
![image](https://user-images.githubusercontent.com/12562305/183632630-ee734449-e6cb-4557-a504-b3d2eaa979ac.png)

接著就可以快樂的使用私有的nexus repo 啦~
