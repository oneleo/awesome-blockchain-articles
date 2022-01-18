# 玩轉 Gnosis Safe 多簽錢包，以轉帳、連接 Compound 及 Snapshot 服務為例
  Play with the Gnosis Safe multi-signature wallet, take transation, connection to Compound and Snapshot services as examples


<details>
  <summary>請展開此以查看本文目錄：Contents</summary>
 </details>
    - [1、緣由]()
    - [2、簡介 Gnosis Safe 多簽錢包可應用的場景]()
            1、創建
        2、轉帳
        3、Compond
        4、snapshot
        5、合約說明
    - [前置作業（Bip39 * 5、Metamask、tETH * 4、]()
    - [創建一個 Gnosis Safe（Rinkeby Testnet）地址]()
    - [將 tETH、tNFT 轉]()
    - []()


## 緣由
  The reason

- 我們在 Ethereum 上使用錢包 Wallet 進行交易、與智能合約互動時，最怕遇到助憶詞（或由助憶詞產生的子私鑰）外洩或遺失，因為助憶詞代表著您對此區塊鏈上的資產存取權，外洩或遺失意謂著您將損失、失去您在區塊鏈上對映的所有資產！

- 為了避免單一帳戶助憶詞外洩或遺失，Gnosis 出品的 Gnosis Safe 多簽錢包就此因應而生！Gnosis Safe 允許公司或個人使用多個賬戶來對同一筆交易進行簽章，實現了多簽錢包資金的安全管理。此時就算其中一組帳戶的助憶詞遺失，我們還有其他帳戶可對其錢包內資產進行管理及遷移！

- 本文即教大家在 Ethereum Rinkeby Testnet 測試網上，透過創建一個 Gnosis Safe 多簽錢包，讓您使用這個錢包進行更安全的資產控管，那麼我們就開始吧！

![](./images/safe-001_safe.png)

## 2、簡介 Gnosis Safe 多簽錢包可應用的場景
  Application scenarios of Gnosis Safe multi-signature wallet

- 自 2017 年 Gnosis 創立以來，不斷開發基於區塊鏈上的資產管理解決方案，Gnosis 開發的多簽錢包 Gnosis Safe 是在 Ethereum 乙太坊上管理數位資產最為信賴的平臺，有許多 DAO 去中心化自治組織用來管理服務。

- Gnosis Safe 允許定義多簽錢包（實際上是智能合約錢包）的管理者列表，以及設置交易時需要的管理者確認閾值（例如：3 人中只需 2 人確認、5 人中只需 3 人確認），假設這筆轉帳收集了超過閾值數量的簽署確認，即可執行交易。

- 公司或團隊可更安全地將資金儲存在 Gnosis Safe 多簽錢包內，因為要求指定數量的管理者接受才能轉移資金。所以沒有一個管理者可以捲款跑路

- 公司或團隊可在大多數管理者的共識之下執行機敏交易

- 而個人可使用 Gnosis Safe 多簽錢包的多簽特性，來增加擁有助憶詞（或由助憶詞產生的子私鑰）的容錯程度，假設您遺失了其中一份助憶詞，您還可用剩下的兩份助憶詞來轉移資產

- 除了數位資產轉移外，Gnosis Safe 也內建多項 DAPPs 服務，如：Compound、Uniswap 等在這些應用上進行多簽管理；若所需的 DAPPs 不在內建列表，Gnosis Safe 亦可與支援 WalletConnect 開源協議的去中心化服務進行加密連線，如：Snapshot，讓多簽功能不中斷。

- 最後，Gnosis Safe 目前可支援 11 條不同的區塊鏈（含測試鏈），如：Ethereum、XDai、Polygon、Binance Smart Chain 等。

![](./images/safe-002_network.png)

## 前置作業

### 安裝 Metamask 錢包，並創建在 Rinkeby Testnet 乙太坊測試網使用的錢包

- 本文使用的 Rinkeby 測試鏈可以使用 Faucet 水龍頭服務來領取免費的測試用 ETH 乙太幣

- 在瀏覽器中安裝 [Metamask](https://metamask.io/) 錢包

  - [Microsoft Edge](https://www.microsoft.com/zh-tw/edge) 版 Metamask 下載位置：[https://microsoftedge.microsoft.com/addons/detail/metamask/ejbalbakoplchlghecdalmeeeajnimhm](https://microsoftedge.microsoft.com/addons/detail/metamask/ejbalbakoplchlghecdalmeeeajnimhm)
  - [Google Chrome](https://www.google.com/intl/zh-TW/chrome/) 版 Metamask 下載位置：[https://chrome.google.com/webstore/detail/metamask/nkbihfbeogaeaoehlefnkodbefgpgknn](https://chrome.google.com/webstore/detail/metamask/nkbihfbeogaeaoehlefnkodbefgpgknn)
  - [Mozilla Firefox](https://www.mozilla.org/zh-TW/) 版 Metamask 下載位置：[https://addons.mozilla.org/zh-TW/firefox/addon/ether-metamask/](https://addons.mozilla.org/zh-TW/firefox/addon/ether-metamask/)

![](./images/safe-003_metamask.png)

- 點選瀏覽器右上角的【Metamask 符號】→【開始使用】→【匯入錢包】→ 點選【No Thanks】捥拒收集匿名資訊

![](./images/safe-004_metamask.png)

- 請輸入 2 次「Metamask 密碼」→ 勾選【I have read and agree to the 使用條款】→【建立】→ 看完介紹影片後點選【下一頁】→ 點選【點選顯示助憶詞】，並將顯示的 12 組助憶詞記錄下來（在這邊以：already across together hood nominee field more crush hen flee example cannon 助憶詞為例）→ 點選【下一頁】

  - 注意：實務上，助憶詞才是最重要、具有資產轉移的權限，這邊所輸入的密碼只是 Metamask 簽章前的臨時保護措施，關鍵還是助憶詞上的保管、勿輕易外洩！

![](./images/safe-005_metamask.png)

- 請【依序點選』上一步記下的助憶詞 →【確認】→ 點選【都完成了】完成 Metamask 新錢包的創建

![](./images/safe-006_metamask.png)

- 因為預設連線至 Ethereum Mainnet 乙太坊主網，餘額顯示為 0，所以要改為連線至 Rinkeby Testnet 乙太坊測試網路。請點選右上角【帳號】圖示 →【設定】→【進階】→【勾選】「Show test networks」以顯示測試網路

![](./images/safe-007_network.png)

- 點選右上角的【Metamask 符號】→ 點選上方【網路圖示】→【Rinkeby】即可看到已切換至 Rinkeby 網路

![](./images/safe-008_metamask.png)

### 透過 BIP-39 協定，使用上一步的助憶詞產生多個 account 錢包

- 比特幣 BIP-39 協定允許使用者透過調整 HD Wallet 的 Path Tree，用同一組助憶詞來產生多組 account 錢包，詳情可參考另一篇文章：「[詳解 HD Wallet、BIP-0032、BIP-0039、BIP-0043 及 BIP-0044](https://github.com/oneleo/awesome-blockchain-articles/tree/main/Explain%20the%20HD%20Wallet%E3%80%81BIP-0032%E3%80%81BIP-0039%E3%80%81BIP-0043%20%26%20BIP-0044%20in%20detail)」