# 結合鏈上智能合約及自架的鏈下 Gateway Server 達到使用低 gas 的自訂義子堿名做為自己的錢包地址

## 參考資料

- ENS Offchain Resolver GitHub

  - [https://github.com/ensdomains/offchain-resolver](https://github.com/ensdomains/offchain-resolver/commit/ed330e4322b1fafe2ffbd1496829c75185dd9e2e)

- EIP-3668

  - [https://github.com/ethereum/EIPs/blob/master/EIPS/eip-3668.md](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-3668.md)

## 前言

- 目前乙太坊錢包地址最常見還是使用 40 個英數字母所組成，實際上不易人類記憶
- 是否可自訂義好記的 ENS 名稱來取代不易記憶的錢包地址？
- 目前有人提出 EIP-3688 解決方案，即可透過組合鏈上智能合約以及鏈下 Gateway Server 來達成
- 如此使智能合約可以以低 gas 成本取得 Gateway Server 簽署的回傳資料，來解析或設置使用者自訂義的 ENS 名稱
- 在不久的將來，各大乙太坊錢包均可為客戶提供有 EIP-3688 功能的自訂義 ENS 名稱來安全地接收乙太幣
- 本處以重現 ens.domains 所實作的 offchain-resolver 來嘗鮮 ENS 名稱解析服務

## 實作作業系統

- Windows 11

## 安裝及設置 Node.js 編譯環境

- 請使用系統管理員權限開啟 PowerShell 並執行以下指令

```powershell
# 安裝 Chocolatey
>Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

- 使用 Chocolatey 安裝版控指令 Git、版控圖形化工具 GitHub Desktop、整合編譯環境 Visual Studio Code、開發工具 Node.js、Node.js 相依套件管理工具 Yarn、程式語言工具 python3

```powershell
# Chocolatey
> choco install -y git.install github-desktop vscode nodejs-lts python3
> git --version
> code --version
> node --version
> python3 --version
# 為使 yarn 可順利執行，修改 Windows 執行腳本權限
> Set-ExecutionPolicy Bypass
# 使用 NPM 安裝 Yarn 套件管理工具
> npm install --global yarn
> yarn --version
```

## 建立鏈上智能合約以及鏈下 Gateway Server 來達成 ENS 解析

- 請下載 ENSdomains 的 offchain-resolver 專案，並使用 Visual Studio Code 開啟

```powershell
> mkdir "$env:USERPROFILE\Documents\GitHub\ensdomains\offchain-resolver"
> git clone https://github.com/ensdomains/offchain-resolver.git "$env:USERPROFILE\Documents\GitHub\ensdomains\offchain-resolver"
> code "$env:USERPROFILE\Documents\GitHub\ensdomains\offchain-resolver"
```

- 使用 Yarn 指令並根據 ./package.json 設置檔下載並安裝本處實作會使用到的相依套件
    - 註：請點選 Visual Studio Code 上方【Terminal】→【New Terminal】以執行下方指令
```powershell
offchain-resolver > yarn
offchain-resolver > yarn build
```

- 一般情況下，請使用 [Mnemonic Code Converter](https://github.com/iancoleman/bip39/releases) 離線工具或使用下方指令產生一組 Gateway Server 專用私鑰

```powershell
# 產生一組 Gateway Server 用私鑰並儲存至 $prvkey 變數
offchain-resolver > $prvkey = python.exe -c "import os; import binascii; print('0x%s' % binascii.hexlify(os.urandom(32)).decode('utf-8'))"
# 在本範例中，我們設置 Hardhat Node 預設的 Private Key 來進行實作
offchain-resolver > $prvkey = "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
offchain-resolver > echo $prvkey
```

```powershell
0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff8
```

- 查看 ./packages/gateway/test.eth.json 範例檔，可以了解如何設置 Domain 主網域名稱或 Subdomain 子網域名稱的 JSON 檔
```powershell
offchain-resolver > code "./packages/gateway/test.eth.json"
```

```json
{
    "//": "主網域 Domain",
    "test.eth": {
        "addresses": {
            "60": "0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266",
            "0":  "0x76a91462e907b15cbf27d5425399ebf6f0fb50ebb88f1888ac"
        },
        "text":{ "email": "test@example.com" },
        "contenthash": "0xe301017012204edd2984eeaf3ddf50bac238ec95c5713fb40b5e428b508fdbe55d3b9f155ffe"
    },
    "//": "子網域 Subdomain",
    "*.test.eth": {
        "addresses": {
            "60": "0x70997970c51812dc3a010c7d01b50e0d17dc79c8",
            "2":  "0xa914b48297bff5dadecc5f36145cec6a5f20d57c8f9b87"
        },
        "text":{ "email": "wildcard@example.com" },
        "contenthash": "0xe301017012204edd2984eeaf3ddf50bac238ec95c5713fb40b5e428b508fdbe55d3b9f155ffe"
    }
}
```

- 接著我們可根據 ./packages/gateway/test.eth.json 檔的 ENS Names 內容在本機啟動一臺 Gateway Server

```powershell
offchain-resolver > yarn start:gateway --private-key $prvkey --data test.eth.json
```
（執行結果如下）

```
yarn run v1.22.19
$ yarn workspace @ensdomains/offchain-resolver-gateway start --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80 --data test.eth.json
$ node dist/index.js --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80 --data test.eth.json    
Serving on port 8080 with signing address 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
```

- 啟動 Hardhat 本機節點及部署 ENSRegistry 及 OffchainResolver 智能合約
    - 註：請保持上述 Gateway Server 常駐執行，請再點選 Visual Studio Code 上方【Terminal】→【New Terminal】開啟新的 Terminal 再執行下方指令

```powershell
# 為避免新版 Node.js 使用較新 openssl 導致問題，設置舊版的 openssl
offchain-resolver > $env:NODE_OPTIONS="--openssl-legacy-provider"
offchain-resolver > echo $env:NODE_OPTIONS
# 啟動 Hardhat 本機節點，因為 ./packages/contracts/hardhat.config.js 有引入 hardhat-deploy 模組
# 所以會根據 ./packages/contracts/deploy 內的腳本部署 ENSRegistry 及 OffchainResolver 智能合約
offchain-resolver > cd packages/contracts
offchain-resolver > npx hardhat node --network hardhat
```

（執行結果如下）

```
Compilation finished successfully
deploying "ENSRegistry" (tx: 0x9571f1662d52a0c15eb986bb82d77e7ed46a1e5468162a219b8448e76a0315fa)...: deployed at 0x5FbDB2315678afecb367f032d93F642f64180aa3 with 1084532 gas
deploying "OffchainResolver" (tx: 0x8d8a164cbcde761fbbec5514b02f6e8697cd58a908deec72981fa6e6e6414310)...: deployed at 0x8464135c8F25Da09e49BC8782676a84730C318bC with 1526010 gas
Started HTTP and WebSocket JSON-RPC server at http://127.0.0.1:8545/

Accounts
========

WARNING: These accounts, and their private keys, are publicly known.
Any funds sent to them on Mainnet or any other live network WILL BE LOST.

Account #0: 0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266 (10000 ETH)
Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

Account #1: 0x70997970c51812dc3a010c7d01b50e0d17dc79c8 (10000 ETH)
Private Key: 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d

…（中間略）

WARNING: These accounts, and their private keys, are publicly known.
Any funds sent to them on Mainnet or any other live network WILL BE LOST.
```

- 請記下上方顯示的 ENSRegistry 智能合約地址（可能每個人不相同），並儲存至 $ensAddress 變數
    - 註：請保持上述 Gateway Server、Hardhat Node 常駐，再點選 Visual Studio Code 上方【Terminal】→【New Terminal】開啟新的 Terminal 再執行下方指令

```powershell
offchain-resolver > $ensAddress = "0x5FbDB2315678afecb367f032d93F642f64180aa3"
offchain-resolver > $provider = "http://localhost:8545/"
offchain-resolver > echo $ensAddress
offchain-resolver > echo $provider
```

- 開始將「test.eth」主網域名稱進行地址解析

```powershell
offchain-resolver > yarn start:client --registry $ensAddress --provider $provider test.eth
```
（執行結果如下）
```
yarn run v1.22.19
$ yarn workspace @ensdomains/offchain-resolver-client start --registry 0x5FbDB2315678afecb367f032d93F642f64180aa3 --provider http://127.0.0.1:8545/ test.eth
$ node dist/index.js --registry 0x5FbDB2315678afecb367f032d93F642f64180aa3 --provider http://127.0.0.1:8545/ test.eth
resolver address 0x8464135c8F25Da09e49BC8782676a84730C318bC
eth address 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
btc address 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa
content ipfs://QmTeW79w7QQ6Npa3b1d5tANreCDxF2iDaAPsDvW6KtLmfB
Done in 4.39s.
```

## 本次實作在進行 *.test.eth 名稱解析時出錯，並著手進行原因查找

- 將 foo.test.eth 子網堿名稱進行地址解析（會出錯）

```powershell
offchain-resolver > yarn start:client --registry $ensAddress --provider $provider foo.test.eth
```

（執行結果如下）

```
…（前略）
{
  reason: 'invalid or unsupported coin data',
  code: 'UNSUPPORTED_OPERATION',
  operation: 'getAddress(0)',
  coinType: 0,
  data: '0x0000000000000000000000000000000000000000'
}
error Command failed with exit code 1.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
error Command failed.
Exit code: 1
…（後略）
```

- 為何在 ./packages/client/src/index.ts 其中 resolver.getAddress(0); 取得第 0 號幣（0 號幣為比特幣，若未帶號碼預設為 60 號乙太幣）程式碼無法解析

```powershell
offchain-resolver > code ./packages/client/src/index.ts 
```

```
…（前略）
  let resolver = await provider.getResolver(name);
  if (resolver) {
    let ethAddress = await resolver.getAddress();
    let btcAddress = await resolver.getAddress(0);
    let content = await resolver.getContentHash();
    console.log(`resolver address ${resolver.address}`);
    console.log(`eth address ${ethAddress}`);
    console.log(`btc address ${btcAddress}`);
    console.log(`content ${content}`);
  } else {
    console.log('no resolver found');
  }
…（後略）
```

- 因為 ./packages/gateway/test.eth.json 中的 *.test.eth 子網域並沒有設置 0 號幣，只有設置 60 號幣及 2 號幣（萊特幣）

```powershell
offchain-resolver > code "./packages/gateway/test.eth.json"
```

```json
…（前略）
    "*.test.eth": {
        "addresses": {
            "60": "0x70997970c51812dc3a010c7d01b50e0d17dc79c8",
            "2":  "0xa914b48297bff5dadecc5f36145cec6a5f20d57c8f9b87"
        },
        "text":{ "email": "wildcard@example.com" },
        "contenthash": "0xe301017012204edd2984eeaf3ddf50bac238ec95c5713fb40b5e428b508fdbe55d3b9f155ffe"
    }
…（後略）
```

- 本次實作我們暫時將 *.test.eth 子網域中的 2 號幣換成 0 號幣後存檔（實際上不同幣種不能任意替換）

```json
…（前略）
    "*.test.eth": {
        "addresses": {
            "60": "0x70997970c51812dc3a010c7d01b50e0d17dc79c8",
            "0":  "0xa914b48297bff5dadecc5f36145cec6a5f20d57c8f9b87"
        },
        "text":{ "email": "wildcard@example.com" },
        "contenthash": "0xe301017012204edd2984eeaf3ddf50bac238ec95c5713fb40b5e428b508fdbe55d3b9f155ffe"
    }
…（後略）
```

- 回到第 1 個正在執行 Gateway Server 的 Terminal
- 按下【Ctrl + C】鈕強制停止 Gateway Server
- 再次輸入以下指令重新啟動 Gateway Server

```
offchain-resolver > yarn start:gateway --private-key $prvkey --data test.eth.json
```

- 回到第 3 個 Terminal
- 重新執行以下指令，即可將 foo.test.eth 子網堿名稱進行地址解析

```powershell
offchain-resolver > yarn start:client --registry $ensAddress --provider $provider foo.test.eth
```

（執行結果如下）

```
yarn run v1.22.19
$ yarn workspace @ensdomains/offchain-resolver-client start --registry 0x5FbDB2315678afecb367f032d93F642f64180aa3 foo.test.eth
$ node dist/index.js --registry 0x5FbDB2315678afecb367f032d93F642f64180aa3 foo.test.eth
resolver address 0x8464135c8F25Da09e49BC8782676a84730C318bC
eth address 0x70997970C51812dc3A010C7d01b50e0d17dc79C8
btc address 3J9TzpQYLReDbfaiTY4XN3izFGMyswQgCb
content ipfs://QmTeW79w7QQ6Npa3b1d5tANreCDxF2iDaAPsDvW6KtLmfB
Done in 3.06s.
```

## 結論

- 經過這次實作可得知 ENS 主／子網域解析可以包含多個幣種
- 這次實作鏈上的智能合約可方便地取得鏈下 ENS 名稱對映的地址
- 本處範例的 Gateway Server 目前無法在啟動時實時取得最新的 ./packages/gateway/test.eth.json 檔，若有此檔有更新，則 Gateway Server 需重新啟動才能生效
- 本處範例 Client 端的 ./packages/client/src/index.ts 應要可判斷 resolve 的標的是比特幣還是萊特幣
- 最後，本處 EIP-3668 實作尚需改善不少地方，例如 Gateway Server 可實時更新、Client 可判斷幣種等，才可用於 Production 生產環境