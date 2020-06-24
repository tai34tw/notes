# Redis-6.0.5安裝-以CentOS 7為例
<p style="text-align:right;">2020.06.23 蔡元泰製</p> 

## 目錄
- [安裝](#安裝)        
- [測試](#測試)        
- [啟動](#啟動)          
- [使用內建客戶端與Redis溝通](#使用內建客戶端與redis溝通)          
- [快速執行](#快速執行)           
- [關閉Server](#關閉server)     
- [ 管理工具(Windows 10為例)](#管理工具windows-10為例)    
- [參考來源](#參考來源)

---

## 安裝 
### 使用以下指令下載，提取和編譯Redis： [[1]](#[1])[[2]](#[2])
``` shell
$ wget http://download.redis.io/releases/redis-6.0.5.tar.gz
$ tar xzf redis-6.0.5.tar.gz
$ cd redis-6.0.5
$ make distclean && make
```
> ### 可能錯誤:  
> - cc: Command not found -> 未有GCC編譯器(C語言)，安裝即可(版本更新如後). [[3]](#[3]) [[4]](#[4])
> ![ccNotFound](./redis-6.0.5-install-centOS7_img/install/ccNotFound.png)  
> 執行:  
>   1. $ cd redis-6.0.5
>   2. $ (sudo) yum install gcc 
>   3. 驗證gcc是否安裝成功: $ rpm -qa |grep gcc   
> ![testGccInstallation](./redis-6.0.5-install-centOS7_img/install/testGccInstallation.png)  
>   4. $ make   
>   
> -  jemalloc/jemalloc.h: No such file or directory -> 上次編譯有殘留文件，需清理後再重新編譯. [[5]](#[5])  
> ![jemalloc-jemallocNoSuchFileOrDirectory](./redis-6.0.5-install-centOS7_img/install/jemalloc-jemallocNoSuchFileOrDirectory.png)  
>   
> - server.c:5172:31: error: ‘struct redisServer’ has no member named 'XXXXX' -> gcc版本不夠新(CentOS 7 默認安裝4.8.5)，升級至gcc 9. [[6]](#[6])  
> ![structRedisServerError](./redis-6.0.5-install-centOS7_img/install/structRedisServerError.png)  
> 執行:
>   1. $ cd redis-6.0.5
>   2. $ make distclean # 清除編譯生成的文件.   
>   3. $ (sudo) yum -y install centos-release-scl
>   4. $ (sudo) yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
>   5. $ scl enable devtoolset-9 bash # scl指令啟用只是臨時的，退出shell或重新打開一個shell就會恢復原系統gcc版本.
>   6. $ sudo sh -c "echo source /opt/rh/devtoolset-9/enable >> /etc/profile" # 執行以永久使用. [[7]](#[7])
>   7. 重打shell (或重開機)，再次編譯. 
>   8. $ gcc -v  # 驗證gcc版本.   
> ![testGccV9](./redis-6.0.5-install-centOS7_img/install/testGccV9.png)  

安裝成功:   
> ![installationSucceeds](./redis-6.0.5-install-centOS7_img/install/installationSucceeds.png)  

<div align="center">
<a href="#目錄">回到目錄</a>
</div>

---

## 測試 
### 使用以下指令執行Redis運行測試：
```shell
$ cd redis-6.0.5
$ make test
```
> ### 可能錯誤:  
> - You need tcl 8.5 or newer in order to run the Redis test -> 未安裝tcl 8.5. [[8]](#[8])  
> ![runTestError](./redis-6.0.5-install-centOS7_img/install/runTestError.png)  
>   執行:  
>   1. $ cd redis-6.0.5
>   2. $ (sudo) yum install tcl   
>   3. 驗證tcl版本: $ echo "puts [info tclversion]" | tclsh   
> ![testTclVersion](./redis-6.0.5-install-centOS7_img/install/testTclVersion.png)  
>    
> - *** [err]: Test FLUSHALL aborts bgsave in tests/integration/rdb.tcl -> 效能配置不足，增加效能配置即可 (測試無法通過，不代表Redis無法運行).[[9]](#[9])
> ![runTestError2](./redis-6.0.5-install-centOS7_img/install/runTestError2.png)  
> 執行: 增加(虛擬機)記憶體及處理器效能，以下供參:  
>   - 記憶體: 4G.
>   - 處理器: 實體2核心，虛擬2核心，共4核心.
>   - 硬碟: 40G.

運行測試通過:   
![testSucceeds](./redis-6.0.5-install-centOS7_img/install/testSucceeds.png)  
<div align="center">
<a href="#目錄">回到目錄</a>
</div>

---

## 啟動
### 使用以下指令運行Redis：
``` shell
$ cd redis-6.0.5
$ src/redis-server
```
> ### 可能錯誤:
> ![runRedisWithError](./redis-6.0.5-install-centOS7_img/install/runRedisWithError.png) 
> - WARNING: The TCP backlog setting of 511 cannot be... (監聽佇列的長度預設128):   
>       執行: [[13]](#[13])  
>    $ echo "net.core.somaxconn = 2048" | sudo tee -a /etc/sysctl.conf  
>   ![configsomaxconn](./redis-6.0.5-install-centOS7_img/config/configsomaxconn.png)
> 
> - WARNING overcommit_memory is set to 0! (内存分配策略參數設置為0):  
>   執行:  
>    $ echo "vm.overcommit_memory = 1" | sudo tee -a /etc/sysctl.conf
>    ![configOverCommitMemoryTo1](./redis-6.0.5-install-centOS7_img/config/configOverCommitMemoryTo1.png)  
>  備註: 其他參數設定詳見: [[11]](#[11])[[12]](#[12]) 
> 
> - WARNING you have Transparent Huge Pages (THP) support enabled in your kernel (你使用的是透明大頁，可能導致redis延遲和內存使用問題)....  
> 執行: [[10]](#[10])  
>   - 暫時解決方法  
>   1. $ sudo su # 切換至root帳號，用sudo無法  
>   2. \# echo never > /sys/kernel/mm/transparent_hugepage/enabled
>   3. \# exit # 切換回User帳號
>   </br> </br>
>   - 永久解決方法  
>   1. $ sudo su
>   2. \# vim /etc/rc.local
>   3. 加上 echo never > /sys/kernel/mm/transparent_hugepage/enabled  
>   4. \# exit  
> 
運行成功:
![runServerAfterResloveTHPError](./redis-6.0.5-install-centOS7_img/install/runServerAfterResloveTHPError.png)
 
<div align="center">
<a href="#目錄">回到目錄</a>
</div>

---

## 使用內建客戶端與Redis溝通
### 保持Redis運行，另外開啟shell，並使用下列指令與Redis溝通:
``` shell
$ cd redis-6.0.5
$ src/redis-cli
```  
![interactWithRedis](./redis-6.0.5-install-centOS7_img/install/interactWithRedis.png) 

- 新增資料: $ set foo bar  
![setData](./redis-6.0.5-install-centOS7_img/install/setData.png) 
- 搜尋資料: $ get foo  
![getData](./redis-6.0.5-install-centOS7_img/install/getData.png) 
<div align="center">
<a href="#目錄">回到目錄</a>
</div>

---

## 快速執行  
### 使用以下指令將主程式複製到/usr/local/bin/: [[4]](#[4])  
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
    ![getData](./redis-6.0.5-install-centOS7_img/install/runRedisDirectly.png) 
- 客戶端:
  ``` shell
  $ redis-cli
  ```
     ![getData](./redis-6.0.5-install-centOS7_img/install/interactWithRedisDirectly.png)   

<div align="center">
<a href="#目錄">回到目錄</a>
</div>

---

## 開啟外部訪問
## 使用
# //TODO
<div align="center">
<a href="#目錄">回到目錄</a>
</div>

---

## 關閉Server
### 在客戶端使用以下指令關閉Server:
- 正常退出: [[98]](#[98])
    ```shell
    $ pkill redis-server
    ```
     查看原先開啟的Server端視窗  
     ![pkillRedisServer](./redis-6.0.5-install-centOS7_img/shutDown/pkillRedisServer.png) 

- 可能不是好方法，但能關掉Server: [[99]](#[99])
    ```shell
    $ redis-cli SHUTDOWN NOSAVE
    ```
    查看原先開啟的Server端視窗  
     ![shutdownRedisServerAndNoSave](./redis-6.0.5-install-centOS7_img/shutDown/shutdownRedisServerAndNoSave.png) 

- 強制關閉(沒有log釋出): [[98]](#[98])
    ```shell
    $ ps -a # 查看現有redis-server之PID
    $ kill -9 3365 # 3365為該server之PID
    ```
    ![kill-9RedisServerCode](./redis-6.0.5-install-centOS7_img/shutDown/kill-9RedisServerCode.png)  
    查看原先開啟的Server端視窗  
    ![kill-9RedisServer](./redis-6.0.5-install-centOS7_img/shutDown/kill-9RedisServer.png)

## 管理工具(Windows 10為例)
### 使用以下指令於cmd下操作:
1. 檢查有無npm環境:
    ```shell
    > npm -v
    ```
   - 有
         ![getNpmVersion](./redis-6.0.5-install-centOS7_img/redis-commander/getNpmVersion.png)
   - 無  
     請自行安裝.
2. 安裝redis-commander [[100]](#[100])
   ```shell
   > npm install -g redis-commander
   ```
    ![installRedisCommander](./redis-6.0.5-install-centOS7_img/redis-commander/installRedisCommander.png)
3. 啟動redis-commander
    ```shell
    > redis-commander
    ```
    ![runRedisCommander](./redis-6.0.5-install-centOS7_img/redis-commander/runRedisCommander.png)  
4. 將上述畫面ip (http://127.0.0.1:8081)以瀏覽器開啟  
    ![redisCommander-chrome](./redis-6.0.5-install-centOS7_img/redis-commander/redisCommander-chrome.png)  

附註:其他管理工具GUI: [[101]](#[101])  
<div align="center">
<a href="#目錄">回到目錄</a>
</div>

---

## 參考來源
1. https://redis.io/download <a name='[1]'></a>
2. https://linuxize.com/post/how-to-install-and-configure-redis-on-centos-7/ <a name='[2]]'></a>
3. https://www.itread01.com/content/1547765307.html <a name='[3]'></a>
4. https://dotblogs.com.tw/jakeuj/2015/12/24/Redis <a name='[4]'></a>  
5. https://www.cnblogs.com/operationhome/p/10342258.html <a name='[5]'></a>  
6. https://www.cnblogs.com/sanduzxcvbnm/p/12955145.html <a name='[6]'></a> 
7. https://stackoverflow.com/questions/51030702/how-to-write-in-etc-profile-using-bash-permission-denied <a name='[7]'></a> 
8. https://www.cnblogs.com/zhaoshunjie/p/5907029.html <a name='[8]'></a> 
9. https://github.com/antirez/redis/issues/2126 <a name='[9]'></a> 
10. https://www.jianshu.com/p/7ca4b74c92be  <a name='[10]'></a> 
11. https://access.redhat.com/documentation/zh-tw/red_hat_enterprise_linux/6/html/performance_tuning_guide/s-memory-captun <a name='[11]'></a> 
12. https://blog.csdn.net/hjx_1000/article/details/46412557 <a name='[12]'></a> 
13. https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/424386 <a name='[13]'></a> 
14. http://www.jeepxie.net/article/964280.html <a name='[98]'></a> 
15. https://redis.io/commands/shutdown <a name='[99]'></a> 
16. https://www.npmjs.com/package/redis-commander <a name='[100]'></a>
17. https://redislabs.com/blog/so-youre-looking-for-the-redis-gui/ <a name='[101]'></a>