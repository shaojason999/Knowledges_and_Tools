## iptables
強烈推薦看這個系列教學，不過因為我太晚發現，所以下面筆記主要不是來自這裡:  
[iptables零基礎快速入門系列](http://www.zsythink.net/archives/category/%E8%BF%90%E7%BB%B4%E7%9B%B8%E5%85%B3/iptables/page/2/)
### iptables基本認知
1. iptables為Linux kernel防火牆軟體Netfilter的操縱軟體
1. iptables規則設定:
    * 有n張table，每張table有m個chain，每個chain有i個rule
2. 四個預設table:
    * filter table
        * 負責過濾進入主機或離開主機、還有經過主機做forward的封包
        * 預設chain: INPUT, OUTPUT, FORWARD
    * nat table
        * 負責進行NAT，也就是更改source IP/port或destination IP/port
        * 預設chain: PREROUTING, POSTROUTING, OUTPUT
    * mangle table(可以暫時跳過)
        * 拆解封包，做出修改，並重新封裝
        * 預設chain: PREROUTING, POSTROUTING, OUTPUT, INPUT, FORWARD
    * raw(可以暫時跳過)
        * 關閉表上啟用的連接追蹤機制
        * 預設chain: PREROUTING, OUTPUT
4. 預設chain有5個: PREROUTING, POSTROUTING, OUTPUT, INPUT, FORWARD
    * 不同的table擁有5個中的其中幾個chain
5. rule包括條件與目標(target)
    * 條件包括:source IP, destination IP, protocol等
    * target包括: ACCEPT, DROP, QUEUE, RETURN, REJECT, LOG, DNAT, SNAT以及其他的chain等
    * 不是每個target在每個地方都可以用到(後面會講道)
6. 流程圖
    ![](https://i.imgur.com/5eLWUjS.png)
    (來源:鳥哥)  
    ![](https://i.imgur.com/NoHhQL2.png)  
    * 路徑A. 封包進入主機
    * 路徑B. 沒有用到主機資源，而是向後端主機過去
    * 路徑C. 由本機回應請求或是主動發出封包
    * note: 如果主機沒有拿來轉送之類的功能，那其實用不到nat
    * 可以注意看PREROUTING放一起，INPUT放一起，同一類放一起
7. 重要觀念:
    * 上圖的這一套流程是固定的
    * 不同table，同一個預設chain會依序執行，順序為 raw->mangle->nat->filter
    * 至於甚麼時候進到下一步，則跟target有關
        * ACCEPT: 直接就會順著箭頭往下(往不同table的同一個chain繼續比對，或是剛好進到下一組chain)
        * DROP: 直接丟掉不繼續比對
        * REJECT: 傳送封包通知對方，不繼續比對
        * REDIRECT: 比如PNAT，做完這個之後，繼續比對其他規則(同一個chain)，而不是順著箭頭往下
        * LOG: 紀錄在/var/log中，然後繼續比對
        * RETURN: 從目前的chain回到呼叫這個chain的那個chain繼續比對。比如我自己創造一個chain，可能就會用到這個回到call我創造的chain的那個chain
        * 其他詳細請看[參考資料](https://blog.csdn.net/xiaoxiaozhu2010/article/details/32318469)
    * 某些target只能在某些chain中使用
    ```
    //這句是可以的
    $ sudo iptables -t nat -A PREROUTING -p tcp -d 15.45.23.67 --dport 80 -j DNAT --to-destination 192.168.10.1-192.168.10.10:80-100
    
    //改成POSTROUTING就會出錯
    $ sudo iptables -t nat -A POSTROUTING -p tcp -d 15.45.23.67 --dport 80 -j DNAT --to-destination 192.168.10.1-192.168.10.10:80-100
    iptables: No chain/target/match by that name.
    
    //查看錯誤訊息(不是每個系統都可以這樣看)
    $ dmesg
    [ 4996.734336]x_tables: ip_tables: DNAT target: used from hooks POSTROUTING, but only usable from PREROUTING/OUTPUT(of nat table)
    ```
8. 自定義chain的使用
    * 可以在某個table中創一個chain，而這個chain必須被其他chain引用才會有作用
    * 在預設chain中的target可以連到這個自定義chain，由這個chain來完成比對
    * 自定義chain的比對結果等同呼叫他的chain的效果(應該啦)(比如說在自定義chain ACCEPT，等同在INPUT ACCEPT，然後往下走)
    * 自定義的chain()內寫的不是policy而是n references
    ```
    //雖然這段在幹嘛我也看不懂，但是重點是chain可以互相call對方
    $ sudo iptables -L
    ...
    Chain FORWARD (policy DROP)
    target     prot opt source              
    DOCKER-USER  all  --  anywhere             anywhere            
    DOCKER-ISOLATION-STAGE-1  all  --  anywhere             anywhere            
    ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED         
    ACCEPT     all  --  anywhere             anywhere   
    
    Chain DOCKER-ISOLATION-STAGE-1 (1 references)
    target     prot opt source               destination         
    DOCKER-ISOLATION-STAGE-2  all  --  anywhere             anywhere            
    RETURN     all  --  anywhere             anywhere            
    
    Chain DOCKER-ISOLATION-STAGE-2 (1 references)
    target     prot opt source               destination         
    DROP       all  --  anywhere             anywhere            
    RETURN     all  --  anywhere             anywhere    
    ...
    ```
    * [參考資料](http://www.zsythink.net/archives/1625)
7. 重要觀念整理: 
    * 一般不會自行新增table, 但會新增chain跟rule
    * chain之間可以跳來跳去
    * 同一個chain的rules有比對順序，所以插入順序很重要
    * 不同table，但是同預設chain會照順序比對:  
    raw->mangle->nat->filter  
    ![](https://i.imgur.com/zAKURoi.png)

    * 比對流程是固定的，可以看上面的圖，頂多中間會跳到其他自定義的chain
    * 比對成功後的動作(target)會影響是否繼續比對(不比對、比對同一個table的同一個chain、比對不同table的同一個chain、不同table的不同chain)(如果是繼續比對，順序可以看上面的圖)
![](https://i.imgur.com/BaFnYEh.png)
    * 不是所有target都適用於所有地方，例如: 
        * DNAT不能用在POSTROUTING chain
        * DROP不能用再nat table

### 一些常用指令
1. 查表(預設table為filter)
    ```
    // 整理過的資料
    $  sudo iptables -t nat -L -nv
    Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
    pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL
    
    Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
    pkts bytes target     prot opt in     out     source               destination         
    
    Chain OUTPUT (policy ACCEPT 32 packets, 2116 bytes)
    pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL
    
    Chain POSTROUTING (policy ACCEPT 32 packets, 2116 bytes)
    pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0           
    
    Chain DOCKER (2 references)
    pkts bytes target     prot opt in     out     source              destination         
    0     0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0       
    
    // 指令形式
    $ sudo iptables-save -t nat
    # Generated by iptables-save v1.6.0 on Mon Mar 25 13:19:50 2019
    *nat
    :PREROUTING ACCEPT [0:0]
    :INPUT ACCEPT [0:0]
    :OUTPUT ACCEPT [19:1231]
    :POSTROUTING ACCEPT [19:1231]
    :DOCKER - [0:0]
    -A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
    -A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
    -A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
    -A DOCKER -i docker0 -j RETURN
    COMMIT
    # Completed on Mon Mar 25 13:19:50 2019
    ```
2. 新增規則(依據protocol, port, state(連線狀態), MAC, source IP, destination IP 等等來限制)
    ```
    $ sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
    $ sudo iptables -A INPUT -j DROP
    $ sudo iptables -I INPUT 2 -m state --state ESTABLISHED,RELATED -j ACCEPT
    $ sudo iptables -I INPUT -m mac --mac-source aa:bb:cc:dd:ee:ff -j ACCEPT
    ```
    * -A --append 把rule插在某條chain的最後面
    * -I --insert 可以選擇插入為第幾條(預設為1)
    * -s 選擇source IP
    * -j --jump 選擇目標(target)動作

    * -m --match 外掛模組。後面可以接一些東西，比如 --state, --mac:
        * ESTABLISHED為向別的server發出連接請求(state為new)後，後續的封包
        * RELATED為與已經存在的連結的相關封包
3. 刪除規則
    ```
    //刪除第三條
    $ sudo iptables -D INPUT 3
    
    //刪除filter表的全部規則
    $ sudo iptables -F
    
    //刪除filter表的input的全部規則
    $ sudo iptables -F INPUT
    ```
4. 改變預設政策(Policy)，也就是比對都失敗時做的動作
    ```
    $ sudo iptables -P INPUT DROP
    ```

### 實際操作
#### 情境1: 限制來源IP(可以達到限制訪問IP，因為收不到回覆封包)
1. iptables為空，且能正常上網
    ```
    $ sudo iptables -t filter -L
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination
    
    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination
    
    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination
    ```
![](https://i.imgur.com/iCjGYwE.png)

2. 新增限制規則，無法連上限制網路位置(106.10.250.11)
    ```
    $ sudo iptables -A INPUT -s 106.10.250.11 -j DROP
    ```

![](https://i.imgur.com/3aawEzK.png)

3. 刪除全部規則，回復正常
    ```
    $ sudo iptables -F
    ```

![](https://i.imgur.com/wV6VStR.png)

#### 情境2: 主機只當http server，其他服務一切拒絕
1. 只允許tcp連線且port為80
    ```
    $ sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
    $ sudo iptables -A INPUT -j DROP
    ```
    * 這麼做雖然可以只接受http連線請求(state為new)，但是
    * 我的主機想要主動連出去時，對方的回覆會收不到(我的電腦現在只可以當server，不能當一般電腦上網)
2. 增加一條在DROP之前
    ```
    $ sudo iptables -I INPUT 2 -m state --state ESTABLISHED,RELATED -j ACCEPT
    $ sudo iptables -L
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination
    ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:http
    ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
    DROP       all  --  anywhere             anywhere
    ```
    * 此時對方的回復(state為ESTABLISHED,RELATED)主機就可以接收了

### 利用Docker 布建 iptables
1. 下載這個[image](https://hub.docker.com/r/devrt/container-firewall)(詳細檔案請看[github](https://github.com/devrt/docker-container-firewall))
    ```
    $ docker pull devrt/container-firewall
    $ docker run --name 123 --env ACCEPT_PORT="80,443" -itd --restart=always --cap-add=NET_ADMIN --net=host devrt/container-firewall
    ```
    * \-\-env 選擇可用port。這是在設定container的系統參數(在這個例子中是執行.sh時會用到)
    * -itd 的d是在背景執行
    * \-\-restart=always 每次實體主機重開機，都會自動開始(firewall會開啟)
    * \-\-cap-add=NET_ADMIN 這樣才可以在conatiner使用iptables的功能
    * \-\- net=host container跟實體主機用一樣的網路設定，介紹可看[這裡](https://ithelp.ithome.com.tw/articles/10193457)
    * 原本的iptables
![](https://i.imgur.com/HDyQR0I.png)
    * run之後的iptables，可以發現多了CONTAINER-FIREWALL及對應的rules
![](https://i.imgur.com/ZbTSe6p.png)

2. 這個結果就是: 
    * 原本的主機跟剛剛這個container沒有受到firewall影響，因為影響到的是連到CONTAINER-FIREWALL的DOCKER相關chain。而剛剛這個container因為是\-\-net=host，所以也沒有受到firewall的影響)
    * 如果這時候隨便用一個image創一個container，則會受到firewall的保護，因為docker都會用到CONTAINER-FIREWALL這個chain
3. 運作方法:
    * 用Dockerfile把.sh檔複製到image
    * .sh檔裡面寫了iptables的相關指令
    * .sh檔是在開啟conatiner後才執行(在Dockerfile裡的最後一句有寫到 CMD ["/bin/enforce-rules.sh"])，並且可以在$docker run 時傳參數給 .sh(透過\-\-env)
4. 加入其他規則的方法: 比如限制IP來源
    * 先去[github](https://github.com/devrt/docker-container-firewall) clone下所有程式碼
    * 在.sh檔裡面加入你要的規則
    * 用Dockerfile重新build一個image
5. 其他討論:
    * 其實這個firewall不一定要用docker來實作，可以直接在實體主機創一些規則，一樣可以達到。只是用docker比較帥吧，而且每次開機會自動運行，雖然不用docker也有辦法每次開機自動運行。


---
參考資料  
[文科生也能看懂的iptables教程(1)](http://dallascao.com/cn/iptables-tutorial-for-newbies/)  
[iptables零基礎快速入門系列](http://www.zsythink.net/archives/category/%E8%BF%90%E7%BB%B4%E7%9B%B8%E5%85%B3/iptables/page/2/)  
[鳥哥的Linux私房菜](http://linux.vbird.org/linux_server/0250simple_firewall.php#netfilter)
