參考網站 [使用readelf和objdump解析目标文件](https://www.jianshu.com/p/863b279c941e)  

### ELF
1. ELF文件一共有4种类型：Relocatable file、Executable file、Shared object file和Core Dump file  
2. Header中主要存放的是一些基本信息，通过Header中的信息，我们可以确定后面其他字段的大小和起始地址  
  $readelf -h 123 来进行对Header的解析  
3. Section部分主要存放的是机器指令代码和数据  
    * $readelf -S -W simple.o 对Section部分的解析  
    * .text(存放代码)
    * .data(存放全局静态变量和局部静态变量)
    * .bss(存未初始化的全局变量和局部静态变量) (.bss不占磁盘空间)  
    
### $objdump(得到每個段的大小、資料、存放位置、反解譯)
objdump displays information about one or more object files  

對section解析  
  * $objdump -h 123.o  
  
對.text解析(包括反解譯)  
  * $objdump -s -d 123.o  
  
知道各个变量的存放位置  
  * objdump -x -s -d 123.o
