### bss段：  
BSS段（bsssegment）通常是指用來存放程序中**未初始化的全局變量**的一塊內存區域。BSS是英文BlockStarted by Symbol的簡稱。BSS段屬於靜態內存分配。  

### data段：  
數據段（datasegment）通常是指用來存放程序中**已初始化的全局變量**的一塊內存區域。數據段屬於靜態內存分配。  

### text段：  
代碼段（codesegment/textsegment）通常是指用來存放程序執行代碼的一塊內存區域。這部分區域的大小在程序運行前就已經確定。  

### rodata段：  
存放C中的字符串和#define定義的常量  

### heap堆：  
堆是用於存放進程運行中被動態分配的內存段，它的大小並不固定，**可動態擴張或縮減**。**malloc**, **free**  

### stack棧：  
是用戶存放程序臨時創建的局部變量，(但不包括static聲明的變量，static意味著在數據段中存放變量)。  

[![memory_layout](https://github.com/shaojason999/picoCTF/blob/master/Knowledges/pictures/memoryLayoutC.jpg)](https://www.geeksforgeeks.org/memory-layout-of-c-program/)

[參考資料](https://read01.com/zh-tw/zeG4M7.html#.XIPIcygzY2w)
