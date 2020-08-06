# 安裝Redis-6.0.5-集群-以CentOS 7為例
<p style="text-align:right;">2020.06.23 蔡元泰製</p> 

---

## 安裝
### 使用以下指令下載，提取和編譯Redis：
``` shell
$ mkdir /opt/redis
$ cd /opt/redis
$ wget http://download.redis.io/releases/redis-6.0.5.tar.gz
$ tar xzf redis-6.0.5.tar.gz
$ cd redis-6.0.5
$ sudo make distclean && sudo make
```

> ### 可能錯誤:  
> - cc: Command not found -> 未有GCC編譯器(C語言)，安裝即可(版本更新如後).
> ![ccNotFound](./redis-6.0.5-install-cluster-centOS7_img/install/ccNotFound.png)  
> 執行:  
>   ```shell
>   $ cd /opt/redis/redis-6.0.5
>   $ sudo yum install gcc 
>   $ rpm -qa |grep gcc # 驗證gcc是否安裝成功  
>   ```    
>   ![testGccInstallation](./redis-6.0.5-install-cluster-centOS7_img/install/testGccInstallation.png)  
>   ```shell  
>   $ sudo make  
>   ```
> -  jemalloc/jemalloc.h: No such file or directory -> 上次編譯有殘留文件，需清理後再重新編譯，並指定Redis分配器為libc
> ![jemalloc-jemallocNoSuchFileOrDirectory](./redis-6.0.5-install-cluster-centOS7_img/install/jemalloc-jemallocNoSuchFileOrDirectory.png)  
> 執行:  
>       ```shell
>       cd /opt/redis/redis-6.0.5
>       sudo make distclean && make MALLOC=libc
>       ```
> - server.c:5172:31: error: ‘struct redisServer’ has no member named 'XXXXX' -> gcc版本不夠新(CentOS 7 默認安裝4.8.5)，升級至gcc 9.   
> ![structRedisServerError](./redis-6.0.5-install-cluster-centOS7_img/install/structRedisServerError.png)  
> 執行:  
>   ```shell  
>   $ cd /opt/redis/redis-6.0.5  
>   $ make distclean # 清除編譯生成的文件.   
>   $ sudo yum -y install centos-release-scl  
>   $ sudo yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils  
>   $ scl enable devtoolset-9 bash # scl指令啟用只是臨時的，退出shell或重新打開一個shell就會恢復原系統gcc版本.  
>   ```  
>   執行以永久使用.  
>   ```shell  
>   $ sudo sh -c "echo source /opt/rh/devtoolset-9/enable >> /etc/profile"  
>   ```  
>   重打shell (或重開機)，再次編譯.  
>   ```shell  
>   $ gcc -v  # 驗證gcc版本.    
>   ```  
>   ![testGccV9](./redis-6.0.5-install-cluster-centOS7_img/install/testGccV9.png)  

安裝成功:   
> ![installationSucceeds](./redis-6.0.5-install-cluster-centOS7_img/install/installationSucceeds.png)  

<div style="text-align:center;">
<a href="#目錄">回到目錄</a>
</div>

---

## 試啟動
### 使用以下指令運行Redis：
``` shell
$ cd redis-6.0.5
$ src/redis-server
```
> ### 可能錯誤 (但可能不影響運行):
> ![runRedisWithError](./redis-6.0.5-install-cluster-centOS7_img/install/runRedisWithError.png) 
> - WARNING: The TCP backlog setting of 511 cannot be... (監聽佇列的長度預設128):   
>       執行:   
>   ```shell
>   $ echo "net.core.somaxconn = 2048" | sudo tee -a /etc/sysctl.conf 
>   ``` 
>   ![configsomaxconn](./redis-6.0.5-install-cluster-centOS7_img/config/configsomaxconn.png)
> 
> - WARNING overcommit_memory is set to 0! (内存分配策略參數設置為0):  
>   執行:  
>   ```shell
>   $ echo "vm.overcommit_memory = 1" | sudo tee -a /etc/sysctl.conf
>   ```
>    ![configOverCommitMemoryTo1](./redis-6.0.5-install-cluster-centOS7_img/config/configOverCommitMemoryTo1.png)    
> 
> - WARNING you have Transparent Huge Pages (THP) support enabled in your kernel (你使用的是透明大頁，可能導致redis延遲和內存使用問題)....  
> 執行: [[13]](#[13])  
>   - 暫時解決方法  
>       ```shell
>       $ sudo su # 切換至root帳號，用sudo無法  
>       # echo never > /sys/kernel/mm/transparent_hugepage/enabled
>       # exit # 切換回User帳號
>       ```
>  
>   - 永久解決方法  
>       ```shell
>       $ sudo su
>       # vim /etc/rc.local
>       # echo never > /sys/kernel/mm/transparent_hugepage/enabled  
>       # exit  
>       ```  
運行成功:
![runServerAfterResloveTHPError](./redis-6.0.5-install-cluster-centOS7_img/install/runServerAfterResloveTHPError.png)
 
<div style="text-align:center;">
<a href="#目錄">回到目錄</a>
</div>

---

## 使用內建客戶端與Redis溝通
### 保持Redis運行，另外開啟shell，並使用下列指令與Redis溝通:
``` shell
$ cd redis-6.0.5
$ src/redis-cli
```  
![interactWithRedis](./redis-6.0.5-install-cluster-centOS7_img/install/interactWithRedis.png) 

- 新增資料: > set foo bar  
![setData](./redis-6.0.5-install-cluster-centOS7_img/install/setData.png) 
- 搜尋資料: > get foo  
![getData](./redis-6.0.5-install-cluster-centOS7_img/install/getData.png) 

<div style="text-align:center;">
<a href="#目錄">回到目錄</a>
</div>

---

## 快速執行  
### 使用以下指令將主程式複製到/usr/local/bin/: 
``` shell
$ cd redis-6.0.5/
$ sudo cp src/redis-server /usr/local/bin/
$ sudo cp src/redis-cli /usr/local/bin/
```
可以直接無視現在的目錄，直接執行redis的server與client
執行:
- server端:
    ```shell
    $ redis-server
    ```
    ![runRedisDirectly](./redis-6.0.5-install-cluster-centOS7_img/install/runRedisDirectly.png) 
- 客戶端:
  ``` shell
  $ redis-cli
  ```
     ![interactWithRedisDirectly](./redis-6.0.5-install-cluster-centOS7_img/install/interactWithRedisDirectly.png)   

<div style="text-align:center;">
<a href="#目錄">回到目錄</a>
</div>

---

## 啟動Redis集群模式  
### 依循下列步驟，啟動設置並啟動Redis集群模式  
1. 建立相關目錄
    ```shell  
    $ cd /opt/redis
    $ sudo mkdir redis-cluster
    $ cd redis-cluster
    $ sudo mkdir conf data log
    $ sudo mkdir -p data/redis-7000 data/redis-7001 data/redis-7002 data/redis-8000 data/redis-8001 data/redis-8002
    ```  
    備註: port: 7000為主機(master)，8000則為從機(slave)，以利區分.  

2. 建置Redis相關配置檔  
      1. 建立配置檔(依port命名)  
         ``` shell  
         $ cd /opt/redis/redis-cluster/conf
         $ sudo cp /opt/redis/redis-6.0.5/redis.conf redis.conf
         $ touch redis-7000.conf redis-7001.conf redis-7002.conf
         $ touch redis-8000.conf redis-8001.conf redis-8002.conf
         $ sudo chmod 775 * # 如果有需要，統一修改權限
         ```  
      2. 編輯配置檔(以7000為例)  
         ```shell  
         $ sudo vim redis-7000.conf 
         ```   
         
           - 於配置檔中，輸入相關配置  
            
            ```shell  
            include /opt/redis/redis-cluster/conf/redis.conf
            daemonize yes  
            bind 127.0.0.1 192.168.200.141  
            dir /opt/redis/redis-cluster/data/redis-7000  
            pidfile /var/run/redis-cluster/redis-7000.pid  
            logfile /opt/redis/redis-cluster/log/redis-7000.log
            port 7000  
            cluster-enabled yes  
            cluster-config-file /opt/redis/redis-cluster/conf/node-7000.conf  
            cluster-node-timeout 10000  
            appendonly yes  
            ```  
   
           - 參數說明:  
            ```shell  
            # 其他本配置檔未設定之設定參照來源   
            include /opt/redis/redis-cluster/conf/redis.conf  
            
            # 守護模式(背景執行)  
            daemonize yes  
            # 繫結的主機埠  
            bind 127.0.0.1 192.168.200.141 # 外部ip依需求調整   
            # 資料存放目錄  
            dir /opt/redis/redis-cluster/data/redis-7000  
            # 程序檔案  
            pidfile /var/run/redis-cluster/redis-7000.pid  
            # 日誌檔案  
            logfile /opt/redis/redis-cluster/log/redis-7000.log  
            # 埠號  
            port 7000  
            # 開啟叢集模式  
            cluster-enabled yes  
            # 叢集的配置，配置檔案首次啟動自動生成  
            cluster-config-file /opt/redis/redis-cluster/conf/node-7000.conf  
            # 請求超時，設定10秒  
            cluster-node-timeout 10000
            # aof日誌開啟，有需要就開啟，它會每次寫操作都記錄一條日誌  
            appendonly yes  
            ```
           - 其他配置檔(7001, 7002, 8000, 8001, 8002)，同理.  
  
3. 啟動伺服器  
    執行下列指令，以啟動伺服器.  
    ```shell
    $ cd /opt/redis/redis-cluster
    $ redis-server conf/redis-7000.conf
    $ redis-server conf/redis-7001.conf
    $ redis-server conf/redis-7002.conf
    $ redis-server conf/redis-8000.conf
    $ redis-server conf/redis-8001.conf
    $ redis-server conf/redis-8002.conf
    ```
    > ### 可能錯誤:  
    > - Can't open the append-only file: Permission denied  
    > ![append-onlyFile_PermissionDenied](./redis-6.0.5-install-cluster-centOS7_img/clusterMode/append-onlyFile_PermissionDenied.png)  
    >   原因: 權限不足寫入aof日誌.  
    >   執行下列指令開啟相關權限  
    >   ```shell 
    >   $ cd /opt/
    >   $ sudo chmod -R 775 redis # 修改目錄下權限
    >   $ sudo chown -R tai:root redis # 修改目錄(檔案)擁有者
    >   ```  


4. 測試是否啟動  
   執行以下指令，檢視伺服器是否啟動.  
    ```shell  
    $ ps -ef | grep redis
    ```  
    成功啟動
    ![ps-efGredis](./redis-6.0.5-install-cluster-centOS7_img/clusterMode/ps-efGredis.png)  

5. 建立集群關係  
   執行以下指令，建立Redis集群關係. Redis 5以上方可使用，以下則需使用redis-trib.rb建立(本篇不演示).  
   ```shell  
   $ redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001  127.0.0.1:7002 127.0.0.1:8000 127.0.0.1:8001 127.0.0.1:8002  --cluster-replicas 1  # cluster-replicas 表示每1個主機要有1個從機.
   ``` 
   備註: 按照從主機到從機的方式， 從左到右，依次排列6個 Redis節點。
    ![createClusterMode](./redis-6.0.5-install-cluster-centOS7_img/clusterMode/createClusterMode.png)  

6. 檢視集群關係  
    執行以下指令，檢視各port伺服器之間的關係.  
    1. 以客戶端進入port 7000的Redis
        ```shell  
        $ redis-cluster]$ redis-cli -c -p 7000 # -c: cluster, -p: port  
        $ 127.0.0.1:7000> info replication
        ``` 
        port 7000為主機，連接1個從.  
        ![cluster-master](./redis-6.0.5-install-cluster-centOS7_img/clusterMode/cluster-master.png)  
    2. 退出該客戶端  
        ```shell  
        $ 127.0.0.1:7000> quit
        ``` 

    3. 以客戶端進入port 8000的Redis  
         ```shell  
        $ redis-cluster]$ redis-cli -c -p 8000 
        $ 127.0.0.1:8000> info replication
        ```   
        port 8000為從機. 跟隨之主機為port 7000.  
        ![cluster-slave](./redis-6.0.5-install-cluster-centOS7_img/clusterMode/cluster-slave.png) 

    4. 退出該客戶端  
        ```shell  
        $ 127.0.0.1:8000> quit  
        ```   

<div style="text-align:center;">
<a href="#目錄">回到目錄</a>
</div>

---

## 簡單測試  
### 依循下列步驟，簡單測試Redis集群功能  



<div style="text-align:center;">
<a href="#目錄">回到目錄</a>
</div>

---  

## 參考來源: 
1. https://redis.io/topics/cluster-tutorial
2. https://www.itread01.com/lqqf.html

<div style="text-align:center;">
<a href="#目錄">回到目錄</a>
</div>

---


