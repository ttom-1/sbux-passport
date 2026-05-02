# マイストアパスポート — セットアップ手順書

## ファイル構成

```
my-passport-app/          ← GitHub リポジトリのルート
├── index.html            ← PWA アプリ本体
├── manifest.json         ← PWA マニフェスト
├── sw.js                 ← Service Worker（オフライン対応）
├── stores.json           ← 店舗データ（スクレイパーが生成）
├── icon-192.png          ← アプリアイコン 192x192（別途用意）
├── icon-512.png          ← アプリアイコン 512x512（別途用意）
└── get_stores.py         ← 店舗データ取得スクリプト（ローカル実行用）
```

---

## ステップ1：Python 環境の準備

```bash
pip install selenium chromedriver-binary-sync requests
```

> Chrome ブラウザが PC にインストールされている必要があります。

---

## ステップ2：店舗データの取得（ローカル PC で実行）

```bash
python get_stores.py
```

- 全47都道府県の店舗情報を取得します（初回は数時間かかります）
- 住所から緯度経度を自動変換します（Nominatim API 使用）
- **途中で止まっても安心**：都道府県ごとに `stores.json` を中間保存します
- 再実行すると続きから処理します

### 動作確認（東京のみ）
`get_stores.py` の以下の行を変更して実行：
```python
PREFECTURES = [13]  # 東京のみ
```

---

## ステップ3：GitHub Pages へのデプロイ

### 3-1. GitHub リポジトリ作成
1. GitHub にログインして新しいリポジトリを作成（名前例：`sbux-passport`）
2. Public に設定（GitHub Pages は Public リポジトリなら無料）

### 3-2. ファイルのアップロード
```bash
git clone https://github.com/あなたのユーザー名/sbux-passport.git
cd sbux-passport

# ファイルをコピー
cp path/to/index.html .
cp path/to/manifest.json .
cp path/to/sw.js .
cp path/to/stores.json .
# icon-192.png と icon-512.png も配置

git add .
git commit -m "Initial commit"
git push origin main
```

### 3-3. GitHub Pages を有効化
1. リポジトリの **Settings** → **Pages**
2. **Source** を `main` ブランチ / `/ (root)` に設定
3. **Save**
4. しばらくすると `https://あなたのユーザー名.github.io/sbux-passport/` で公開されます

---

## ステップ4：スマホにインストール（PWA）

### Android（Chrome）
1. スマホの Chrome で上記 URL を開く
2. アドレスバー右の「⋮」→「ホーム画面に追加」
3. アイコンがホーム画面に追加されネイティブアプリ風に起動

### iPhone（Safari）
1. Safari で URL を開く（Chrome では PWA インストール不可）
2. 下の「共有」ボタン →「ホーム画面に追加」
3. ホーム画面にアイコンが追加される

---

## ステップ5：店舗データの定期更新

月1回程度、以下を実行して GitHub に push するだけです：

```bash
# データ再取得（既存データに追記・上書き）
python get_stores.py

# GitHub に push
git add stores.json
git commit -m "Update store data $(date +%Y-%m)"
git push origin main
```

---

## 将来の拡張：ネイティブアプリ化

### React Native（推奨）
- `stores.json` を GitHub Pages でそのまま API として使用可能
- `react-native-maps` + `react-native-leaflet` で地図表示
- App Store / Google Play 公開が可能

### バックエンド（Supabase、無料枠あり）
- スタンプデータをクラウドに保存（複数端末で同期）
- ユーザー認証（スターバックスアカウント連携なども将来検討）

---

## トラブルシューティング

| 問題 | 対処 |
|------|------|
| `stores.json` が空 | `PREFECTURES = [13]`（東京のみ）で動作確認 |
| ジオコーディング失敗が多い | Nominatim の代わりに Google Geocoding API キーを取得して使用 |
| Chrome が起動しない | `HEADLESS = False` にして画面表示で確認 |
| GitHub Pages が表示されない | Settings → Pages で Branch が正しく設定されているか確認 |
| スマホで地図が表示されない | HTTPS でアクセスしているか確認（HTTP では位置情報が使えない） |
