# Weather Viewer

Vue.js で構築したシンプルな天気表示アプリです。  
OpenWeatherMap API を利用して、都市ごとの現在の天気、最高・最低気温、気圧などをリアルタイムで表示します。

[こちらから実際の動作を確認できます](https://yoku-094.github.io/weather-app/)

---

## 使用技術

- Vue.js 3.2.40（Options API を使用）
- Axios（HTTP 通信に利用）
- OpenWeatherMap API（天気情報の取得）
- HTML / CSS（Scoped Style でコンポーネント単位のスタイリング）
- Bootstrap 5（コンポーネントのスタイリング・レイアウト）
- JavaScript（セッションストレージによる状態管理）

---

## 機能一覧と実装概要

### 1. 都市選択機能

- `city-list.json` に登録された全国主要都市をプルダウンで選択可能
- セッションストレージを利用し、再読み込み後も直前に選択した都市を保持

### 2. 現在の天気情報取得・表示

- OpenWeatherMap API を用いて選択都市の天気をリアルタイム取得
- 天気概要、気温（最高・最低）、気圧などをわかりやすく表示

### 3. 天気アイコンの動的切替え

- 晴れ・曇り・雨・雪など天気の状態に応じて独自作成のアイコンを自動表示

### 4. ローディング表示による UX 向上

- 天気情報の取得中はローディング画面を表示し、画面のちらつきや空白状態を防止

### 5. 表示名称の補正機能

- API から返される不正確な都市名を、`display` プロパティで正確な日本語名称に補正表示

### 6. 週間予報の詳細表示

- OpenWeatherMap の ４ 日間・3 時間ごとの予報データを取得し、日付・時間別に気温・天気・気圧をテーブル形式で表示

### 7. 現在地取得連動表示

- ブラウザの Geolocation API でユーザーの現在地を取得し、逆ジオコーディングで地名を特定
- 国土地理院 API を利用し都道府県コードを判定、連動して適切な都市名を選択
- 現在地のリアルタイム天気と週間予報を API から取得し表示

---

## セットアップ手順

以下の手順でローカル環境にセットアップできます。

### 1. リポジトリをクローン

```bash
git clone https://github.com/your-username/weather-viewer.git
cd weather-viewer
```

### 2. パッケージのインストール

```bash
npm install
```

### 3. 開発用サーバーを起動

```bash
npm run serve
```

### 4. ブラウザでアクセス

```
http://localhost:8080/
```

### 5. 本番用にビルド

```bash
npm run build
```

### 6. コードのリントと自動修正

```bash
npm run lint
```

### 7. カスタマイズ設定

詳細は[Vue CLI Configuration Reference](https://cli.vuejs.org/config/)を参照してください。

---

## ディレクトリ構成（抜粋）

```
.
├── babel.config.js
├── docs/                        # GitHub Pages 公開用のビルド成果物
│   ├── css/
│   ├── favicon.ico
│   ├── img/
│   ├── index.html
│   └── js/
├── package-lock.json
├── package.json
├── public/
│   ├── favicon.ico
│   └── index.html
├── src/
│   ├── App.vue                  # アプリ全体のルートコンポーネント
│   ├── assets/
│   │   ├── css/
│   │   │   └── global.css       # 全体共通のレイアウト
│   │   ├── data/
│   │   │   └── city-list.json   # 都市名リスト（API名・表示名）
│   │   ├── logo_Cloud.png
│   │   ├── logo_Other.png
│   │   ├── logo_Rain.png
│   │   ├── logo_Snow.png
│   │   └── logo_Sunny.png
│   ├── components/
│   │   └── WeatherTop.vue       # メイン天気表示コンポーネント
│   └── main.js                  # アプリのエントリーポイント
└── vue.config.js                # ビルドや公開先設定など
```

---

## 今後の改善予定

- **グラフ表示の導入**  
  Chart.js などで気温・気圧のグラフ化

---

## ライセンス

MIT License（自由に利用・改変 OK です）

---
