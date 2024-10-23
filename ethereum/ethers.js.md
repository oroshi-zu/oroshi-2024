# ethers.js

## テストプログラムについて
### テストの構造:
```javascript
describe("カテゴリ名", function() {
  it("テストケース名", async function() {
    // テストの内容
  });
});
```

- `describe`: テストをグループ化
- `it`: 個別のテストケース
- 入れ子にすることで階層的な構造を作れる

### beforeEach:
```javascript
beforeEach(async function() {
// セットアップコード
});
```

- 各テストケースの前に実行される
- テスト環境を初期化する
- 共通の準備処理を書く

## ethers.js v5からv6への主な変更点
[Migrating from v5](https://docs.ethers.org/v6/migrating/)

- アドレス取得:
```javascript
// v5
contract.address

// v6
await contract.getAddress()
```

- コントラクトのデプロイ待機:
```javascript
// v5
await contract.deployed()

// v6
await contract.waitForDeployment()
```

- BigNumber関連:
```javascript
// v5
ethers.BigNumber.from("1000")

// v6
ethers.parseUnits("1000", 18)  // より明示的な方法
```

- プロバイダー作成:
```javascript
// v5
new ethers.providers.JsonRpcProvider()

// v6
ethers.getDefaultProvider()
// または
new ethers.JsonRpcProvider()
```

- コントラクトインスタンスの作成:
```javascript
// v5
new ethers.Contract(address, abi, signer)

// v6
ethers.getContractAt("ContractName", address)
```
