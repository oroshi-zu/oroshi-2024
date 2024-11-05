# ERC20
OpenZeppelinのERC20を利用したトークン間での送金を行うコントラクト

## ERC20の仕様
[ERC-20: Token Standard](https://eips.ethereum.org/EIPS/eip-20)

## hardhatプロジェクトの作成
- ERC20プロジェクトディレクトリ
```
cd ~/hardhat
mkdir erc20
cd erc20
```

- hardhatのインストール
```
npm init
npm install --save-dev hardhat
```

- hardhatの初期化
```
npx hardhat init
```

- Create a JavaScript project を選択
- サンプルプログラムの削除
```
rm contracts/Lock.sol
rm test/Lock.js
```

## OpenZeppelinコントラクトのインストール
```
npm install --save-dev @openzeppelin/contracts
```

## コントラクトの作成
`MyERC20.sol`という名で作成

```
nano contracts/MyERC20.sol
```
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyERC20 is ERC20 {
    constructor(uint256 initialSupply) ERC20("MyERC20", "ME20") {
        _mint(msg.sender, initialSupply);
    }
}
```

## hardhat node の起動
### hardhat設定ファイル hardhat.config.js の修正
- ネットワークを hardhat networkにする
- チェインIDを 31337
- マイニングの間隔を10秒に設定

```
nano hardhat.config.js
```
```
require("@nomicfoundation/hardhat-toolbox");

/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: "0.8.27",
  networks: {
    hardhat: {
      chainId: 31337,
      mining: {
        auto: false,
        interval: [10000, 11000]
      }
    },
  },
};
```
## ローカルノードの起動
新しく端末を開いて実行
```
npx hardhat node
```

## デプロイスクリプトの作成
### Ethers.jsのインストール
デプロイスクリプト作成にあたって、ethers.js ライブラリをインストールする
```
npm install --save-dev ethers
```

scripts ディレクトリを作成して deploy スクリプトを作成する

```
mkdir scripts
nano scripts/deploy.js
```

```javascript
const hre = require("hardhat");

async function main() {
  const [deployer] = await hre.ethers.getSigners();
  console.log("デプロイ主体のアカウント:", deployer.address);

  // 初期供給量を ethers.parseUnits を使用して設定
  const initialSupply = hre.ethers.parseUnits("1000", 18);

  // トークンをデプロイ
  const MyERC20 = await hre.ethers.deployContract("MyERC20",[initialSupply]);
  await MyERC20.waitForDeployment();

  // デプロイされたコントラクトのアドレスを取得
  const contractAddress = await MyERC20.getAddress();
  console.log("オーナー:", contractAddress);

}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

### ローカルノードへのデプロイ
```
npx hardhat run scripts/deploy.js --network localhost
```

上手くいくと、以下のように表示される
```
デプロイ主体のアカウント: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
オーナー: 0x5FbDB2315678afecb367f032d93F642f64180aa3
```

- ログの確認
```
Mined block #3
  Block: 0x02c3554d6ffc0357be96adeee7f166c0f6da4c4e61fb847381def660894d2fbe
    Base fee: 669921875
    Transaction:           0x2ee57f341412d97757f09f272e36f4b1d31ceddcc91b62d22305f7146808e41a
      Contract deployment: MyERC20
      Contract address:    0x5fbdb2315678afecb367f032d93f642f64180aa3
      From:                0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266
      Value:               0 ETH
```

コントラクトアドレスをメモしておく

## コンソールからの操作

### コンソールの起動
```
npx hardhat console --network localhost 
```

### ERC20コントラクトへの操作
```
> const MyERC20 = "0x5fbdb2315678afecb367f032d93f642f64180aa3"

> const MyERC20factory = await ethers.getContractFactory("MyERC20")

// 当該コントラクトアドレスのコントラクトに接続する
> const ME20 = await MyERC20factory.attach(MyERC20)

// EOAのアドレス
> const [owner, addr1, addr2] = await ethers.getSigners();

// トークンの総供給量
> await ME20.totalSupply()
1000000000000000000000n
// owner のトークン保有量
> await ME20.balanceOf(owner)
1000000000000000000000n
> await ME20.name()
'MyERC20'
> await ME20.symbol()
'ME20'
> await ME20.decimals()
18n
```

- トークンの送付
```js
> await contract.transfer(addr1,80)
ContractTransactionResponse {
  provider: HardhatEthersProvider {
    _hardhatProvider: LazyInitializationProviderAdapter {
      _providerFactory: [AsyncFunction (anonymous)],
      _emitter: [EventEmitter],
      _initializingPromise: [Promise],
      provider: [BackwardsCompatibilityProviderAdapter]
    },
    _networkName: 'localhost',
    _blockListeners: [],
    _transactionHashListeners: Map(0) {},
    _eventListeners: []
  },
  blockNumber: null,
  blockHash: null,
  index: undefined,
  hash: '0x58f12db917786dad6b7da43b9ac95bd6687a53a0fc95fecf0382ccd00e39aafe',
  type: 2,
  to: '0x5FbDB2315678afecb367f032d93F642f64180aa3',
  from: '0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266',
  nonce: 1,
  gasLimit: 30000000n,
  gasPrice: 1002745697n,
  maxPriorityFeePerGas: 1000000000n,
  maxFeePerGas: 1002745697n,
  maxFeePerBlobGas: null,
  data: '0xa9059cbb00000000000000000000000070997970c51812dc3a010c7d01b50e0d17dc79c80000000000000000000000000000000000000000000000000000000000000050',
  value: 0n,
  chainId: 31337n,
  signature: Signature { r: "0x120f7d74e148135834964d109f816956cd382d3f6550b4e8ff7fea8983f01381", s: "0x15b6b8f7ef2aab9ff440ff95fefb7749db4e49fc046af36eaa14346d3fb50059", yParity: 1, networkV: null },
  accessList: [],
  blobVersionedHashes: null
}
```

- アカウントのトークン保有残高の確認
```js
> await contract.balanceOf(owner)
999999999999999999920n
> await contract.balanceOf(addr1)
80n
```

## テストプログラム
```
nano test/Token20.js
```

```javascript
// hardhat tool box の利用
const { loadFixture } = require("@nomicfoundation/hardhat-toolbox/network-helpers");

const { ethers } = require("hardhat");

// Chaiの利用
const { expect } = require("chai");

describe("ERC20トークンのコントラクト", function () {
  // フィクスチャの定義
  async function deployTokenFixture() {
    // コントラクトをデプロイするアカウントとテスト用アカウントを取得
    [owner, addr1, addr2, ...addrs] = await ethers.getSigners();

    // コントラクトのデプロイ
    const MyERC20 = await ethers.getContractFactory("MyERC20");
    // 初期供給量を設定
    const initialSupply = ethers.parseUnits("1000", 18);
    const myToken = await MyERC20.deploy(initialSupply);
    await myToken.waitForDeployment();

    // 必要な値をまとめて返す
    return { myToken, owner, addr1, addr2, addrs, initialSupply };
  }

  describe("トークンの基本情報", function() {
    it("トークンの基本情報を表示", async function() {
      const { myToken, owner } = await loadFixture(deployTokenFixture);

      const name = await myToken.name();
      const symbol = await myToken.symbol();
      const decimals = await myToken.decimals();
      const totalSupply = await myToken.totalSupply();
      const address = await myToken.getAddress();

      console.log("\n=== トークン基本情報 ===");
      console.log(`トークン名: ${name}`);
      console.log(`シンボル: ${symbol}`);
      console.log(`小数点桁数: ${decimals}`);
      console.log(`総供給量: ${ethers.formatUnits(totalSupply, decimals)} ${symbol}`);
      console.log(`コントラクトアドレス: ${address}`);
      console.log("=====================\n");
    });
  });

  // トークン残高のテスト
  describe("トークン残高のテスト", function() {
    it("トークンの総量が所有者に割り当てられていること", async function() {
      const { myToken, owner, initialSupply } = await loadFixture(
        deployTokenFixture
      );

      // オーナーの所持金額
      const ownerBalance = await myToken.balanceOf(owner.address);
      // トークンの総額がオーナーの所持金に等しい
      expect(await myToken.totalSupply()).to.equal(ownerBalance);
    });
  });

  // トークン転送のテスト
  describe("トークン転送のテスト", function() {
    it("アカウント間でトークンが転送されること", async function() {

      const { myToken, owner, addr1, addr2 } = await loadFixture(
        deployTokenFixture
      );

      const transferAmount = ethers.parseUnits("50", 18);

      // オーナーからaddr1に50トークン送金
      await expect(
        myToken.transfer(addr1.address, transferAmount)
      ).to.changeTokenBalances(
        myToken,
        [owner, addr1],
        [ethers.parseUnits("-50", 18), ethers.parseUnits("50", 18)]
      );

      // addr1からaddr2に50トークン送金
      await expect(
        myToken.connect(addr1).transfer(addr2.address, transferAmount)
      ).to.changeTokenBalances(
        myToken,
        [addr1, addr2],
        [ethers.parseUnits("-50", 18), ethers.parseUnits("50", 18)]
      );
    });
  });

  // 承認と委任転送のテスト
  describe("承認と委任転送のテスト", function() {
    it("承認額が正しく更新されること", async function() {
      const { myToken, owner, addr1 } = await loadFixture(
        deployTokenFixture
      );

      // 承認を実行し、トランザクションの完了を待つ
      const approveAmount = ethers.parseUnits("100", 18);
      const approveTx = await myToken.approve(addr1.address, approveAmount);
      await approveTx.wait();

      expect(await myToken.allowance(owner.address, addr1.address))
        .to.equal(approveAmount);
    });

    it("承認を受けたアドレスが委任転送が正しく実行できること", async function() {
      const {myToken, owner, addr1, addr2 } = await loadFixture(
        deployTokenFixture
      );

      // 100トークン承認する
      const approveAmount = ethers.parseUnits("100", 18);
      // 50トークン委任転送する
      const transferAmount = ethers.parseUnits("50", 18);

      // 承認を実行
      const approveTx = await myToken.approve(addr1.address, approveAmount);
      await approveTx.wait();

      // 転送を実行
      const transferTx = await myToken.connect(addr1)
        .transferFrom(owner.address, addr2.address, transferAmount);
      await transferTx.wait();

      expect(await myToken.balanceOf(addr2.address)).to.equal(transferAmount);
      expect(await myToken.allowance(owner.address, addr1.address))
        .to.equal(approveAmount - transferAmount);
    });
  });
});
```

テストの実行
```
npx hardhat test

=>

  ERC20トークンのコントラクト
    トークンの基本情報

=== トークン基本情報 ===
トークン名: MyERC20
シンボル: ME20
小数点桁数: 18
総供給量: 1000.0 ME20
コントラクトアドレス: 0x5FbDB2315678afecb367f032d93F642f64180aa3
=====================

      ✔ トークンの基本情報を表示 (11570ms)
    トークン残高のテスト
      ✔ トークンの総量が所有者に割り当てられていること
    トークン転送のテスト
      ✔ アカウント間でトークンが転送されること (20523ms)
    承認と委任転送のテスト
      ✔ 承認額が正しく更新されること (10772ms)
      ✔ 承認を受けたアドレスが委任転送が正しく実行できること (20389ms)


  5 passing (1m)
```

## MetaMaskでの送金
hardhat node が別ターミナルで実行されているか確認

### トークンの追加
- メタマスクで「トークンをインポート」を選択

- 先ほどのコントラクトアドレスを入力する

- トークンがインポートされる


![MetaMask3](https://github.com/user-attachments/assets/401db7ec-6c6f-4bb7-83e5-aa150a836e3e)


- 同じ手順で移動先のアカウントにもトークンをインポートする


### 他のアカウントへのトークンの送金
- 送金ボタンを押したのち、移動元と移動先を選択

- 今回は100トークン送金

- お互いのアカウントでトークンの残高の増減を確認


![MetaMsk6](https://github.com/user-attachments/assets/795a89e8-bbd2-4132-9c7f-af084415dcdd)


![MetaMask7](https://github.com/user-attachments/assets/c735f443-36ac-4a67-af9c-ec10c27c16eb)


## Faucet : ERC20トークンを Eth で購入するコントラクトの作成
- プロジェクトルートに移動

### Eth でトークンを販売するコントラクトの作成手順
1. トークン購入者はETHを支払い、トークンを受け取る。
2. トークンの売り手は、コントラクトが指定する量のトークンをtransferFrom関数を使って購入者に送る。
3. レートは1ETHあたりのトークン量として決めます。

### コントラクトのコード
```
nano contracts/Faucet.sol
```

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;
import "./JPYcoin.sol";

// 無料配布（Faucet）システムのコントラクト
contract Faucets {
    // ユーザーごとの次回リクエスト可能時間を管理するマッピング
    mapping(address => uint256) userNextBuyTime;
    
    // リクエスト間の待機時間（変更不可）
    uint256 private immutable buyTimeLimit;
    
    // ERC20トークンのインスタンス
    CWMToken private immutable cwmToken;

    // コンストラクタ
    // @param tokenAddress - ERC20トークンのアドレス
    // @param _buyTimeLimit - リクエスト間の待機時間（秒）
    constructor(address tokenAddress, uint256 _buyTimeLimit) {
        cwmToken = CWMToken(tokenAddress);  // トークンコントラクトのインスタンス化
        buyTimeLimit = _buyTimeLimit;        // 待機時間の設定
    }

    // トークンの無料配布をリクエストする関数
    function requestTokens() public {
        // ゼロアドレスチェック
        require(msg.sender != address(0), "Cannot send token zero address");
        
        // 次回リクエスト可能時間のチェック
        require(
            block.timestamp > userNextBuyTime[msg.sender],
            "Your next request time is not reached yet"
        );
        
        // 10 etherの量のトークンを送信
        require(
            cwmToken.transfer(msg.sender, 10 ether),
            "requestTokens(): Failed to Transfer"
        );

        // 次回リクエスト可能時間を更新
        userNextBuyTime[msg.sender] = block.timestamp + buyTimeLimit;
    }

    // 次回リクエスト可能時間を取得する関数
    function getNextBuyTime() public view returns (uint256) {
        return userNextBuyTime[msg.sender];
    }
}

// ETHでトークンを購入するためのコントラクト
contract Faucet {
    // JPYcoinインターフェースの定義
    // トークンコントラクトとの相互作用に必要な関数を定義
    interface JPYcoin {
        // 指定したアドレスにトークンの使用を承認する
        function approve(address _spender, uint256 _amount) external returns (bool);
        
        // 承認された額のトークンを送信者から受信者に転送する
        function transferFrom(address _sender, address _recipient, uint256 _amount) external returns (bool);
        
        // アドレスの残高を確認する
        function balanceOf(address _account) external view returns (uint256);
    }

    JPYcoin public token;      // トークンコントラクトへの参照
    address public seller;      // トークンの販売者アドレス
    uint256 public rate;       // 交換レート（1 ETHあたりのトークン数）

    // コンストラクタ
    // @param _token - JPYcoinコントラクトのアドレス
    // @param _seller - 販売者のアドレス
    // @param _rate - 交換レート
    constructor(JPYcoin _token, address _seller, uint256 _rate) {
        token = _token;
        seller = _seller;
        rate = _rate;
    }

    // ETHを送信してトークンを購入する関数
    function buyTokens() public payable {
        // 送信されたETHが0より大きいことを確認
        require(msg.value > 0, "ETH must be greater than 0");

        // 購入できるトークン量を計算（ETH量 * レート）
        uint256 tokenAmount = msg.value * rate;

        // 販売者が十分なトークンを持っているか確認
        require(token.balanceOf(seller) >= tokenAmount, "Seller does not have enough tokens");

        // 販売者から購入者へトークンを転送
        require(token.transferFrom(seller, msg.sender, tokenAmount), "Token transfer failed");
    }

    // 販売者がコントラクトに蓄積されたETHを引き出す関数
    function withdrawETH() public {
        // 販売者のみが実行できることを確認
        require(msg.sender == seller, "Only seller can withdraw");
        
        // 販売者にETHを送信
        payable(seller).transfer(address(this).balance);
    }
}
```
