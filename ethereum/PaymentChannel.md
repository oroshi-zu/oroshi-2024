# PaymentChannelの実装

## プロジェクトの目的
ストリーミングサービスやMetamask等の連携  
最終的にはペイメントチャネルを使った視聴履歴の集積を目的とする

基本的なペイメントチャネルの流れは以下を想定
![ペイメントチャネル流れ](https://github.com/user-attachments/assets/1030bcec-7e85-4025-a3d9-601b44569382)

## コントラクトの作成

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.22;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";

contract MusicPaymentChannel is ReentrancyGuard {
    // ECDSAライブラリを bytes32 型に対して使用可能にする
    using ECDSA for bytes32;

    struct Channel {
        address listener;      // リスナーのアドレス
        address artist;        // アーティストのアドレス
        uint256 deposit;      // デポジット額(月額料金)
        uint256 startTime;    // チャネル開設時間
        uint256 duration;     // チャネルの有効期間
        bool closed;          // チャネルが閉じられているかどうか
    }

    // チャネルIDはkeccak256ハッシュで生成される
    mapping(bytes32 => Channel) public channels;

    // イベント定義
    event ChannelOpened(bytes32 indexed channelId, address listener, address artist, uint256 deposit);
    event ChannelClosed(bytes32 indexed channelId, uint256 payment);

    // チャネルの開設
    function openChannel(address artist, uint256 duration) external payable nonReentrant {
        // デポジット額の確認
        require(msg.value > 0, "Deposit required");
        // アーティストのアドレスの確認
        require(artist != address(0), "Invalid artist address");
        // 有効期間の確認
        require(duration > 0, "Duration must be greater than 0");

        // チャネルIDの生成：送信者、アーティスト、タイムスタンプから一意のIDを作成
        bytes32 channelId = keccak256(
            abi.encodePacked(msg.sender, artist, block.timestamp)
        );

        // チャネル情報の保存
        channels[channelId] = Channel({
            listener: msg.sender,
            artist: artist,
            deposit: msg.value,
            startTime: block.timestamp,
            duration: duration,
            closed: false
        });

        // チャネル開設イベントの発行
        emit ChannelOpened(channelId, msg.sender, artist, msg.value);
    }

    // チャネルの精算
    // アーティストが最新の署名付き支払い状態を提示してチャネルを閉じる
    function closeChannel(
        bytes32 channelId, // 対象のチャネルID
        uint256 amount,  // 支払い額
        bytes memory signature  // リスナーの署名
    ) external nonReentrant {
        // チャネル情報の取得
        Channel storage channel = channels[channelId];

        // チャネルが閉じられていないこと、有効期間が終了していないこと、支払い額がデポジット額以下であることを確認
        require(!channel.closed, "Channel already closed");
        require(block.timestamp <= channel.startTime + channel.duration, "Channel expired");
        require(amount <= channel.deposit, "Amount exceeds deposit");

        // 署名の検証に使用するメッセージハッシュの作成
        bytes32 message = keccak256(abi.encodePacked(channelId, amount));
        bytes32 signedHash = keccak256(
            abi.encodePacked("\x19Ethereum Signed Message:\n32", message)
        );

        // 署名からアドレスを復元して検証
        address signer = signedHash.recover(signature);
        require(signer == channel.listener, "Invalid signature");

        // 支払いの実行(チャネルを閉じる)
        channel.closed = true;
        
        // アーティストへの支払い
        (bool success, ) = payable(channel.artist).transfer(amount);
        require(success, "Artist payment failed");
        
        // 残額をリスナーへ返金
        if (channel.deposit > amount) {
            payable(channel.listener).transfer(channel.deposit - amount);
        }

        // チャネル閉鎖イベントの発行
        emit ChannelClosed(channelId, amount);
    }

    // チャネルの強制クローズ（タイムアウト時）
    // 有効期限が切れた場合、リスナーは預けたETHを回収できる
    function forceClose(bytes32 channelId) external nonReentrant {
        Channel storage channel = channels[channelId]; // 対称のチャネルID

        require(!channel.closed, "Channel already closed");
        require(
            block.timestamp > channel.startTime + channel.duration,
            "Channel not expired"
        );

        // チャネルを閉じてデポジットを返金
        channel.closed = true;
        payable(channel.listener).transfer(channel.deposit);

        // チャネル閉鎖イベントの発行（支払額0として記録）
        emit ChannelClosed(channelId, 0);
    }
}
```

## セキュリティ面について
- OpenZeppelinのReentrancyGuardを使用して再入攻撃を防止

- ECDSAライブラリ（楕円曲線デジタル署名アルゴリズム）を使用して署名の検証を実装

```solidity
contract ModernChannel is ReentrancyGuard {
    using ECDSA for bytes32;
```
