# NOTES

## 2026-07-11 見積書・請求書作成アプリの初回作成

### 実行したプロンプト（全部）

1. 「CLAUDE.mdを作りそこに下記の内容を書き込んでください。（変更履歴の記録ルール／commit・push前の安全確認の全文）」
2. 「見積もりが作れるアプリ作って。請求書変換を押したらすぐに請求書に変換できる様にして。自社情報入力できる様にしておいて。テンプレを用意。商品項目は２０個サンプル作って。デザイン業の場合で。シンプルでみやすく。PDF保存できるように。情報はno sqlに保存。すぐ見れるようにして。ユーザーだけが知ってるパスワードを設定しておいて。HTMLで」
3. 「git add .」
4. 「git commit -m"first commit"」

### 実装方式

- **構成**: index.html 1ファイル完結（HTML + CSS + Vanilla JS、外部ライブラリなし）。ローカルで `open index.html` するだけで動く
- **データ保存（NoSQL）**: ブラウザのlocalStorageをキー・バリュー型ドキュメントストアとして使用
  - `mitsumori.docs` … 書類コレクション（JSON配列、新しいものが先頭）
  - `mitsumori.company` … 自社情報
  - `mitsumori.pw` … パスワードのSHA-256ハッシュ（`mitsumori::` プレフィックス付きでハッシュ化。平文は保存しない）
- **パスワードロック**: 初回起動時に設定（4文字以上・確認入力あり）。`crypto.subtle.digest("SHA-256")` を使用、非対応環境はdjb2ハッシュにフォールバック。セッション中は sessionStorage `mitsumori.unlocked` で解除状態を維持。ヘッダーの「🔒 ロック」で再ロック。自社情報タブから変更可能
- **書類モデル**: `{ id, type: 'quote'|'invoice', number, date, due, client, honorific, subject, taxRate, items:[{name, qty, unit, price}], notes, createdAt, updatedAt, sourceQuote }`
- **採番**: 見積 `Q-YYYYMMDD-001`、請求 `I-YYYYMMDD-001`（同日連番）
- **請求書変換**: 元の見積書を保存したまま複製 → type=invoice、新番号採番、発行日=当日、支払期限=翌月末、備考を請求用に差し替え、振込先ボックスをプレビューに表示。`sourceQuote` に元見積番号を保持
- **テンプレ**: デザイン業向け20項目（ロゴ80,000／名刺15,000／チラシA4片面30,000／Webトップ100,000／バナー8,000／ディレクション費50,000 など）をJS内の `TEMPLATES` 配列に定義。プルダウン選択→追加。先頭の空行があれば置き換え
- **PDF保存**: `window.print()` + `@media print` でプレビュー（A4風 `.doc-sheet`）だけを印刷。`@page { size: A4; margin: 15mm }`。ブラウザの印刷ダイアログで「PDFとして保存」を選ぶ方式
- **UI**: 3タブ構成（書類作成／保存済み一覧／自社情報）。作成画面は左フォーム・右A4プレビューの2カラム（1100px以下で縦積み）。入力は即時プレビュー反映。削除は2度押し確認（confirm()等のブラウザダイアログは不使用、トースト通知）
- **消費税**: 10%／8%／0% 選択可。端数は切り捨て（Math.floor）

### 検証手順

- `open index.html` でブラウザ起動を確認（macOS）
- 動作確認ポイント: 初回パスワード設定 → 自社情報入力 → テンプレから明細追加 → 保存 → 一覧表示 → 請求書に変換 → PDF保存（印刷ダイアログ）

### 制約・注意

- データはブラウザ（そのPC・そのブラウザ）内のみ。別端末とは共有されない。ブラウザのサイトデータ消去で書類も消える
- パスワードはクライアントサイドのロックであり、localStorageを直接見れば書類データ自体は読める（暗号化はしていない）

### git

- コミット: e81a2c6 "first commit"（CLAUDE.md, index.html）
- コミット前確認: `git status` / `git diff --staged --stat` で対象2ファイルのみ・秘密情報なしを確認済み
- リモート: `git remote add origin https://github.com/minaco6311/mitumori_app.git` → `git push -u origin main` でpush済み（mainがorigin/mainを追跡）
