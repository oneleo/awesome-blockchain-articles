# 橢圓曲線加密演算法 ECDSA 與 RFC6979 改進提案

橢圓曲線加密演算法（Elliptic Curve Digital Signature Algorithm，ECDSA）是比特幣、以太坊區塊鏈所使用的非對稱式金鑰加密技術，

可輕易讓貨幣持有者透過私鑰 Private Key（變數：d_A）對資訊進行簽章（Digital Signature），讓所有人使用 Public Key（變數：d, e, p, n, G, Q_A）來進行驗證（或是反向進行秘密傳遞）。

然而，早期的 ECDSA 演算法中，有一個僅產生一次性使用的臨時簽名變數 k（Ephemeral Key）

因不夠隨機，又或是忽略了其重要性（如每次收發訊息都使用相同的 k 值），導致駭客可透過反推方式求得 k 值，

一旦 k 值遭洩，持幣者的 Private Key 也可輕易的被回推計算出來。

像是早期 [Sony PS3](https://www.bbc.com/news/technology-12116051) 就是每次都使用相同的 k 值，導致遊戲機遭人破解。

--------------------------------------------------

## 1、流程

而這篇文章，將會簡介什麼是 ECDSA，以及它是如何產生非對稱式金鑰

接著再介紹如何使用 Private Key、Public Key 進行簽章與檢驗

以及展示透過 k 值取得 Private Key 推導過程

最後再說明 RFC6979 針對產生 k 值的改進提案

--------------------------------------------------

## 2、橢圓曲線

橢圓曲線一般式如下：

![](https://i.imgur.com/qsaSpQo.png)

而在 ECDSA 內所使用的是特殊規範的橢圓曲線：

![](https://i.imgur.com/C8nEqTT.png)

條件的設定目的是在為讓  x^3 + dx + e = 0 方程式的 x 能夠存在三個不同的（實數或複數）解好處是 ECDSA 在使用此特殊規範的曲線，總是可以同時找到 P、Q、R 三個點，並且這三個點又符合交換群（Abelian Commutative Group）的特性

--------------------------------------------------

## 3、交換群

什麼是交換群？就是這個群內的所有元素符合下面 5 條規範

以現實世界來說，其實就是我們習以為常的「乘法運算」、「加法運算」，像是「2 * 1 = 1 * 2」、「1 +（-1）=（-1）+ 1」等

而在線性代數的世界裡，我們是可以自己定義「運算」的，像是「P 點 ＊ Q 點 = P 點 ＊ Q 點」。

![](https://i.imgur.com/BYqm5Oa.png)

在這邊，我們也定義特殊規範的橢圓曲線，它的每一個點都要符合交換群的特性，我們定義一下這個群的符號

![](https://i.imgur.com/b4TmmLc.png)

而交換群的特性，下面僅列出 ECDSA 會用到的部份

![](https://i.imgur.com/IPRK4Pr.png)

--------------------------------------------------

## 4、點的加法運算

我們如何進行 ECDSA 點的加法運算呢？因為 ECDSA 一定會找到三個點符合 P + Q +（-R）= O

我們透過兩種不同的橢圓曲線，分別將 P、Q、R 三點繪畫在座標上，就會很清楚知道它們之間的關係

### 考慮 G =（E（d, e）, +）=（E（-1, 0）, +）的橢圓曲線

![](https://i.imgur.com/FEhX4Ng.png)

當 P ≠ Q 時：

![](https://cdn.steemitimages.com/DQmc5J24tchDjuWzxa9kds7og3ACUFzUNw3ogDrVZAMuV6C/image.png)

當 P = Q 時：

![](https://cdn.steemitimages.com/DQmRLgodQwu5shxypAEm642r5E9BXKhau8Benv8U2jh6NF2/image.png)

### 考慮 G =（E（d, e）, +）=（E（1, 1）, +）的橢圓曲線

![](https://i.imgur.com/NxR2GxV.png)

當 P ≠ Q 時：

![](https://cdn.steemitimages.com/DQmRfuc2MmDNX8QfGVB88oDn9pq4NzqhnhaeFpoBU3WjPJ4/image.png)

當 P = Q 時：

![](https://cdn.steemitimages.com/DQmSCAkGWDgRegTd8i5ShEXsekeHQcJcMC7WjBvj4Hfsmar/image.png)

--------------------------------------------------

## 5、已知 R = P + Q，求 R

因為 P、Q、（-R）三點共線，所以我們可以透過求 P、Q 斜率，計算出 R 點。

![](https://i.imgur.com/uyK8mSX.png)

--------------------------------------------------

## 6、點的乘法運算

那我們如何進行 ECDSA 點的乘法運算呢？可以視做連續的加法來比照辦理就好

![](https://i.imgur.com/D6yZFPx.png)

--------------------------------------------------

## 7、質數有限體

雖然在橢圓曲線上能夠取得無限多種不同的 P 點及 Q 點，但我們在使用 ECDSA 時並不會無限制的擴張，所以這時會將質數有限體 GF（p）的特性帶入，符號如下：

![](https://i.imgur.com/K1DdUeR.png)

而符合質數有限體的橢圓曲線，符號如下：

![](https://i.imgur.com/4tjbPvq.png)

使得 ECDSA 用的橢圓曲線會有如下特性：

![](https://i.imgur.com/21zuwmX.png)

--------------------------------------------------

## 8、反元素測試

有點混亂了嗎？沒關係，我們來舉個例子吧！

我們考慮使用 d = 1，e = 1，p = 13 的橢圓曲線如下

註：因為所有的變數會被限制在 13 - 1 = 12 內，所以均要在計算完畢後進行取餘（mod）運算，以限制範圍

![](https://i.imgur.com/RYNyLiR.png)

來小小測試一下在橢圓曲線上的（1, 4）這一個點，是符合上述條件的

![](https://i.imgur.com/PKqQz5V.png)

有了限制條件，我們就可以列舉出所有符合 d = 1，e = 1，p = 13 的橢圓曲線的點，如下：

![](https://i.imgur.com/OuqNwOJ.png)

還記得群的概念嗎？所有的元素都會有一個反元素

![](https://i.imgur.com/shKGJ0O.png)

那我們就來測試看看是否所有的點都有符合這個特性，找了一個點（4, 2）來進行測試

![](https://i.imgur.com/JrRlUcW.png)

可以發現（4, 11）這個點也在 d = 1，e = 1，p = 13 的橢圓曲線內，得以證明確實有符合群的各大特性，所以經過計算，我們就可以列出一個反元素速查表：

| P       | -P       |
| :-----: | :------: |
| (0, 1)  | (0, 12)  |
| (1, 4)  | (1, 9)   |
| (4, 2)  | (4, 11)  |
| (5, 1)  | (5, 12)  |
| (7, 0)  | (7, 0)   |
| (8, 1)  | (8, 12)  |
| (10, 6) | (10, 7)  |
| (11, 2) | (11, 11) |

--------------------------------------------------

## 9、加法測試，已知 R = P + Q，求 R

接下來就可以依樣畫葫蘆，透過 P + Q 來計算 R 點了

假設 P 點為（4, 2），Q 點為（10, 6）

![](https://i.imgur.com/gFygDEV.png)

這邊會讓人不知道如何進行的是分數的取餘計算，下方為計算方法：

![](https://i.imgur.com/qP3hTTo.png)

接下來繼續計算 R 點

![](https://i.imgur.com/mSU0e6x.png)

計算出來的 R 點也確實在 d = 1，e = 1，p = 13 的橢圓曲線內，同樣符合質數有限體的規範。

![](https://i.imgur.com/IiRVRkW.png)

--------------------------------------------------

## 10、乘法測試

接下來要介紹點的乘法，上面有提到點的乘法可以看作是連續的點的加法，在這邊同樣以 P 點為（4, 2）來做計算

![](https://i.imgur.com/EFghfUM.png)

首先計算（4, 2）+（4, 2）

![](https://i.imgur.com/zwfT0TI.png)

我們可以得到 （4, 2）+（4, 2）=（8, 1），仍在體的規範內，再來我們計算（8, 1）+（4, 2）

![](https://i.imgur.com/sDnghJ5.png)

最後我們得到（4, 2）+（4, 2）+（4, 2）的答案為（10, 6），證實整個 d = 1，e = 1，p = 13 的橢圓曲線內的點都在體的規範內

![](https://i.imgur.com/25i2K0U.png)

--------------------------------------------------

## 11、數位簽章及驗章

在我們了解了橢圓曲線的概論後

我們接著就要透過橢圓曲線來進行資訊簽章，以下是概念圖（截取至[維基百科](https://zh.wikipedia.org/wiki/%E6%95%B8%E4%BD%8D%E7%B0%BD%E7%AB%A0)）：

![](https://cdn.steemitimages.com/DQmQweTHw1KuX32xXi1EbAAjiFqNKpas3nnPduxB2xsLMw7/Digital_Signature_diagram_zh-CN.png)

首先我們要先透過橢圓曲線及私鑰來建立公鑰

![](https://i.imgur.com/WEQszvR.png)

透過橢圓曲線、私鑰來進行簽章

![](https://i.imgur.com/xd7IvwD.png)

透過橢圓曲線、公鑰來進行驗章

![](https://i.imgur.com/IwbFt4L.png)

以下證明為何 r = x_c 就代表證明驗章通過

![](https://i.imgur.com/4mivu0V.png)

--------------------------------------------------

## 12、Ephemeral Key k 的重要性

為什麼 Ephemeral Key k 那麼重要？假設我們每次在進行簽章時都使用相同的 k 值，或是選擇一個沒有那麼隨機產生出來的 k 值，導致駭客可透過反推方式求得 k 值，
一旦 k 值遭洩，持幣者的 Private Key 也可輕易的被回推計算出來。

![](https://i.imgur.com/CSQHZML.png)

所以這就是為什麼 RFC6979 提出我們需要有較佳的 k 值選法，以下是最簡式，
使用私鑰、每次都不一樣的資訊來進行 k 值選定，以確保 k 值足夠隨機

![](https://i.imgur.com/yyR9usO.png)

--------------------------------------------------

## 13、結語

經過了一大串的說明，我們不論在離線的冷錢包（Cold Wallet）、在線的熱錢包（Hot Wallet）、印在紙上的紙錢包（Paper Wallet）…等選擇上

都一定要選用符合 RFC6979 提案的 k 值選定，以確保私鑰不會遭人破解

--------------------------------------------------

## 14、參考資料

- [Request for Comments 6979](https://tools.ietf.org/html/rfc6979)

- [Request for Comments 6979 流程](https://tools.ietf.org/html/rfc6979#section-3.2)

- [Elliptic Curve Cryptography in Practice](https://eprint.iacr.org/2013/734.pdf)

- [RFC6979 講解：分分鐘搞懂 RFC6979](http://www.wanbizu.com/baike/201412083991.html)

- [DSA簽名算法筆記](https://my.oschina.net/u/1382972/blog/330657)

- [密碼貨幣與區塊鏈原理](https://www.math.sinica.edu.tw/www/file_upload/summer/crypt2017/data/2017/%E4%B8%8A%E8%AA%B2%E8%AC%9B%E7%BE%A9/[20170726][%E5%AF%86%E7%A2%BC%E8%B2%A8%E5%B9%A3%E8%88%87%E5%8D%80%E5%A1%8A%E9%8F%88%E5%8E%9F%E7%90%86].pdf)

- [Elliptic Curve Digital Signature Algorithm](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)

- [Digital Signature Algorithm](https://en.wikipedia.org/wiki/Digital_Signature_Algorithm)

- [Source code for ecpy.ecrand](https://ec-python.readthedocs.io/en/latest/_modules/ecpy/ecrand.html)

- [cslashm/ECPy/src/ecpy/ecrand.py](https://github.com/cslashm/ECPy/blob/master/src/ecpy/ecrand.py)

- [pybitcointools源碼分析之RFC6979](https://www.jianshu.com/p/0c27e5b51cdd)

- [進一步探討比特幣簽名中的隨機風險](https://www.8btc.com/article/36023)

- [iPhone hacker publishes secret Sony PlayStation 3 key](https://www.bbc.com/news/technology-12116051)

- [密碼學相關概念](https://www.jianshu.com/p/15fc08422724)

- [《精通比特幣》第二版 區塊鏈研究社 雲天明聯合出品](https://gittobook.org/books/192/MasterBitcoin2CN)

- [現代密碼學實踐指南](http://gad.qq.com/article/detail/12527)

- [Tool - Elliptic Curve Points](https://www.desmos.com/calculator)

- [Book - 網路安全與密碼學概論](https://www.books.com.tw/products/0010638686)

- [Book - 密碼編碼學與網絡安全：原理與實踐](https://www.books.com.tw/products/CN11523094)

- [Book - 近代密碼學與其應用](https://www.books.com.tw/products/0010303599)

--------------------------------------------------

（歡迎社群分享。但引用至政大【以太坊原理與應用開發】課程中的[作業一](https://hackmd.io/wApWgkUpR8WJEaLNbYz5Kw)請註明來自 [oneleo Steemit](https://steemit.com/@oneleo)，及附上原文連結：[橢圓曲線加密演算法 ECDSA 與 RFC6979 改進提案](https://steemit.com/cryptography/@oneleo/ecdsa-rfc6979)）

--------------------------------------------------

Donate Cardano ADA：
DdzFFzCqrhsup2Q4nnhKJJZ5BRuPkYUSPqDJn72t2dtHtVqsz5kQQmopMQR16Sv9qS5NC4w8Kv5P8XrDH2n2FD2akxtrntjc8hbgAmTz