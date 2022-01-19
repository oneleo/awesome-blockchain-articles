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

### 透過 BIP-39 協定，使用上一步的助憶詞產生多個 account 錢包，以模擬多簽行為

- 比特幣 BIP-39 協定允許使用者透過調整 HD Wallet 的 Derivation Path，用同一組助憶詞來產生多組 Account 錢包，詳情可參考另一篇文章：「[詳解 HD Wallet、BIP-0032、BIP-0039、BIP-0043 及 BIP-0044](https://github.com/oneleo/awesome-blockchain-articles/tree/main/Explain%20the%20HD%20Wallet%E3%80%81BIP-0032%E3%80%81BIP-0039%E3%80%81BIP-0043%20%26%20BIP-0044%20in%20detail)」

- 將上一步所複製的助憶詞，貼在開源 [iancoleman/bip39](https://iancoleman.io/bip39/) 專案的「BIP39 Mnemonic」【欄位】→ 並將「Coin」改為【ETH - Ethereum】區塊鏈

    - 注意：實務上建議至 [iancoleman/bip39 Github](https://github.com/iancoleman/bip39/releases) 下載單機版「bip39-standalone.html」網頁，使用起來更為安全

![](./images/safe-009_bip39.png)

- 同樣在 iancoleman/bip39 頁面，將卷軸捲至下方的「Derived Addresses」區域，可以看到「m/44'/60'/0'/0/0」所顯示的 Address 地址要和 Metamask 的地址一致（右方為產生的子私鑰）

- 接下來我們要將「m/44'/60'/0'/0/1」到「m/44'/60'/0'/0/4」右方的子私鑰，依序輸入至 MetaMask 裡，以產生多組可供多簽的 Account 帳戶

![](./images/safe-010_bip39.png)

- 請點選 Metamask 右上角的【帳戶圖示】→ 點選【匯入帳戶】→ 將上一個步驟複製的其中一組子私鑰貼在「請貼上您的私鑰字串」【欄位】中 → 點選【匯入】，此時應可看到 MetaMask 多出了第 2 組帳戶

![](./images/safe-011_metamask.png)

- 請按照上述方法，依序將「m/44'/60'/0'/0/1」到「m/44'/60'/0'/0/4」右方的子私鑰輸入到 MetaMask 中，最終會在 MetaMask 中看到總共 5 組帳戶（Account 1 至 Account 5）可供使用

![](./images/safe-012_metamask.png)

### 取得免費的 Rinkeby Testnet 測試網路用 ETH 幣

- 因為後面要建立 Gnosis Safe 多簽錢包，需要與乙太網路上的智能合約溝通，所以要先取得用於 Rinkeby 測試網用的 ETH 幣，才可以透過支付 Gas 手續費和智能合約互動

- 請至 Chain Link 提供的 [Rinkeby Faucets](https://faucets.chain.link/rinkeby) 水龍頭網站依序將 Account 1 至 Account 5 輸入以取得測試網乙太幣

- 請在「Network」點選【Ethereum Rinkeby】→【複製】MetaMask 上 Account 1 的地址 → 貼在「Testnet account address」【欄位】→【勾選】「0.1 test ETH」→【勾選】「我不是機器人」→完成 Google 免費勞工【驗證】→ 點選【Send request】

![](./images/safe-013_faucets.png)

- 看到「Request complete」訊息後 → 即可【Close】關閉視窗 → 在 MetaMask 也看到 Account 1 獲得 0.1 顆測試用 ETH 幣

![](./images/safe-014_faucets.png)

- 請依序將 Account 2 至 Account 5 輸入至 [Rinkeby Faucets](https://faucets.chain.link/rinkeby) 以各取得 0.1 顆 ETH 測試幣

![](./images/safe-015_faucets.png)

### 取得免費的 Rinkeby Testnet 測試網路用 NFT 非同質化代幣

- 因為後面會使用 Gnosis Safe 來演練多簽狀態下轉送 NFT，所以先取得用於 Rinkeby 測試網用的 NFT

- 請至 Scrappy Squirrels 團隊提供的 [Rinkeby Squirrels](https://rsq-frontend.vercel.app/) 網站，只需讓 Account 1 取得測試網 NFT

- 請點選 MetaMask 右上角的【帳戶圖示】→ 點選並切換至【Account 1】→ 點選【Connect to  Wallet】→ 確認已【勾選】「Account 1」→【下一頁】→ 點選【連線】

![](./images/safe-016_nft.png)

- 再點選【Mint a Rinkeby Squirrel NFT】鈕開始鑄造 → 鑄造 Squirrel NFT 最多需要約 0.01024（會因實際上執行的運算量而減少），點選【確認】開始支付，等待一陣子可以點選 MetaMask 的【交易紀錄】已完成 NFT 鑄造

![](./images/safe-017_nft.png)

- 接下來即可以至 [OpenSea on Testnets](https://testnets.opensea.io/account) 上看到自己已完成鑄造的測試網 NFT

- 請點選連線【MetaMask】→【打勾】Account →【下一頁】→【連線】，即可看到自己在 Rinkeby 測試網上鑄造出的 NFT 了！

![](./images/safe-018_nft.png)

## 開始至 Gnosis Safe 建置一個多簽錢包
  Start to Gnosis Safe Create a Multisig Wallet

- 本處將使用 MetaMask 的「Account 2」、「Account 3」、「Account 4」三個帳戶來建置一個多簽錢包，並且限制其中 2 個帳戶允許轉帳時將交易送出

- 請至「[Gnosis Safe](https://gnosis-safe.io/)」官網，在建立多簽錢包前，先點選切換至 MetaMask 上的【Account 2】→ 再點選網站右上角【Open app】鈕

![](./images/safe-019_safe.png)

- 首先點選 APP 網站右上角的【網路圖示】→ 點選切換至【Rinkeby】網路 →點選【+ Create new Safe】鈕

![](./images/safe-020_safe.png)

- 請點選【Connect】→ 點選【Metamask】→ 確定【勾選】「Account 2」→【下一頁】→【連線】

    - 注意：這邊可以看到，若想提升安全性至極致，是可以連結使用像是 Ledger、Trezor 等硬體錢包來建立多簽錢包的

![](./images/safe-021_safe.png)

- 請點選【Continue】→ 在「Safe name」中輸入名稱【Safe Test】（可自由取名）→【Continue】→ 在「Owner Name」中輸入【Account 2】→ 點選【+ Add another owner】以增加此多簽錢包的其他管理者

![](./images/safe-022_safe.png)

- 將「MetaMask」內的「Account 3」地址【複製】到 Safe Test 錢包第二格，並將「Owner Name」取名為【Account 3】→ 將「Account 4」地址【複製】到 Safe Test 錢包第三格，並將「Owner Name」取名為【Account 4】→ 點選「【2】out of 3 owner(s)」將同意數量閾值設置在 2 位管理者 → 確認 MetaMask 已切換回【Account 2】→ 點選【Continue】繼續

![](./images/safe-023_safe.png)

- 確認資訊無誤後，即可點選【Create】進行 Safe Test 錢包的建置 → 點選【確認】→ 點選【Get started】→ 點選【Continue】開始使用

    - 注意：本處在 Rinkeby 測試網上建置出來的 Safe Test 錢包（其實是一份智能合約），就只適用於 Rinkeby 測試網上使用，若是將其他網路，如：Mainnet 主網上的資產傳送至此 Safe Test 錢包內，將直接遺失（因為 Mainnet 主網上並沒有此份智能合約），實際上操作時需非常當心所在的網路位置！

![](./images/safe-024_safe.png)

- 因為在 Rinkeby 網上使用 Gnosis Safe 錢包地址前方會帶有「rin:」前綴字，為方便後續操作，我們可以設定不要複製到「rin:」前綴字，請點選左側【SETTINGS】→【Appearance】→【取消勾選】「Copy addresses with chain prefix.」

![](./images/safe-025_safe.png)

- 若對 Gnosis Safe 多簽錢包感興趣，在建置自己的 Gnosis Safe 多簽錢包其實是呼叫 Gnosis Safe Proxy 智能合約（在 Rinkeby 網上的 Gnosis Safe Proxy 位置在：[0xa6b71e26c5e0845f74c812102ca7114b6a896ab2](https://rinkeby.etherscan.io/address/0xa6b71e26c5e0845f74c812102ca7114b6a896ab2)（每一個在 Rinkeby 測網上都一樣））來建置「多簽錢包合約」（執行「createProxyWithNonce(_singleton, initializer, saltNonceWithCallback);」此 Write 函數來產生，本處多簽錢包合約位置在：[0xf425aAcc34D53295e56e5A872cD77F5ce8D0ecc9](https://rinkeby.etherscan.io/address/0xf425aAcc34D53295e56e5A872cD77F5ce8D0ecc9)（每一個人的位置均不會相同））

![](./images/safe-026_proxy.png)

- 以 Proxy 智能合約部署的「[Proxy 合約設計模式](https://blog.openzeppelin.com/proxy-patterns/)」可以降低部署程式碼的計算成本（Gas 手續費），將合約拆開為 Storage（Proxy 合約）及 Logic（錢包合約）兩大部份，同時也降低了程式碼的重覆次數（因為 Logic 合約程式碼大家都是相同的

![](./images/safe-027_proxy.png)

## 將「Account 1」的資產及 NFT 轉到「Safe Test」多簽錢包
  Transfer "Account 1" assets and NFT to "Safe Test" multi-signature wallet

- 接著為實現將「Safe Test 錢包」轉帳至「Account 5」，首先將「Account 1」的測試用 ETH 及 NFT 轉移至「Safe Test 錢包」內

- 請【複製】Safe Test 錢包地址 → 點選 MetaMask 右上角【帳戶圖示】確認現在為【Account 1】→ 點選【發送】鈕 → 貼上【Safe Test 錢包地址】→ 在「數量」輸入【0.05】→【下一頁】→【確認】

- 待交易完成後，即可在 Safe Test 錢包點選【ASSETS】→【Coins】看到「0.05 測試 ETH」資產已入帳

![](./images/safe-028_safe.png)

- 再來請至 [OpenSea on Testnets](https://testnets.opensea.io/account) 點選測試網 NFT 左下角的【…】符號 →【Transfer】→ 點選下方【Transfer】→ 輸入【Safe Test 錢包地址】→【確認】

- 待交易完成後，即可在 Safe Test 錢包點選【ASSETS】→【Collectibles】看到「測試 NFT」資產已轉入

![](./images/safe-029_safe.png)







### 取得免費的 The Graph Testnet 測試網路用 GRT 代幣

- The Graph 是區塊鏈上的 GraphQL API 搜尋引擎，主要由 5 種角色組成
  - Developer：DAPPs 項目方，為讓自家的 DAPPs 被推廣及被搜尋，會在 The Graph 上建立 Subgraph，藉此紀錄 DAPPs 資料在以太坊上的位置、及資料的儲存格式
  - Indexer：索引節點維護者，負責提供索引的建立與查詢，並收取查詢手續費作為收入
  - Curator：評估 Developer 建置的 Subgraph 是否值得採用（透過 Signal），告訴 Indexer 可以索引這些 Subgraph，且 Curator 可取得部份查詢手續費作為正確標記 Subgraph 的獎勵
  - Delegator：質押手中的 GRT 給 Indexer，並收取利息作為收入
  - Consumer：支付 GRT 進行指定 Subgraph 的查詢

![](./images/graph-001_subgraph.png)

- 為免費取得在 Rinkeby 測試網上的 GRT 代幣，需先加入 [The Graph Discord](https://discord.gg/vtvv7FP) 