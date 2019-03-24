1. 霍夫曼編碼是一種用於無失真資料壓縮的演算法  
2. 出現機率高的字母使用較短的編碼，反之出現機率低的則使用較長的編碼  
    * 例如e很常出現，z很少出現，則e的編碼可能就是0，z的編碼可能是1001  
    * 不是每個字母是一個byte  
3. 編碼方式可以透過霍夫曼樹取得  
  a. 最小兩兩相加得到新的節點  
  b. 最後從根結點往下，右節點給1，左節點給0  
  過程:  
  ![gif](https://github.com/shaojason999/Knowledges_and_Tools/blob/master/Knowledges/pictures/Huffman_algorithm.gif)  
  結果:  
  ![jpg](https://github.com/shaojason999/Knowledges_and_Tools/blob/master/Knowledges/pictures/TABLE8.JPG)  
4. 特別注意: 因為每個字母都是終端節點，路徑唯一，所以不會有prefix相同，比如說假設t是101，則不會有一個字母是10100，也不會有10  
5. 解碼: 其中一個方法是利用5的特性，就可以把壓縮後的10101010010010101解碼回字母，只要比對到，就換回字母。當然，取得檔案內容時也要包括霍夫曼樹才能解碼  

## 參考資料
[wikipedia](https://zh.wikipedia.org/wiki/%E9%9C%8D%E5%A4%AB%E6%9B%BC%E7%BC%96%E7%A0%81)  
[演算法筆記](http://www.csie.ntnu.edu.tw/~u91029/Compression.html#7)  
