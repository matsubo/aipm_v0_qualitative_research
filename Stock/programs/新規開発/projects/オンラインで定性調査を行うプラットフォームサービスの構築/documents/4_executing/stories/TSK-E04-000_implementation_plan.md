---
doc_type: story_implementation
project_id: オンラインで定性調査を行うプラットフォームサービスの構築
story_id: TSK-E04-000
created_at: 2025-05-09
version: v1.0
---

# TSK-E04-000: WebRTCを利用したリアルタイムビデオ・音声通話機能の技術検証 (PoC) 実装計画

## 1. ストーリー概要

オンラインインタビュー機能のコアとなるWebRTCを用いたビデオ・音声通話について、実現可能性、パフォーマンス、必要なライブラリ/サービスを検証する。

## 2. 受入条件

- 基本的な1対1のビデオ・音声通信がブラウザ間で確立できること
- 遅延や品質について基本的な評価が行えること
- 利用する技術スタック（ライブラリ、STUN/TURNサーバー等）の目処が立つこと
- 主要ブラウザ（Chrome, Firefox, Safari）での動作互換性を確認する

## 3. 技術要素

- **実装言語/フレームワーク**: CTOによる最終決定待ち。PoCのため、検証に適した技術（例: JavaScript, Node.js, WebRTCライブラリ like PeerJS, SimplePeerなど）を柔軟に選択。
- **関連ドキュメント/コード**:
    - `Flow/202505/2025-05-09/development/draft_development_plan.md`
    - `Flow/202505/2025-05-09/draft_setup.md`
    - `Stock/programs/新規開発/projects/オンラインで定性調査を行うプラットフォームサービスの構築/documents/3_planning/backlog/backlog.yaml`
- **制約事項**:
    - CTOによる最終的な技術スタックの決定待ち
    - セキュリティ要件の詳細（特にデータ保護やアクセス制御）は別途確認が必要
    - ローカル開発環境の標準化方針（Docker利用有無など）は別途決定が必要
    - WebRTC関連ライブラリ/サービスを利用予定

## 4. 実装計画

### 実装ステップ

1.  **環境準備**:
    *   検証用のシンプルなフロントエンド環境と、必要であればシグナリングサーバー用のバックエンド環境を準備する。(例: HTML/JS, Node.js)
    *   必要なライブラリ (WebRTCクライアントライブラリ、シグナリング用WebSocketライブラリ等) を選定・導入する。
    *   STUN/TURNサーバーの準備（パブリックなものを利用するか、ローカルで簡易的なものを立てるか検討）。

2.  **テスト計画**:
    *   1対1のビデオ通話確立テスト。
    *   音声通話確立テスト。
    *   基本的な遅延測定（RTTなど）。
    *   主観的なビデオ・音声品質評価。
    *   Chrome, Firefox, Safariでの基本的な接続テスト。

3.  **コア機能実装 (PoC)**:
    *   **シグナリング処理**: ピア間の接続情報を交換するためのシグナリングロジックを実装する (WebSocket等を利用)。
        *   Offer/Answer SDP交換
        *   ICE Candidate交換
    *   **メディア取得**: `getUserMedia` を使用してローカルのカメラとマイクのメディアストリームを取得する。
    *   **ピアコネクション確立**: `RTCPeerConnection` を使用してピア間の接続を確立する。
        *   ローカルストリームをピアコネクションに追加。
        *   リモートストリームを受信して表示。
    *   **UI**: 非常にシンプルなUIで、ローカルビデオ、リモートビデオを表示し、接続開始・切断ボタンを配置する。

4.  **評価・検証**:
    *   作成したPoCを用いて、受入条件に基づき検証を行う。
    *   技術的な課題、パフォーマンス上のボトルネック、ライブラリの使い勝手などを記録する。
    *   STUN/TURNサーバーの必要性や設定について評価する。

### 実装コード (概念)

```javascript
// client.js (ブラウザ側 - 概念)

// シグナリングサーバーへの接続 (例: WebSocket)
const socket = io();

const localVideo = document.getElementById('localVideo');
const remoteVideo = document.getElementById('remoteVideo');
let localStream;
let peerConnection;

// 1. メディア取得
navigator.mediaDevices.getUserMedia({ video: true, audio: true })
    .then(stream => {
        localVideo.srcObject = stream;
        localStream = stream;
    });

// 2. RTCPeerConnectionの作成とイベントハンドラ設定
function createPeerConnection() {
    peerConnection = new RTCPeerConnection({
        iceServers: [{ urls: 'stun:stun.l.google.com:19302' }] // STUNサーバー
    });

    localStream.getTracks().forEach(track => peerConnection.addTrack(track, localStream));

    peerConnection.ontrack = event => {
        remoteVideo.srcObject = event.streams[0];
    };

    peerConnection.onicecandidate = event => {
        if (event.candidate) {
            socket.emit('ice-candidate', event.candidate);
        }
    };
    // ...その他のイベントハンドラ
}

// 3. シグナリング処理 (Offer/Answer, ICE Candidate)
// socket.on('offer', async sdpOffer => { /* ... */ });
// socket.on('answer', async sdpAnswer => { /* ... */ });
// socket.on('ice-candidate', async candidate => { /* ... */ });

// UIからのアクションでOffer作成・送信など
// document.getElementById('callButton').onclick = async () => {
//     createPeerConnection();
//     const offer = await peerConnection.createOffer();
//     await peerConnection.setLocalDescription(offer);
//     socket.emit('offer', offer);
// };
```

```javascript
// server.js (シグナリングサーバー - 概念 - Node.js + Socket.IO)
// const io = require('socket.io')(httpServer);
// io.on('connection', socket => {
//     socket.on('offer', offer => {
//         socket.broadcast.emit('offer', offer); // 簡易的なブロードキャスト
//     });
//     socket.on('answer', answer => {
//         socket.broadcast.emit('answer', answer);
//     });
//     socket.on('ice-candidate', candidate => {
//         socket.broadcast.emit('ice-candidate', candidate);
//     });
// });
```

## 5. 実装結果

*実装完了後に記入*

### 成果物の場所
- コード: `Stock/programs/新規開発/projects/オンラインで定性調査を行うプラットフォームサービスの構築/development/code/poc/webrtc/` (想定)
- 検証レポート: `Stock/programs/新規開発/projects/オンラインで定性調査を行うプラットフォームサービスの構築/development/articles/poc_webrtc_report.md` (想定)

### 動作確認結果
- [テスト結果をここに記述]

## 6. 次のステップ

1. PoCの結果をCTOと共有し、技術スタック選定の判断材料とする。
2. 検証で得られた知見を元に、本番機能の設計・実装計画を詳細化する。
3. 次のストーリー (US-E01-001: アカウント登録) の実装準備を開始する。
