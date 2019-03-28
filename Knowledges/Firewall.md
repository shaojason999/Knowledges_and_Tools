## 防火牆
分析與過濾進出我們管理之網域的封包  
1. 可以安裝在個人主機上，也可以在router等網路設備上
2. 防火牆分為硬體與軟體
    * 軟體: 例如Netfilter, TCP Wrappers
3. 限制某些服務的存取來源，例如  
    * 限制FTP只能在子網中使用，不對外(Internet)開放  
    * 僅接受www的要求，其他服務一律關閉  
    * 對內限制某個MAC的行為
4. Netfilter
    * 分析網路封包header(主要是OSI的2,3,4層)，決定是否阻擋
    * 封包內容修改等功能
    * Linux核心內建，因此效率非常高
    * Netfilter提供[iptables](iptables.md)軟體做為防火牆封包過濾的指令
5. TCP Wrappers
    * 爭對程式去控管
    * 透過 /etc/hosts.allow, /etc/hosts.deny來管理
    * 並非所有程式都受控制(受限於支援上面兩個的程式)
    * 舉例對支援的程式(rsync)依IP位置來做限制
    ```
    [root@www ~]# vim /etc/hosts.allow
    ALL: 127.0.0.1
    rsync: 192.168.1.0/255.255.255.0 10.0.0.100

    [root@www ~]# vim /etc/hosts.deny
    rsync: ALL
    ```  
    * 比對順序: 先進到allow，若有比對到則通過。若沒有則比對deny，若有比對到，則拒絕服務。若沒有，則比對都失敗，封包通過  
