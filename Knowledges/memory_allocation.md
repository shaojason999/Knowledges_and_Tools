# 記憶體分配問題
* function的位置固定，但是變數位置不一定
    * 前提是動態分配(ALSR)有打開，且PIE有設置(看gcc版本)(下面會介紹)
    ```$ echo 1 > /proc/sys/kernel/randomize_va_space``` or
    ```$ echo 2 > /proc/sys/kernel/randomize_va_space```
    * 如果ALSR為0則全固定
* 我寫了一個簡單的程式來測試，其中int跟char的宣告故意穿插
```c
#include <stdio.h>
  
int main()
{
      int a,aa,aaa;
      char e;
      int b;
      char c;
      printf("main: %p\nint:  %p %p %p\nchar: %p\nint:  %p\nchar: %p\n",&m    ain,&a,&aa,&aaa,&e,&b,&c);
}
```
* 執行結果
```
$ ./temp 
main: 0x400596
int:  0x7ffef43b2f18 0x7ffef43b2f1c 0x7ffef43b2f20
char: 0x7ffef43b2f16
int:  0x7ffef43b2f24
char: 0x7ffef43b2f17

$ ./temp 
main: 0x400596
int:  0x7ffd771f5c88 0x7ffd771f5c8c 0x7ffd771f5c90
char: 0x7ffd771f5c86
int:  0x7ffd771f5c94
char: 0x7ffd771f5c87
```
1. 可以發現main function的位置固定，其他變數每次執行位置都不一樣
2. 變數的位置分配可以看到是照(1)變數類型(2)宣告順序來分配
3. 這些記憶體位置都是虛擬的，不是實體位置
    * 所以即使同時開啟兩個一樣的程式，main位置固定，也不會有衝突
    * 有興趣可以去看看OS的書
4. 如果想要main也隨機的話可以這樣編譯
    ```gcc -o temp temp.c -fpie -pie```
    [Linux平台的ASLR机制](https://blog.csdn.net/Plus_RE/article/details/79199772)
    * 執行結果，所有位置都改變了
    ```
    $ ./temp2
    global: 0x56096aada048
    global1: 0x56096aada040
    main: 0x56096a8d97c0
    int:  0x7ffdefeabab8 0x7ffdefeababc 0x7ffdefeabac0
    char: 0x7ffdefeabab6
    int:  0x7ffdefeabac4
    char: 0x7ffdefeabab7
    
    $ ./temp2
    global: 0x5652d3cb6048
    global1: 0x5652d3cb6040
    main: 0x5652d3ab57c0
    int:  0x7ffd8cdae2e8 0x7ffd8cdae2ec 0x7ffd8cdae2f0
    char: 0x7ffd8cdae2e6
    int:  0x7ffd8cdae2f4
    char: 0x7ffd8cdae2e7
    ```
5. 有趣的是可以發現如果PIE有開啟(全部隨機)，ELF類型會變成shared object
    ```
    $ file temp
    temp: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=50b4f0d93bb45197e52acb6909fdba54e7a07162, not stripped
    
    $ file temp2
    temp2: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=2f1ac26bf1c29d607ef80de6ba95a3b486f5d5ea, not stripped
    ```
6. 總結來說
    * 記憶體位置都是虛擬位置
    * ALSR及PIE決定記憶體為固定或是隨機
    * ALSR並不負責數據段和代碼段的隨機化
        * 數據段（datasegment）通常是指用來存放程序中已初始化的全局變量的一塊內存區域。數據段屬於靜態內存分配
        * 代碼段（codesegment/textsegment）通常是指用來存放程序執行代碼的一塊內存區域。這部分區域的大小在程序運行前就已經確定
    * PIE負責隨機化，但是前提是ALSR需設為1或2
    * 基本上記憶體全隨機化會比較安全(不會輕易buffer overflow到某一個位置)
    * 參考資料: [Linux平台的ASLR机制](https://blog.csdn.net/Plus_RE/article/details/79199772)

![memory_layout](https://i.imgur.com/RPEL5aq.png)


### 參考資料
1. [Linux平台的ASLR机制](https://blog.csdn.net/Plus_RE/article/details/79199772)
2. [How is it that main function is always loaded at the same address whereas variables have different address most of the time?](https://stackoverflow.com/questions/3699845/how-is-it-that-main-function-is-always-loaded-at-the-same-address-whereas-variab)
