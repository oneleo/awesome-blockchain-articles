# 詳解 HD Wallet、BIP-0032、BIP-0039、BIP-0043 及 BIP-0044
## 加密貨幣錢包（Cryptocurrency Wallet）

1. 在加密貨幣的世界中，錢包並非儲存加密貨幣的地方。

2. 反倒是錢包的主要功能是一個儲存私鑰（Private Key）的工具，私鑰是一串很長的英文字母、數字組成的字串（通常是 64 個 16 進制位元 = 256 Bits = 32 Bytes 所組成），而這條很長的字串讓您有權力把自己的加密貨幣傳送給別人。

3. 錢包可在使用者產生交易（Transaction）時使用私鑰將交易簽章（Digital Signature），而簽章的目的除了確認使用者為貨幣擁有者外，還可確保在交易完成後不可否認（Non-Repudiation）此交易；而儲存加密貨幣的地方則是負責維護區塊鏈（Blockchain）的礦工節點內。

4. 加密貨幣錢包形式多樣，分為離線的冷錢包（Cold Wallet）、在線的熱錢包（Hot Wallet）、印在紙上的紙錢包（Paper Wallet）… 等。無論何種形式的錢包，錢包的開發商均需要有一個共同遵循的規範（例如：私鑰為多少位元、如何依據一個亂數產生私鑰等）。

--------------------------------------------------

## 比特幣改進提案（Bitcoin Improvement Proposals，BIP）

1. BIP 全名是 Bitcoin Improvement Proposals，是開發者向比特幣社群提出比特幣 Bitcoin 新功能或改進建議的技術設計文件。

2. 根據 [BIP-0001](https://github.com/bitcoin/bips/blob/master/bip-0001.mediawiki) 改進提案，BIP 總共分為：

- 描述系統面更動的標準類 BIP（Standards Track BIP）

- 描述系統指導資訊的資訊類 BIP（Informational BIP）

- 描述建議更動流程的程序類 BIP（Process BIP）

--------------------------------------------------

## 分層確定性錢包（Hierarchical Determinstic Wallet，HD Wallet）

- HD Wallet 被 BIP-0032、BIP-0039、BIP-0043、BIP-0044 改進提案所共同定義，包含了錢包的設計動機、理念、實作方式等。

### [BIP-0032](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 改進提案：

- BIP-0032 是 HD Wallet 的核心提案，在系統使用 [SECP256K1](http://www.secg.org/sec2-v2.pdf) 橢圓曲線加密演算法前提下，我們可用熵（Entropy）函數產生隨機的 32 個 16 進制字元（128 Bits）S，

`S = Entropy_128_Bits`

- 將 S 字串透過雜湊演算法 [HMAC-SHA512](https://tools.ietf.org/html/rfc4231) 產生一組 64 個 16 進制字元（256 Bits），我們將（左半邊的）256 Bits 字串作為父擴展私鑰（Parent Extended Private Key）m；另一組（右半邊的）256 Bits 字串作為下一級的鏈碼（Chain Code）c。而 m 可推導成父擴展公鑰（Parent Extended Public Key）M。

`m = Left（HMAC_SHA512（S）, 256）`

`c = Right（HMAC_SHA512（S）, 256）`

`M = SECP256K1（m）`

- 使用子私鑰求導（Child Private Key Derivation，CKDpri）函數，我們可以透過父擴展私鑰來產生出 index = i 子私鑰（Private Child Key）。

`Child Private Key（Index = i）`
`= m / i`
`= CKDpri（m, i）`
`= Left（HMAC_SHA512（c, M, i）, 256）⊕ m`

```
m / 0 = CKDpri（m, 0）
m / 1 = CKDpri（m, 1）
m / 2 = CKDpri（m, 2）
…
m / i = CKDpri（m, i）
```

![](https://i.imgur.com/cAfONYE.png)

- 使用增強子私鑰求導（Child Hardened Key Derivation，HKD）函數，我們可以透過父擴展私鑰來產生出 index = i' 子私鑰（Private Child Key）。

`Child Private Key（Index = i'）`
`= m / i'`
`= HKD（m, i）`
`= Left（HMAC_SHA512（c, m, i）, 256）⊕ m`

```
m / 0' = HKD（m, 0）
m / 1' = HKD（m, 1）
m / 2' = HKD（m, 2）
…
m / i' = HKD（m, i）
```

![](https://i.imgur.com/BvSt5KO.png)

- 使用子公鑰求導（Child Public Key Derivation，CKDpub）函數，我們可以透過父擴展公鑰來產生出子公鑰（Public Child Key）。

`Child Public Key`
`= M / i`
`= CKDpub（M, i）`
`= Left（HMAC_SHA512（c, M, i）, 256）⊕ M`

```
M / 0 = CKDpub（M, 0）
M / 1 = CKDpub（M, 1）
M / 2 = CKDpub（M, 2）
…
M / i = CKDpub（M, i）
```

![](https://i.imgur.com/9SRXYRz.png)

- BIP-0032 改進提案的好處是，我們只要備份、轉移 S，即可透過鑰匙樹（Key Tree）在其他相容 BIP-0032 的裝置上還原私鑰、公鑰及地址。

--------------------------------------------------

### [BIP-0039](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) 改進提案：

- 有鑑於 BIP-0032 的 S 不好記憶，BIP-0039 提出 S 可以透過 12 組英文單字助記詞（Mnemonic Code）來產生。

- 熵（Entropy）函數產生隨機的 32 個 16 進制字元（128 Bits ~ 256 Bits）ENT（初始熵長度，Initial Entropy Length），再取 ENT 的位元數除以 32 得到 CS（校驗和長度，Checksum Length）。

- 註：`ENT = Entropy（）= 128 + 32 * N Bits`，`N = 0 ~ 4`，這邊採 Entropy 128 Bits／12 Words 來作計算。

`ENT = Entropy_128_Bits`

`CS = Length（ENT）/ 32`

- 在 ENT 右側串接 SHA256（ENT）左邊的 CS 個字元。

`X = ENT *（2 ^ CS）+ Left（SHA256（ENT）, CS）`

- 將 X 每 11 Bits 進行切割生成的助記詞（Generated Mnemonic Sentence，MS）

`MS[] = Split（X, 11）`

- 因為 ENT 位元數範圍為（128 + 32 * N）Bits，N = 0 ~ 4，下表為不同 ENT 位元數時會產生助記詞 MS 數量。

| Length（ENT） | Length（CS） | Length（ENT）+<br>Length（CS） | Length（MS） |
|:---------:|:--------:|:--------:|:------:|
| 128 | 4 | 132 | 12 |
| 160 | 5 | 165 | 15 |
| 192 | 6 | 198 | 18 |
| 224 | 7 | 231 | 21 |
| 256 | 8 | 264 | 24 |

- 以太坊 MetaMask 錢包是使用當 ENT 為 128 位元時產生的 12 組 MS，透過查表法，我們可得對映的 12 組英文單字（或是中文、日文、韓文等）

- 註：對映表可參考[此連結](https://github.com/bitcoin/bips/blob/master/bip-0039/bip-0039-wordlists.md)。

- 可以將 12 ~ 24 組的 MS + [Salt](https://en.wikipedia.org/wiki/Salt_(cryptography))（密碼學中加強雜湊亂度用）經過 2048 次的 HMAC-SHA512 計算後取得 BIP-0032 所使用的 S。

`S =（HMAC_SHA512 ^ 2048）（MS + Salt）`

![](https://i.imgur.com/FRCVzVz.png)

--------------------------------------------------

### [BIP-0043](https://github.com/bitcoin/bips/blob/master/bip-0043.mediawiki) 改進提案：

- 根據 BIP-0032 改進提案，將 Key Tree 的第一級定義為宗旨（Purpose），Purpose 值與 BIP-00XX 改進提按的編號相等，例如 BIP-0044 中的 Purpose = 44。

`BIP-0044 改進提案中的中的 Key Tree`
`= m / 44'`
`= HKD（m, 44）`
`= Left（HMAC_SHA512（c, m, 44））⊕ m`

### [BIP-0044](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) 改進提案：

- 根據 BIP-0032、BIP-0043 的改進提案，將 Key Tree 的
    - 第一級定義為宗旨（Purpose）
    - 第二級定義為代幣類型（Coin Type）
    - 第三級定義為帳戶（Account）
    - 第四級定義為鏈的變化（Change）
    - 第五級定義為地址索引（Index）

`BIP-0044 的 Key Tree`
`= CKDpub（CKDpub（HKD（HKD（HKD（m, i）, j）, n）, o）, p）`
`= m / i' / j' / n' / o / p`

`i = Purpose，i = 44 表示 BIP-0044。`
`j = Coin Type，j = 0 表示比特幣；j = 1 表示比特幣測試鏈；j = 60 表示以太坊。`
`n = Account，n = 0 表示第一個帳戶；n = 1 表示第二個帳戶。`
`o = Change，o = 0 表示用於收款的外部鏈（External Chain）；o = 1 表示用於更改地址的內部鏈（Internal Chain）。`
`p = Index，p = 0 表示第一組地址；p = 1 表示第二組地址。`

![](https://i.imgur.com/hlMvHTw.png)


--------------------------------------------------

## 結論

- 本篇介紹了什麼是 HD Wallet，以及 BIP-0032、BIP-0039、BIP-0043、BIP-0044 改進提案，而我們可以根據 BIP-0044 簡易的得知：若我們想取得以太坊的第一組帳號中用於收款之第一組地址 Key Tree 即為 `m / 44' / 60' / 0' / 0 / 0`。

--------------------------------------------------

## 參考資料：

- 【加密貨幣錢包】從 BIP32、BIP39、BIP44 到 Ethereum HD Wallet
https://medium.com/taipei-ethereum-meetup/%E8%99%9B%E6%93%AC%E8%B2%A8%E5%B9%A3%E9%8C%A2%E5%8C%85-%E5%BE%9E-bip32-bip39-bip44-%E5%88%B0-ethereum-hd-%EF%BD%97allet-a40b1c87c1f7

- 數字貨幣錢包 - 助記詞 及 HD 錢包密鑰原理
https://huangwenwei.com/blogs/bip32-bip39-and-hd-wallet

- 分層確定性錢包 HD Wallet 介紹
http://bigshark.club/2017/10/20/intr-hd-wallet/

- 區塊鏈錢包之BIP32, BIP39, BIP44
https://blog.csdn.net/qq634416025/article/details/79686015

- BIP32, BIP39 和BIP44 有什麼區別？
http://8btc.com/thread-35670-1-1.html

- 區塊鏈錢包之BIP32, BIP39, BIP44
https://juejin.im/post/5ab70c146fb9a028d664203f

- Working with Bitcoin HD wallets II: Deriving public keys
https://sevdev.hu/posts/2017-01-16-working-with-bitcoin-hd-wallets-ii.html

- What are BIP32, BIP39 and BIP44 ?
https://bitcointalk.org/index.php?topic=644755.0

- Bitcoin Improvement Proposals (BIPs)
https://ithelp.ithome.com.tw/articles/10201358

- How long to hack an address that is used to send BTC multiple times?
https://bitcointalk.org/index.php?topic=2669689.0

--------------------------------------------------

（歡迎社群分享。但引用至政大【以太坊原理與應用開發】課程中的[作業一](https://hackmd.io/wApWgkUpR8WJEaLNbYz5Kw)請註明來自 [oneleo Steemit](https://steemit.com/@oneleo)，及附上原文連結：[詳解 HD Wallet、BIP-0032、BIP-0039、BIP-0043 及 BIP-0044](https://steemit.com/blockchain/@oneleo/hd-wallet-bip-0032-bip-0039-bip-0043-bip-0044)）

--------------------------------------------------

Donate Cardano ADA：
DdzFFzCqrhsup2Q4nnhKJJZ5BRuPkYUSPqDJn72t2dtHtVqsz5kQQmopMQR16Sv9qS5NC4w8Kv5P8XrDH2n2FD2akxtrntjc8hbgAmTz