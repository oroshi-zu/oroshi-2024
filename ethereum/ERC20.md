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
デプロイ主体のアカウント: 0x.. // 0xから始まるアドレス
オーナー: 0x..
```

## テストプログラム
```
nano test/Token20.js
```
