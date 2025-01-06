# 楽曲著作権NFTと楽曲原盤権NFT

## プロジェクト概要
音楽著作権と原盤権を管理するための基盤となるスマートコントラクトを実装する
楽曲著作権と楽曲原盤権の双方の関係図は以下の図の通り
![NFT間の関係図](https://github.com/user-attachments/assets/550e48ab-265e-43a7-9a17-1b4af197ccff)


## hardhatプロジェクトの作成
- プロジェクトディレクトリ
```
~/hardhat-projects/MusicNFT
```

## OpenZeppelinコントラクトのインストール
```
npm install --save-dev @openzeppelin/contracts
```

## コントラクトの作成
楽曲著作権NFTは`MusicNFT.sol`という名で作成, 楽曲原盤権NFTは`RecordingNFT.sol`という名で作成

### 主な機能の説明：

**1. 継承しているコントラクト**

  - ERC721: NFTの基本機能
  - ERC721Enumerable: トークン一覧の取得機能
  - ERC721URIStorage: メタデータのURI管理機能
  - ERC721Burnable: トークンの破棄機能
  - Ownable: 所有権管理機能

**2. 主要な関数**

  - safeMint: 新しいNFTを発行
  - setRecordingReference: 原盤権NFTとの関連付け
  - getRecordingReference: 関連付けられた原盤権NFTの取得

**3. ストレージ**

  - _nextTokenId: 次に発行するトークンのID
  - recordingTokenIds: 著作権NFTと原盤権NFTの紐付けを保存
  - recordingContract: 原盤権NFTコントラクトのアドレス

**4. アクセス制御**

  - onlyOwner: コントラクト所有者のみが実行可能な関数の制限
  - 所有者チェック: トークン所有者のみが参照設定可能

## テストの作成



