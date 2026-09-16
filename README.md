# 起きたら○○

終電・始発で寝過ごした場合に、乗り換えなしでどこまで運ばれるかを表示するツール。
首都圏 34 路線・316 駅を内蔵データとして収録している。

## ファイル構成

| ファイル | 役割 |
|---|---|
| `index.html` | **アプリ本体（ポップ版）**。公開時に表示されるのはこれ |
| `dark.html` | 別デザイン（ダーク版）。機能は同じ。不要なら削除してよい |
| `ogp.png` | LINE などに貼ったときのプレビュー画像（1200×630・ポップ版） |
| `ogp.svg` | 上記の編集用ソース。`rsvg-convert -w 1200 -h 630 -o ogp.png ogp.svg` で再生成 |
| `ogp-dark.png` / `ogp-dark.svg` | ダーク版用のプレビュー画像。`dark.html` を使わないなら削除してよい |
| `README.md` | このファイル |

`index.html` と `dark.html` は **CSS だけが違う同じアプリ**。データもロジックも同一で、
どちらも HTML 1 枚で動く。片方のデータを直した場合、もう片方にも同じ修正が必要になる。

### 外部通信

| 接続先 | 用途 | タイミング |
|---|---|---|
| `fonts.googleapis.com` / `fonts.gstatic.com` | Web フォント | ページ表示時 |
| `cdnjs.cloudflare.com` | 地図ライブラリ Leaflet 1.9.4 | ページ表示時 |
| `cyberjapandata.gsi.go.jp` | 国土地理院の地図タイル | 「地図で見る」を押したときだけ |

API キーは不要。地図タイルは押されたカードの分だけ取得する。

## 動かし方（ローカル）

`index.html` をブラウザにドラッグするだけで動く。サーバーは不要。

## GitHub Pages で公開する手順（個人 PC 用）

### 方法 A: ブラウザだけで完結（GitHub に不慣れでもこちらが早い）

1. GitHub にログインし、右上の **+ → New repository**
2. Repository name に `okitara` と入力
3. **Public** を選択 → **Create repository**
4. 次の画面の **uploading an existing file** をクリック
5. フォルダ内のファイルをまとめてドラッグ＆ドロップ → **Commit changes**
   （最低限必要なのは `index.html` と `ogp.png` の 2 つ）
6. リポジトリの **Settings → Pages** を開く
7. Source を **Deploy from a branch**、Branch を **main / (root)** にして **Save**
8. 1〜2 分待つと、同じ画面に公開 URL が表示される

```
https://<ユーザー名>.github.io/okitara/
```

### 方法 B: gh CLI

```bash
cd okitara
git init
git add .
git commit -m "起きたら○○"
gh repo create okitara --public --source=. --push
gh api -X POST repos/:owner/okitara/pages -f 'source[branch]=main' -f 'source[path]=/'
```

### 公開後の確認

- 発行された URL をスマホで開く
- LINE に URL を貼ると `ogp.png` のプレビューが出る（初回は数分かかることがある）
- 相手側は **GitHub アカウント不要・ログイン不要**。ブラウザだけで開ける

### ホーム画面への追加（配布先に伝える文面）

- **iPhone（Safari）**: 下部の共有ボタン（□に↑）→「ホーム画面に追加」
- **Android（Chrome）**: 右上の ⋮ →「ホーム画面に追加」

## 注意事項

- URL を知っていれば誰でも閲覧できる（パスワードはかけられない）
- 無料プランでは Public リポジトリが必要なため、`index.html` の中身は誰でも読める
- 距離・料金は直線距離をもとにした概算。駅の緯度経度も概算値
- 地図に引く線も直線。実際の線路や道路の経路ではない
- 終着駅・直通先・深夜の途中止まりは代表的なパターンのみ。実際の運行はダイヤ改正で変わる

## 地図について

行き先カードの「地図で見る」で、自宅と終着駅を地図上に表示する。
「Googleマップ」ボタンは自宅からの経路を Google マップで開く（アプリ内には表示しない）。

- 地図タイルは **国土地理院**提供。出典表示を消さないこと（利用規約上の条件）
- 出典リンクは地図右下に自動表示される
- 地図は押したときに初めて読み込む。開かなければ通信は発生しない

## データを直したいとき

`index.html` 内の 2 つの定数を編集する。

- `const P = {...}` … 駅名 → `[緯度, 経度]`
- `const LINES = [...]` … 路線定義
  - `st` … 駅の並び。配列の先頭側が「方向 0」、末尾側が「方向 1」
  - `ends[0]` / `ends[1]` … 各方向の終着情報
    - `dest` … そのまま乗り続けた場合に着く駅（`n` 駅名 / `pref` 県 / `city` 市区町村 / `via` 説明）
    - `turn` … 終電間際の折り返し・途中止まりの説明
  - `loop: true` … 環状線。`ends` の代わりに `loopNote` と `loopEnds` を使う
