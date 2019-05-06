## http與https的介紹與比較
#### http與https的介紹
* http
    * 是明文傳輸
    * 通訊雙方沒有進行認證 
    * 資料容易被截取、竄改
* https [參考資料](彻底搞懂HTTPS的加密机制)
    * 非對稱加密(一個公鑰、一個私鑰)+對稱加密(同一個密鑰)
        * 一開始server先給browser公鑰(非對稱加密)，browser產生一個密鑰，用公鑰加密後傳給server，server用私鑰解開後得到密鑰，之後雙方就用這個密鑰溝通(對稱加密)
        * 因為非對稱加密非常耗時，所以傾向用對稱加密
        * 為了讓對方拿到對稱加密的密鑰，要先用非對稱加密
        * 有可能會被中間人攻擊，所以只有加密不夠，還要證書認證
    * 數字證書
        * 證書裡面包括網站資訊及公鑰，確保我們得到的公鑰是這個網站給的，而不是中間人
        * 證書還是可能被中間人攻擊，此時就需要數字簽名
    * 數字簽名是由第三方CA提供
        * 數字簽名由CA用自己的私鑰鎖上數字證書經過一些程序製作
        * browser一開始就存有信任的CA公鑰可以解鎖(不會被中間人竄改)
        * browser由server收到數字證書(明文)及數字簽名(被CA用私鑰鎖住)，經過解密及運算後比對是否正確
        * 數字證書因為是明文，所以可能被中間人竄改，但是因為數字簽名是由CA用私鑰鎖住，所以不可能被竄改
    * CA
        * 因為CA算是公家機構，提供的內容不會假，所以可以拿來認證
        * server會跟CA購買數字證書及數字簽名，得到後就用這個來跟browser溝通，不會透過CA
        * 數字證書可以成功運作的一個關鍵點是，browser一開始就存有某些信任CA的公鑰，不用怕公鑰來源是中間人攻擊
        * 幾百幾千幾萬台幣(一年)的都有，看需求及可信度
    * 溝通流程 [參考資料1](https://www.jianshu.com/p/7158568e4867) [參考資料2](https://zhuanlan.zhihu.com/p/22142170)
        * TCP 三次握手
        * Client hello
            * 生成第一個random key
        * Server hello
            * 生成第二個random key
        * Certificate
            * server回覆數字證書
        * Server hello done
        * Certificate verify
        * Client key exchange
            * 生成第三個random key, 又叫Premaster key
            * 雙方利用三個random key產生出密鑰
        * Client發出: change cipher spec, encrypted handshaker message
            * 第一個是通知事件: 通知接下來要用密鑰溝通
            * 第二個是client finish訊息(第一個用密鑰加密的訊息)，若server能解開，表示協商成功
        * Server也發出: change cipher spec, encrypted handshaker message
        * Application data
            * 雙方開始用密鑰溝通
* 其他比較
    * HTTP over SSL over TCP
    * http 是用port 80 or 8080; https 是用port 443
    * https分為SSL(TLS前身),  TLS
    * http屬於application layer; SSL, TLS沒有明確的在哪一層(OSI, TCP/IP模型不能完全套用) [參考資料](https://www.jianshu.com/p/5ee027c51af0)

#### http與https封包分析
很多成大網站都是用http，此時只要前面改成https很多都可以，只是可能"你的連線不是私人連線"
1. 使用http登入成大email: 帳密直接明文傳輸(利用POST)
![](https://i.imgur.com/zfgFqMR.png)

2. 使用https登入成大email: 會先用TLS溝通，然後以加密的訊息傳遞
![](https://i.imgur.com/HtYZm4W.png)
    * 可以看到雙方一開始來往時會有一開始的雙方hello，還有證書確認，premaster傳送，最後的雙方確認
    * 前置作業都完成後，開始傳送所需資料，也就是最底下的圈圈那裏，Info顯示的是Application Data，也就是加密的部分
        * 加密的部份顯示的都是亂碼
    ![](https://i.imgur.com/J7jKQsH.png)
3. 我們來詳細的看2.的TLS溝通部分裡面包含的訊息
    * Client hello
        * 可以看到有random的產生
        * cipher suites: 這個的功用是告訴server，我可以接受那些形式的"密鑰交換算法-對稱加密算法-哈希算法"
        ![](https://i.imgur.com/4vFMkrN.png)
    * Server hello
        * 可以看到server也產生了一個random
        * 並且回覆要用哪一種cipher suite: TLS_RSA_WITH_AES_128_CBC_SHA (0x002f)
        ![](https://i.imgur.com/djdrqFZ.png)
    * Certificate(回覆證書)
        * 可以看到證書內容包括網域名稱以及publickey等資訊
    ![](https://i.imgur.com/BKdzUD2.png)
    * Client key exchange
        * 可以看到第三個random產生了，並且用公鑰加密傳給server
        * (三個random都產生後，可以製作出master key密鑰)
    ![](https://i.imgur.com/NhYaJQN.png)
    * client和server都發出: change cipher spec, encrypted handshaker message
    ![](https://i.imgur.com/xZLbQFq.png)
    * Application data
    ![](https://i.imgur.com/aHfdHLo.png)
#### 為甚麼有的https網站會出現不安全?
![](https://i.imgur.com/cMEHbBy.png)
1. 像這個是成大的一個網站，CA為TWCA Secure SSL Certificate Authority，但是這個CA不被Chrome認可，因此出現警告
2. CA以及數字證書的介紹在上面有講到，這裡就不多說
3. 一般會出現不安全的原因可能是:
    * 這是自行簽署的憑證，或由未認可的憑證授權單位所簽署
    * 憑證已損毀
    * 憑證已過期或尚未生效
    * 憑證的網域名與你要造訪的不一樣
5. 解決辦法: a.讓你的電腦認可這個CA b.按進階，繼續訪問網頁

收到封包後出現
![](https://i.imgur.com/1bD0q0o.png)





---
#### 參考資料
[彻底搞懂HTTPS的加密机制](https://zhuanlan.zhihu.com/p/43789231)  
[SSL/TLS协议运行机制的概述](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)
