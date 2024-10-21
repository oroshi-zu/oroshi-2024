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
    constructor(uint initialSupply) ERC20("MyToken", "MTK") {
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
## デプロイスクリプトの作成
