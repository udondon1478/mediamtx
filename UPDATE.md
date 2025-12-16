# MediaMTX カスタムビルド - アップデート手順

## 概要
VRChat の RTSP 60秒タイムアウト問題を解決するため、`IdleTimeout` を 120秒に延長したカスタムビルドを使用しています。

## カスタム変更点
| ファイル | 変更内容 |
|---------|---------|
| `internal/servers/rtsp/server.go` | `IdleTimeout: 120 * time.Second` を追加 |

---

## 更新手順

### 1. ソースコードを最新に更新
```bash
cd /home/ubuntu/mediamtx-build/mediamtx
git fetch origin
git stash  # ローカル変更を一時保存
git pull origin main
git stash pop  # ローカル変更を復元
```

### 2. 競合があった場合
```bash
# server.go の変更を手動で再適用
# 以下の行を s.srv = &gortsplib.Server{ ブロック内に追加:
#     IdleTimeout: 120 * time.Second,
```

### 3. ビルド
```bash
cd /home/ubuntu/mediamtx-build/mediamtx
CGO_ENABLED=0 go build -o mediamtx .
```

### 4. デプロイ
```bash
sudo systemctl stop mediamtx
sudo cp /usr/local/bin/mediamtx /usr/local/bin/mediamtx.backup
sudo cp /home/ubuntu/mediamtx-build/mediamtx/mediamtx /usr/local/bin/mediamtx
sudo chmod +x /usr/local/bin/mediamtx
sudo systemctl start mediamtx
```

### 5. 確認
```bash
mediamtx --help  # バージョン確認
sudo systemctl status mediamtx
```

---

## ロールバック
```bash
sudo systemctl stop mediamtx
sudo mv /usr/local/bin/mediamtx.backup /usr/local/bin/mediamtx
sudo systemctl start mediamtx
```

---

## ファイル構成
```
/home/ubuntu/mediamtx-build/
└── mediamtx/           # ソースコード（Git管理）
    └── internal/servers/rtsp/server.go  # カスタム変更

/usr/local/bin/
├── mediamtx            # 現在のバイナリ
└── mediamtx.backup     # 前のバージョン

/usr/local/etc/
└── mediamtx.yml        # 設定ファイル
```
