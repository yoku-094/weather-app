# Weather Viewer

Vue.js で構築したシンプルな天気表示アプリです。  
OpenWeatherMap API を利用して、都市ごとの現在の天気、最高・最低気温、気圧などをリアルタイムで表示します。  
フロントエンド学習用として作成し、今後のポートフォリオ拡張に向けて開発を継続中です。

---

## 機能一覧と実装概要

### 1. 都市の選択機能
- `city-list.json` に定義された都市名をプルダウンで選択可能
- セッションストレージにより、再読み込み後も最後に選択した都市を保持

### 2. 天気情報の取得・表示
- OpenWeatherMap API を使用して選択都市の天気を取得
- 天気概要、気温（最高・最低）、気圧などの情報をリアルタイム表示

### 3. 天気アイコンの切り替え表示
- 天気の状態（晴れ・曇り・雨・その他）に応じてアイコンを変更
- 独自に用意した画像（`assets/`）を表示

### 4. ローディング対応
- 天気情報の取得中にローディング用の空白画面を表示し、チラつきを防止

### 5. 表示名称の補正
- APIからの都市名が不正確なケースに対応し、表示用名称（例：新宿、札幌）を `display` プロパティで調整

---

## セットアップ手順

以下の手順でローカル環境にセットアップできます。

### 1. リポジトリをクローン
```bash
git clone https://github.com/your-username/weather-viewer.git
cd weather-viewer
````

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

## 使用技術

- Vue.js 3.2.40（Options API を使用）
- Axios（HTTP通信に利用）
- OpenWeatherMap API（天気情報の取得）
- HTML / CSS（Scoped Styleでコンポーネント単位のスタイリング）
- JavaScript（セッションストレージによる状態管理）

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

- **現在地の天気取得**  
  Geolocation API を使ってユーザーの現在地の天気を表示

- **週間予報の表示**  
  日ごとの天気や気温の推移を取得・表示

- **グラフ表示の導入**  
  Chart.js などで気温・気圧のグラフ化

- **お気に入り都市登録機能**  
  よく見る都市をブックマークとして保存・切替

- **天気に応じた背景切替**  
  晴れ・雨・夜など、天気に応じた背景スタイル変更

- **PWA対応**  
  スマホやオフライン環境でも快適に使えるように最適化

- **UI/UX改善**  
  レスポンシブ対応、アクセシビリティ考慮などデザイン面の強化

---

## ライセンス

MIT License（自由に利用・改変OKです）

---
