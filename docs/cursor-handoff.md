# Cursor → Claude Code 引き継ぎ全文（MyZipViewer / 書庫オフライン）

このファイルは Cursor チャットの経緯控えです。作業ルールの要約はルートの `CLAUDE.md` を優先してください。Cursor の Export Chat / Export Session で Markdown を書き出した場合は、このファイルを差し替えるか末尾に追記してください。

関連リポジトリ:

| 役割 | パス | 備考 |
|------|------|------|
| ZIP ビューア | `C:\U_App\Dev\MyZipViewer` | GitHub Pages: https://vcmilan1899-cafe.github.io/My-Zip-Viewer/ |
| 書庫 | `C:\U_App\Dev\MyBookShelf` | NAS 静的: `X:\MyBookShelf\`、API は `http://localhost:8080` 想定 |
| NAS ビューア配信 | `X:\BookFile\viewer3.html` | `http://192.168.1.40/BookFile/viewer3.html` |

手元時点のビューア HEAD: `ca2ef0a`（Shift_JIS 名＋シリーズ API 解決）。その直前: `0faa941`（書庫 save 連携＋オフライン棚）、`f2e4f16`（端末保存機能そのもの）。

---

## 1. プロダクト概要

自炊 ZIP（漫画スキャン）をブラウザで読むビューア。

- 技術: 単一 HTML（`viewer3.html`）、JSZip、Service Worker（`sw.js`）、`manifest.json`（PWA）
- 操作: タップでページ送り、横画面は見開き、ヘッダーに進捗・巻切替・Wake Lock・端末の本
- 書庫（MyBookShelf）から `?file=` と任意でシリーズ hash `#s=` 付きで開く

---

## 2. 会話で積み上げた機能（時系列）

### 2.1 閲覧まわり

- 自然順ソート（ページ番号の数値ソート）
- 横画面の見開き（右開き想定）
- 巻切替トースト（「○巻」など短時間表示）
- ヘッダーがステータスバーと重ならないよう調整（最終: Android は負の margin、iOS は `env(safe-area-inset-top)`）
- レジューム（同じ本を開き直すと続きから）
- PWA / Wake Lock

### 2.2 端末保存（オフライン棚）

目的: 自宅で本を端末に置き、外出先で再 DL なしで読む。

実装要点:

- IndexedDB `zip-viewer-books` に ZIP blob を保存（キーは URL / 識別子）
- UI: 「端末に保存」「端末の本」（オフライン棚一覧）
- クエリ `?save=1` で開いた直後に自動保存
- **OPFS は使わない（HTTP NAS）**: `navigator.storage.getDirectory` は secure context 専用。`http://192.168.1.40` では失敗したため IndexedDB のみ。

サイト分離:

- 保存は **オリジンごと**。GitHub Pages に保存した本は NAS ビューアから見えない（逆も同じ）。
- 外出フローは **常に NAS ビューアのオリジン** で保存・再開する。

Mixed Content:

- GitHub Pages（https）から書庫ファイル（`http://localhost:8080/...`）は読めない。
- そのため「外出先は GitHub だけで済ます」案は却下。VPN で自宅 NAS に繋ぎ、同じ `http://192.168.1.40/...` を開く。

### 2.3 書庫側「端末に保存」

`C:\U_App\Dev\MyBookShelf\public\js\app.js`:

- 「読む」に並べて「端末に保存」リンク
- ビューア URL + `&save=1`
- スタイルは `public/css/style.css`
- NAS へは `X:\MyBookShelf\js\app.js` / `css\style.css` にデプロイ済み

### 2.4 シリーズ hash 切れと API 解決

問題: `#s={i,f,t,a}` の `f`（巻 URL 配列）が長く、モバイルで hash が切れて「読む／端末に保存」が壊れることがある。

方針:

- 書庫は `f` を省略し、`a`（API origin）と `t`（title id）を渡す場合がある
- ビューア `resolveSeries` が `/api/config` と `/api/title/{id}` から巻一覧を再構築

関連コミット: `ca2ef0a`

### 2.5 Shift_JIS ZIP 名

スキャン ZIP のエントリ名が Shift_JIS のとき、UTF-8 前提だとページ名が衝突しページ数が潰れる。

- `JSZip.loadAsync` の `decodeFileName` で UTF-8 試行 → だめなら Shift_JIS
- 同じく `ca2ef0a`

### 2.6 Service Worker

- `sw.js` の `CACHE_NAME` を変更のたびに上げる（手元は `zip-viewer-v15`）
- 古いキャッシュだと新 UI（端末の本など）が見えない → Ctrl+F5 / サイトデータ削除

---

## 3. 確定した外出フロー（ユーザー合意）

1. 自宅 Wi‑Fi で書庫を開く
2. 読みたい本の「端末に保存」→ NAS ビューアが開き `save=1` で IndexedDB に保存
3. 外出先で VPN 接続
4. 同じ `http://192.168.1.40/BookFile/viewer3.html` を開く
5. 「端末の本」から開く（再ダウンロードなし）

補足:

- スマホの「見えるフォルダ」指定は Android Chrome の制限で却下
- ホーム画面に追加した PWA も同じオリジンなら同じ IndexedDB

---

## 4. 環境・URL メモ

- ローカル開発: `C:\U_App\Dev\MyZipViewer`
- GitHub: `https://github.com/vcmilan1899-cafe/My-Zip-Viewer.git` / Pages 上記
- NAS ビューア: `http://192.168.1.40/BookFile/viewer3.html` ← 実ファイル `X:\BookFile\viewer3.html`
- 書庫 API: だいたい `http://localhost:8080`（ビューアのシリーズ解決や書庫リンクの `a`）
- 書庫フロント NAS: `http://192.168.1.40/...`（`X:\MyBookShelf\`）

`index.html` は GitHub Pages 用。変更時は **必ず `viewer3.html` と同一内容に同期**してからコミットする。

---

## 5. 触った主なコード位置（ビューア）

ファイル: `viewer3.html`（巨大な単一 HTML）

おおよその関心事:

- ZIP 読込 / `decodeFileName` / ページリスト構築
- シリーズ hash パースと `resolveSeries`（API）
- IndexedDB 保存・一覧・`save=1`
- ヘッダー UI・巻トースト・見開き・Wake Lock
- Service Worker 登録

`sw.js`: precache 対象と `CACHE_NAME`

---

## 6. 書庫側メモ

- MyBookShelf は別 git リポジトリ。カバー／背表紙検出の作業用 `CLAUDE.md` が書庫ルートにある場合あり（ビューア作業とは別トラック）。
- ビューア連携で重要なのは `viewerLink` / `saveView`（`save=1`）と、必要なら hash 短縮。

---

## 7. 失敗・ハマり一覧（詳細）

| 現象 | 原因 | 対処 |
|------|------|------|
| 「端末保存に対応していません」 | HTTP で OPFS | IndexedDB のみ |
| 「端末の本」が出ない | 古い SW / 別オリジン | キャッシュ削除、CACHE_NAME 更新、正しい URL |
| GitHub で保存した本が NAS にない | オリジン分離 | 保存も再開も NAS URL |
| https ビューアで書庫の本が開けない | Mixed Content | NAS http ビューアを使う |
| 巻が足りない／読めない | hash 切れ | `a`+`t` と API resolve |
| ページが少ない | SJIS 名の衝突 | decodeFileName |
| ヘッダー二重余白 | Android + safe-area 過剰 | 負 margin 調整 |

---

## 8. Cursor 作業ルールのうち Claude に必要なもの

`.cursor/rules` は Claude Code から見えない想定。要点だけ:

- ユーザーが頼んでいないリファクタやドキュメント増殖をしない
- コミットはユーザーが依頼したときだけ（今回の引き継ぎでは状態固定のためコミット済み）
- `index.html` と `viewer3.html` を食い違わせない
- NAS 更新時は `X:\BookFile\viewer3.html` と、書庫なら `X:\MyBookShelf\` を忘れない
- 破壊的 git（force push 等）はしない

---

## 9. 次にやること（優先順）

1. **実機:** 自宅で「端末に保存」→ VPN 切断/再接続想定 → 「端末の本」で開けるか
2. **実機:** 巻数の多いタイトルで「読む／端末に保存」が切れないか（hash / API）
3. 問題があれば `resolveSeries` または書庫の `viewerLink` を修正
4. 修正したら `viewer3.html` → `index.html` 同期 → `sw.js` の CACHE_NAME 上げ → GitHub push → `X:\BookFile\viewer3.html` コピー

---

## 10. Claude Code 起動時

Windows の Claude Code で `C:\U_App\Dev\MyZipViewer` を開く。最初のメッセージ例:

```
MyZipViewer の CLAUDE.md と docs/cursor-handoff.md を読んで続けて。
次は外出時 VPN＋端末の本の実機確認と、書庫のシリーズ hash が切れたときの挙動確認。
```

書庫を触るときは同じマシンの `C:\U_App\Dev\MyBookShelf` も開くか、パスを明示する。

---

## 11. Export Chat について

Cursor チャット画面のメニューから **Export Chat**（Composer なら **Export Session**）で Markdown を書き出し、このパスに上書きまたは追記すると、この要約より細かいやりとりが残ります。要約と矛盾する場合は Export 側の直近の合意を優先してください。
