## Some useful tool
1. 收尋字串 "pico"  
  $grep pico text.txt
2. 將結果輸出到某個檔案  
  $./abc>text.txt
3. 把亂把變成可讀字串(可搭配1.2.)  
  $strings text  

## pwn
### buffer overflow 1
使用工具: <b>$readelf</b>  
解決方法: 找出win的位置，透過overflow，將return address複寫成win的位置  
