### bss段：  
* BSS段（bsssegment）通常是指用來存放程序中
    * 未初始化和初始化為 0 的全局變量
* BSS是英文Block Started by Symbol的簡稱
* 不占文件空間，只占運行時的內存空間
* 作業系統加載時可能會初始化為 0

### data段：  
* 數據段（datasegment）通常是指用來存放程序中
    * 已初始化(非0)的全局變量
    * static 變數(不見得是全局變量)
        * static 在 local 宣告時是區域變數，但是生命週期卻是跟著整個程式(多次在區域裡面重複使用)
        * static 在 global 宣告時，只有這個檔案(.o)可以看到，連結器連結多個 .o 後，彼此之間看不到
            * 一般的 global 變數是多的檔案間可以看到，也就是說整個 .exe 都可以使用
* 程式開始執行前就已經占空間了
    * 占文件空間以及執行時的內存空間

### rodata段： (read only data)  
* const 常量放在這(但有時放在 text段)
* 多個進程(process)間共享，提高內存利用率
* 在有的嵌入式系統中，無須加載到 RAM 中

### text段：  
代碼段（codesegment/textsegment）通常是指用來存放程序執行代碼與部分常量的一塊內存區域。這部分區域的大小在程序運行前就已經確定

### heap堆：  
堆是用於存放進程運行中被動態分配的內存段，它的大小並不固定，**可動態擴張或縮減**。**malloc**, **realloc**, **free**

### stack棧：  
是用戶存放程序臨時創建的局部變量，(但不包括static聲明的變量，static意味著在數據段中存放變量)

[![memory_layout](https://github.com/shaojason999/picoCTF/blob/master/Knowledges/pictures/memoryLayoutC.jpg)](https://www.geeksforgeeks.org/memory-layout-of-c-program/)

---
#### 參考資料: [1](https://read01.com/zh-tw/zeG4M7.html#.XIPIcygzY2w), [2](http://hatsukiakio.blogspot.com/2009/04/c-static.html)
