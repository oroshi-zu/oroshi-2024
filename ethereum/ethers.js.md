# ethers.js

## ethers.js v5からv6への主な変更点
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
