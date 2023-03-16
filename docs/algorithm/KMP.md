---
draft: true
---

KMP 算法是要在一個字串 A 找一個子字串 B

## 暴力法

暴力法是從 A 的某一端的第一個字符開始跟 B 逐字符比對，如果不一樣了，就從該端的第二個字符開始跟 B 逐字符比對，直到找到一樣的

如果 A 的長度是 n，B 的長度是 m，時間複雜度是 O(n \* m)

```
s1 = "abcabcabe"
s2 = "abcabe"

def findIndex(s1, s2):
  i = 0
  j = 0

  while i < len(s1) and j < len(s2):
    if s1[i] == s2[j]:
      i += 1
      j += 1
    else:
      i = i - j + 1
      j = 0

  if j == len(s2):
    return i - j

  return -1

result = findIndex(s1, s2)

print(result)
```

## KMP 算法

這個算法改進暴力法的辦法是利用前一次逐字符比對的經驗去提高效率

經驗的意思是指前一次的逐字符比對雖然不能完全對上，但直到發生錯誤之前都是對的上的，就可以用這個正確的部分去找到下一次逐字符比對的起點

先看這個例子

```
i 012345678
  ✔✔✔✔✔✗
A abcabcabe
B abcabe
```

第一次的逐字符比對從 i = 0 開始比，一開始都對的上，直到 c 跟 e 對不上，結束這一輪

暴力法的第二輪就是從 i = 1 開始比，第一個字符就對不上了

```
i 012345678
   ✗
A abcabcabe
B  abcabe
```

KMP 算法是利用第一輪已經知道 B 字串中的 abcabe 中的 abcab 是對的上的，可以發現 ab 是重複的，那下一輪的逐字串比對可以從 A 中的第二個 ab 當起點來比，也沒有必要從頭(i=3)開始比，既然已經知道 ab 是重複的，ab 就不需要比了，從 c(i=5) 開始比就可以了，這個位置就是上一輪第一個對不上的位置

```
i 012345678

A abcabcabe
B    abcabe
```

看到這邊會疑惑兩個 ab 之間可能有別的可以當開頭的可能性? （後面會解答)

那要怎麼找到 ab 呢?

abcab 中挑選第二個 ab 作為下一輪的子字串起點是因為開頭是 ab，節尾也是 ab，但這種情況可能會有多個

如下

aaaa: a, aa, aaa, aaaa 都符合 prefix == suffix

```
i 0123456

  ???????

  ?????
    ?????
```

首先，們要找長度為第二長的，因為第一長的會是該字串本身，所以跳過

把這個字串稱為 Longest Proper Prefix Which Is Also Suffix，簡稱 LPS

如果我可以從字串中找到包含開頭的子字串跟包含結尾的子字串是相同的，如下

那我們就可以從 A 中的第二個 LPS 作為下一輪的起點

接下來回答前面提到的問題

兩個 LPS 之間可能有別的可以當開頭的可能性?

先看以下例子

```
i 0123456

  abababa

  a     a

  aba aba

  ababa
    ababa
```

如果不是選 ababa，而是選 a 或是 aba，下一輪的開始就會比較靠右(i=6,i=4)，會漏掉靠左成功的可能

也就是說，LPS 有要求長度要次長，suffix 為次長代表最靠左的起點，既然已經是最靠左了，那中間就不會有別的可能了

那要怎麼整理出 B 字串每一個字符的 LPS 呢?

並且要考慮時間複雜度要 O(m)

其實我們真正需要的是 LPS 的長度

為了做到 O(m)，我們要利用前一個字符的 LPS 長度，這樣就不需要回退

xzyabcabcabe
abcabe

    AAAC

LPS 01

LSP 的長度也會等於 LPS 的 index，因為 index 從 0

ABABA
00123
