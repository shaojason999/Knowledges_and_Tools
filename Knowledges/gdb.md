### 編譯時要打 -g  
$gcc -g -o 123 123.c  

### 開始degub  
$gdb 123  

### 設中斷點:  
可以先看設在哪一行比較好  
(gdb) list 0  

(gdb) break 10  

### 執行  
(gdb) r  
一行一行執行:  
(gdb) step/n(n不會進到function)  

### 顯示變數值和位置以及函式位置  
(gdb) print a  =  (gdb) p a  
(gdb) print &a  
(gdb) print $rax (register)  


### 顯示register  
info registers (info all-registers)  

### info使用方式  
(gdb) info  (相當於 --help)  

## 參考資料  
[1][GDB 使用教學](https://henrybear327.gitbooks.io/gitbook_tutorial/content/Linux/GDB/index.html)  
[2][一個簡單的 gdb 使用範例](http://puremonkey2010.blogspot.com/2010/07/gdb-gdb.html)  
