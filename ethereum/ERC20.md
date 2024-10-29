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
デプロイ主体のアカウント: 0x.. // 0xから始まるアドレス
オーナー: 0x.. // ←コントラクトアドレス
```
コントラクトアドレスをメモしておく

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
    // 1000トークンを初期供給量として設定
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

oroshi@LAPTOP-9RCG2VAF:~/hardhat-projects/erc20$ npx hardhat test


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
![MetaMask1](https://github.com/user-attachments/assets/235e85f8-a5cb-4cec-8ede-16dfe6ddf5c0)


- 先ほどのコントラクトアドレスを入力する


![MetaMask2](https://github.com/user-attachments/assets/4df8bff2-94fe-4bae-9e4c-6d79eae0a6c7)


- トークンがインポートされる


![MetaMask3](https://github.com/user-attachments/assets/401db7ec-6c6f-4bb7-83e5-aa150a836e3e)


- 同じ手順で移動先のアカウントにもトークンをインポートする


![MetaMask8](https://github.com/user-attachments/assets/8c14c956-9b3c-4188-9fc8-11a56bf5b1fa)



### 他のアカウントへのトークンの送金
- 送金ボタンを押したのち、移動元と移動先を選択
- 今回は100トークン送金
- **続行**を選択


![MetaMask4](https://github.com/user-attachments/assets/4dca46c0-a60e-4e05-b306-0f33ac998a8b)


- **確認**を選択


![MetaMask5](https://github.com/user-attachments/assets/7079d2a3-6f4d-4a06-9514-da66584d2af4)


- お互いのアカウントでトークンの残高の増減を確認


![MetaMsk6](https://github.com/user-attachments/assets/795a89e8-bbd2-4132-9c7f-af084415dcdd)


![MetaMask7](https://github.com/user-attachments/assets/c735f443-36ac-4a67-af9c-ec10c27c16eb)
