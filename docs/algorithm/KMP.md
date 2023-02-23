---
draft: true
---

KMP 算法是要在一個字串 A 找一個子字串 B

## 暴力法

暴力法是從 A 的某一端的第一個字符開始跟 B 逐個字符比對，如果不一樣了，就從該端的第二個字符開始跟 B 逐個字符比對，直到找到一樣的

如果 A 的長度是 n，B 的長度是 m，好的情況是逐字比對一開始就發現不一樣，就可以換到下一個字符再開始逐字比對，這情況的時間複雜度是 O(n + m)，最差的情況是逐字比對一開始都是對的，比到後面才發現不一樣，換到下一個字符再開始逐字比對，然後又是比到後面才發現不一樣，就會花很多時間，這情況的時間複雜度是 O(n \* m)

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

這個算法改進暴力法的辦法是利用前一次的逐步比對的經驗去提高效率
