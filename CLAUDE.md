# MyZipViewer — Claude Code 引き継ぎ

## 今の目的

自炊 ZIP ビューア（`viewer3.html`）を、自宅の書庫（MyBookShelf）から開いて読み、外出先では同じスマホ内の「端末の本」から再ダウンロードなしで読めるようにする。GitHub Pages 版と NAS 配信版を揃えて維持する。

## 確定した方針と、却下した案

### 確定

- **外出先のオフライン:** NAS の `http://192.168.1.40/BookFile/viewer3.html` に本を「端末に保存」し、外出時は VPN で同じ URL を開いて「端末の本」から読む。保存先はサイトごとの IndexedDB（`zip-viewer-books`）。
- **書庫連携:** MyBookShelf の「端末に保存」はビューア URL に `&save=1` を付け、開いた直後に自動保存する。
- **シリーズ:** URL hash `#s=` が長すぎて切れることがある。hash に `a`（書庫 origin）と `t`（title id）があれば、ビューアが `/api/title/{id}` から巻一覧を取り直す。
- **ZIP ファイル名:** スキャン ZIP は Shift_JIS 名が多い。`JSZip.loadAsync` に `decodeFileName` で UTF-8 → Shift_JIS の順でデコードする（衝突でページが潰れるのを防ぐ）。
- **HTTP の NAS:** OPFS（`navigator.storage.getDirectory`）は secure context 専用。NAS の `http://` では IndexedDB を使う。

### 却下

- 外出先を GitHub Pages だけにする案（書庫のファイルが `http://localhost:8080` のため Mixed Content で読めない。保存もサイト別）。
- スマホの見えるフォルダを指定して永続アクセスする案（Android Chrome はディレクトリ許可の永続化が未実装）。
- 「読む」だけで自動保存する案（明示の「端末に保存」／`save=1` のみ）。

## 変更済みのファイル

### MyZipViewer（このリポジトリ）

- `viewer3.html` / `index.html` — 閲覧・巻切替・端末保存・PWA・`save=1`・シリーズ API 解決・Shift_JIS 名
- `sw.js` — PWA キャッシュ（現在 `zip-viewer-v15`）
- `manifest.json` / アイコン類

NAS 配信先: `X:\BookFile\viewer3.html`（`http://192.168.1.40/BookFile/viewer3.html`）

### MyBookShelf（別フォルダ `C:\U_App\Dev\MyBookShelf`）

- `public/js/app.js` — 「端末に保存」リンク（`save=1`）。長 hash 対策として `f` を省略し `a`+`t` で渡す場合あり
- `public/css/style.css` — ボタン見た目
- NAS 静的版: `X:\MyBookShelf\js\app.js` / `css\style.css`

## 試して失敗した手順

- GitHub で保存した本を NAS ビューアで見る → サイトが違うので IndexedDB は共有されない。
- NAS `http://` で OPFS 保存 → 「このブラウザでは端末保存に対応していません」。IndexedDB に切替済み。
- ホーム画面 PWA / タブの古いキャッシュのまま → 「端末の本」ボタンが見えない。Ctrl+F5 またはサイトデータ削除が必要。`sw.js` の `CACHE_NAME` を上げる。
- ヘッダーに 36px の safe-area を足す → Android で二重余白。最終的に Android は負の margin、iOS は safe-area のみ。

## 次に手を付ける一点

外出フローの実機確認と、必要なら MyBookShelf 側の hash 短縮（`f` 省略＋`a`/`t`）が本番の「読む／端末に保存」で常に効いているかを確認する。問題があればビューアの `resolveSeries` / 書庫の `viewerLink` を直す。

Claude Code への最初の一文の例:

> MyZipViewer の CLAUDE.md と docs/cursor-handoff.md を読んで続けて。次は外出時 VPN＋端末の本の実機確認と、書庫のシリーズ hash が切れたときの挙動確認。
