---
layout: post
title: Memcached vs Redis 適用的狀況思考
date: 2019-04-11 01:32:24.000000000 +09:00
---
### 常見的記憶體暫存服務：Memchached 還是 Redis?
隨著業務需求的擴張以及業務的多樣性，這時候我們的系統就需要增加新的功能，有時候是短時間需要處理大量請求的系統，又是較低硬體設備成本的選擇，該用哪一個？又或者需要改進性能的時候，這是每次技術討論中最常見的一個問題。每當性能需要改善時，採用暫存常常是邁出的第一步。那麼，選擇Memcached 或者 Redis 通常就是需要考慮的地方。哪個能給我們提供更佳的性能？它們的優點和缺點又是什麼？

#### 簡單介紹：
#### Redis
Redis ( REmote DIctionary Server )是一個使用ANSI C編寫的開源記憶體資料庫系統、支援網路、基於記憶體、它支持資料持久化(也就是你可以把它存在記憶體中的資料 dump 到硬碟上,以備下次載入 Redis 時使用), 它支持多種抽象的資料結構(字符串、列表、映射、集合、有序集合、位圖,空間索引等)。

#### Memcached
Memcached 是一種開源且高性能的分布式記憶體對象緩存系統，最初的目的是通過減輕資料庫負載以加速動態web應用程式，這也是它現今主要的用途。Memcached 的特性是簡潔且高效,但其簡單的資料類型系統讓它在某些應用場合下不是那麼合適。

#### 在設計任何暫存系統時，我們考慮如下幾點：

#### Read/write speed
兩者都非常快。
![Image](https://imgur.com/W06c4jI.png)

![Image](https://imgur.com/0ImIAsY.png)
[資料來源＿Synthetic Semantics](http://synsem.com)

除非，我們投入大量的硬體和資源建立龐大的 Memcached ，否則應如以上的數據顯示，兩者效能是差不多的，如果以成本以及開發效率為考量會優先採用 Redis 會是比較好的作法。

#### 記憶體使用率
memcached：使用者指定暫存大小，當您新增項目時，守護程序會快速增長到略大於此大小。除了重新啟動memcached之外，沒有其他的方法可以回收空間，將使用配置的整塊記憶體。
Redis：設置最大尺寸取決於我們的專案合適的情境，Redis將永遠不會使用超過設定，並提供回收記憶體空間且不再使用此記憶體區塊。

#### 硬碟的 I/O 轉存
Redis明顯地優於 Memcached，因為它默認執行此硬碟的轉存並且具有非常好的持久可配置性。 如果沒有第三方套件或工具，Memcached並沒有轉存到硬碟的機制。

#### 擴展性
Redis & Memchached 之間的相似之處:
Memcached/Redis 兩者資源提供都是基於記憶體、Key-Value資料儲存，儘管Redis更準確的說是結構化資料儲存。 Redis是記憶體中的結構化資料儲存器，用於資料庫、暫存、消息代理。兩者（Memcached/Redis）都屬於資料管理方案中的NoSQL家族，都是基於Key-Value儲存的。它們都在記憶體中保存資料，當然使它們作為暫存層特別有用。

#### Redis & Memchached 的架構

![Image](https://i.imgur.com/mhDayK3.png ,"Redis & Memchached")

Memcached提供的每項主要功能及其優勢，都是Redis功能和特性的子集。任何用例中可能使用Memcached的地方都可以對等的使用Redis。
Memcached是一個基於易失性記憶體的Key-Value儲存器。
Redis一樣可以做到（跟Memcached做得一樣好），不僅如此，它還是一個結構化資料庫伺服器。

#### 為什麼選 Memcached?
當暫存相對較小和使用靜態的資料時候，比如HTML程式片段或者驗證碼圖片，Memcached可能更為適合選用。 Memcached內部的記憶體管理在最簡單的應用場景中更為有效，因為它的單位資料消耗相對更少的記憶體資源。

當資料尺寸是動態的時候，Memcached的記憶體管理效率下降的很快，此時Memcached的記憶體會變成碎片。而且，大的資料集經常牽扯到資料 _序列化_（_serialization_），總是需要更多的空間來儲存。如果你使用Memcached，資料會隨著重啟動而丟失，重建暫存是個代價高昂的過程。

Memcached比Redis更具優勢的另一個狀況，擴展性。因為Memcached是多線程的，所以你可以通過給它更多計算資源讓它輕鬆擴展。 而Redis是單線程的，可以通過集群無損水平擴展。集群是一個有效的擴展方案，但是相對來說配置、操作複雜。

但Memcached不支援同步資料功能（資料從一台機器自動複製到另外一台）。

Memcached 非常適合處理高流量的網站。它可以一次性讀取大量的信息，並在優秀的反應時間內返回。



#### 為什麼選 Redis?
Redis有五種主要的資料結構可以選擇，它為應用程序開發人員打開了存在各種可能的新世界。由於其資料結構（使用多種格式儲存資料：List，Set， Hashes，Sorted Set等等）特性，Redis作為暫存系統提供了更多的能力和總體上更好的效率。暫存使用一種稱為“資料回收”的機制，透過從記憶體中刪除舊資料為新資料騰出空間。 Memcached的資料回收機制使用了LRU（Least Recently Used-最近最少使用）算法。

Redis允許對回收進行大小（細粒度）的控制，讓你選擇六種不同的回收策略。 Redis同時支持惰性（被動）和主動回收，只有在需要更多空間或主動啟用時才回收資料。另一方面，Memcached只支持惰性回收。

以下是redis提供的一些功能，可以用於“真實”資料儲存，而不僅僅是暫存。
```
1. 強大的資料類型和可利用它們的強大指令支持。List，Set， Hashes，Sorted Set等。
2. 預設的硬碟持久化（Persistence）支持
3. 使用樂觀鎖的事務支持 (WATCH/MULTI/EXEC)
4. 發布/訂閱（pub/sub）功能，速度極快
5. 高達512MB的鍵值尺寸上限（Memcached每個鍵值限於1MB大小）
6. Lua 腳本支持 (2.6及以上版本)
7. 內置集群支持 (3.0及以上版本)
```

強大的資料類型尤為重要。Redis提供出色的共享佇列（list）、優秀的消息傳遞（pub/sub）解決方案，也是一個存儲會話信息（hashes）的好地方，還有一個引人注目的高分值追踪區域（sorted sets）。

#### 結論
Redis與Memcached相比，性能和記憶體使用情況相當相似。除非你已經在Memcached上投入了大筆資金，不妨可以採用Redis。

Redis可能會非常有用的一些範例應用程序：

電子商務應用：大多數的電子商務應用量級比較重，Redis可以提升你的頁面加載速度。你可以儲存所有的配置文件到Redis，從記憶體中讀取這些配置文件速度會提升不少。你也可以在Redis中儲存完整的頁面暫存，因為它的鍵值容量很大。你也可以儲存Session到Redis。

物聯網應用：在物聯網應用中，物聯網設備非常頻繁的發送資料到伺服器，比如每秒鐘數千條。在把它們存到任何持久性儲存器之前，可以先把這些高容量的原始資料發送到Redis。

實時分析：可以在Memcached上實現一個實時的分析引擎，以資料庫為後盾。但是Redis非常擅長統計列表和一系列事物。在所有的Redis功能特性中，它對鍵值進行排序的能力超過了Memcached，還有計算一組頁面的點擊次數等數據，然後將這些數字彙總進入分析系統，這些資料可通過工作人員輸入到更大的分析引擎。

Redis的單線程設計，同時也帶來了一些重要的隱患。 Redis有資料持久化功能，這個功能與Redis的單線程特性結合，就成了Redis故障的好發區。預設的RDB持久化會阻塞線程，使得Redis對正常請求無法響應，在高流量網站上容易出現大量請求錯誤。當然後來Redis也發展出了AOF持久化方式（預設沒有開啟，要手動開啟），一定程度上減緩了Redis的持久化問題。 Redis會fork一個子進程來單獨處理持久化。可是fork功能並非無代價，它一樣有消耗記憶體資源，影響主程序響應請求的問題。

#### 本篇的學習參考資料

| 標題   | URL                                                |
| ------------ | ---------------------------------------------------|
| Memcached vs. Redis?   | [https://stackoverflow.com](https://stackoverflow.com/questions/10558465/memcached-vs-redis)           |
| Memcached vs Redis, Which One to Pick    | [https://www.linkedin.com](https://www.linkedin.com/pulse/memcached-vs-redis-which-one-pick-ranjeet-vimal)     |
| Memcached vs Redis, Which One to Pick    |[www.ranjeetvimal.com](http://www.ranjeetvimal.com/memcached-vs-redis-one-pick/) |
| Memcached vs Redis, 挑选哪一个？  |[https://blog.csdn.net](https://blog.csdn.net/itguangit/article/details/80019179) |
