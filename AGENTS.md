# AGENTS.md

## Cursor Cloud specific instructions

### Product

**Zip Viewer Pro Max** — a single-file static web app (`viewer2.html`) for browsing image archives inside ZIP files (manga/comic-style reader). No backend, build step, or package manager in the repository.

### Running the app

From the repo root:

```bash
python3 -m http.server 8080
```

Open `http://localhost:8080/viewer2.html` in a browser.

- **JSZip** loads from `https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js` — outbound HTTPS to cdnjs is required unless you vendor JSZip and change the script tag.
- Serving over HTTP is recommended; `file://` can behave differently across browsers.

Use **tmux** for the dev server so the session survives backgrounding (see Cloud Agent tmux conventions).

### Lint / tests

There is no linter, formatter, or automated test suite in this repo. Validation is manual or browser-based E2E (upload a ZIP, confirm thumbnails and preview).

### Test fixtures

The repo does not ship sample ZIPs. For E2E, create images and zip them, for example:

```bash
mkdir -p fixtures/sample-pages
for i in 1 2 3; do
  ffmpeg -y -f lavfi -i "color=c=0x$(printf '%02x' $((40+i*40)))4080:s=400x600" -frames:v 1 \
    -vf "drawtext=text='Page ${i}':fontsize=48:fontcolor=white:x=(w-text_w)/2:y=(h-text_h)/2" \
    "fixtures/sample-pages/$(printf '%02d' $i)_page.png"
done
cd fixtures && zip -r sample-manga.zip sample-pages/
```

Upload `fixtures/sample-manga.zip` in the app to exercise ZIP parsing, thumbnails, and the preview overlay.

### Optional headless E2E

Headless checks can use `puppeteer-core` with system Chrome (`/usr/local/bin/google-chrome`). Install deps under `/tmp` (not the repo) to avoid adding `package.json` to the project:

```bash
mkdir -p /tmp/zip-viewer-e2e && cd /tmp/zip-viewer-e2e
npm init -y && npm install puppeteer-core
# Then run a script that opens http://127.0.0.1:8080/viewer2.html and uploadFile(sample-manga.zip)
```
