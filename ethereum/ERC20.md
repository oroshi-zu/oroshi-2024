# ERC20
OpenZeppelinのERC20を利用したトークン間での送金を行うコントラクト

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
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// OpenZeppelinのERC20コントラクトをインポート
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

// MyERC20コントラクトで、OpenZeppelinのERC20コントラクトを継承
contract MyERC20 is ERC20 {
　　// コントラクトのオーナー
    address public owner;
　　
    // コンストラクタ：コントラクトのデプロイ時に実行される
    // initialSupply: 初期供給量を指定
    constructor(uint256 initialSupply) ERC20("MyToken", "MTK") {
      // _mint関数を呼び出して、指定された量のトークンを作成し、
      // それらをコントラクトをデプロイしたアドレス（msg.sender）に割り当てる
　　　　owner = msg.sender
        _mint(owner, initialSupply);
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
  const [deployer] = await ethers.getSigners();

  console.log("Deploying contracts with the account:", deployer.address);

  const MyERC20 = await hre.ethers.getContractFactory("MyERC20");
  const myERC20 = await MyERC20.deploy(hre.ethers.utils.parseEther("1000"));
  await myERC20.deployed();

  console.log("MyERC20 deployed to:", myERC20.address);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

### 解説みたいなもの
- `ethers.getSigners()`: この関数は、Hardhat Networkで利用可能なアカウント（シグナー）のリストを返します。デフォルトでは、Hardhatは10個のテストアカウントを提供します。[deployer] で最初のアカウントを取得しています。
- `ethers.getContractFactory(name)`: コントラクトのファクトリを取得します。これを使用してコントラクトをデプロイできます。

### ローカルノードへのデプロイ
```
npx hardhat run scripts/deploy.js --network localhost
```
