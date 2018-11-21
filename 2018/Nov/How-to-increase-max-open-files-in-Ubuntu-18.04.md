### How to increase max open files in Ubuntu 18.04?

#### 描述
今天發現在新的專案跑 maven unit tests 的時候一直失敗，看了一下 error log 滿滿的 `Too many open files`

上網查了一下，就是你測試程式在跑得時候開了太多的檔案，超過了系統的限制。
測試半天網路上的調整方式，登出登入/重開機都沒有生效...

後來才是成功，原來是 ubuntu 有GUI和無GUI的生效方式不一樣
感謝網路上的大神們！

主要參考這一篇文章：
https://superuser.com/questions/1200539/cannot-increase-open-file-limit-past-4096-ubuntu

#### 相關指令/設定檔
* `$ cat /proc/sys/fs/file-max` : 
* `$ ulimit -n`
* `$ ulimit -a`
* `/etc/systemd/system.conf`
* `/etc/security/limits.conf`: edit it

#### 概念
* 先看作業系統本身能最多支援的可開啟檔案數量（我總共有多少）- `$ cat /proc/sys/fs/file-max` 
* 在看目前系統預設的可開啟檔案數量（我目前提供多少給使用者）- `$ ulimit -n`
* 調整目前預設的檔案數量到你需要的數字（當然不能超過總量）

#### 調整可開啟檔案大小的步驟 
* 檢查可開啟檔案總數量
    ```
    # 看系統可開啟檔案總量，我的電腦是 1602274
    $ cat /proc/sys/fs/file-max 

    1602274
    ```
  
* 檢查目前預設可開啟檔案的數量
    ```
    # ulimit -n 看可開啟檔案數量，可以看到我的電腦預設是 1024 (也可以試試 ulimit -a，可以看到更多資訊)
    $ ulimit -n
    
    1024
    ``` 
    
##### 無 GUI
* 編輯檔案 /etc/systemd/system.conf 加入以下設定
    ```
    # 設定預設的可開啟檔案數量 20000
    DefaultLimitNOFILE=20000
    ```
* 重新開機

##### 無 GUI
* 編輯檔案 /etc/security/limits.conf 並加入以下設定
    ```
    # 設定使用者 tensen 可開啟檔案數量 20000，詳細可以看 limits.conf 的註解
    tensen soft nofile 20000
    tensen hard nofile 20000
    ```
* 重新登出在登入

#### 參考
* http://www.thinkplexx.com/learn/howto/maven2/debug/fixing-too-many-open-files-maven-problem-solution-for-teamcity-hudson-local-and-ssh-shell-environments
* https://superuser.com/questions/1200539/cannot-increase-open-file-limit-past-4096-ubuntu