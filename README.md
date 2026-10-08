# PrintCraft: unofficial browser experience

Community mirror of the official PrintCraft v0.2.1 web build, intended to make it easier to try without installing. PrintCraft is made by ArtCraft Team + contributors; this mirror is not operated or endorsed by them.

[Original source](https://github.com/storytold/printcraft) · [Official website](https://getartcraft.com/apps/printcraft) · [Official desktop downloads](https://github.com/storytold/printcraft/releases/latest)

## Build and hosting

Node.js 24: `node scripts/build-site.cjs` previews the build; `node scripts/build-site.cjs --write` creates a fresh `_site/` and refuses existing output. Serve it with `python -m http.server 8765 --bind 127.0.0.1 --directory _site`; open http://localhost:8765/. Source-root previews use the original single-file download. GitHub Pages builds through `.github/workflows/pages.yml`; generated chunks never enter Git.

`app/` keeps the official release files byte-for-byte, with supplemental upstream license notices. The builder pins the Wasm size/SHA-256, gzip-compresses it into content-addressed 512 KiB parts and verifies exact reconstruction. The service worker downloads a bounded window of four parts, retries each failed part once, checks each part's hash and streams decompression. EOF is withheld until the entire original Wasm passes size/SHA-256 verification; only verified bytes enter the application cache. Settings live in `delivery-config.js`.

The original binary remains available for browsers without decompression support. Unsupported/blocked service workers use the official loader. Cache/storage failure is reported without blocking the editor; browser cache eviction can require another download. Percentage measures downloaded compressed bytes, not compilation or GPU startup. This is not a full offline app; additional concurrency does not guarantee higher throughput.

The independent hosting page provides upstream/download links, transparent credits, a collapsible toolbar and a dismissible notice. English is the default; a first preferred browser language starting with `zh` uses Traditional Chinese, including zh-CN/zh-Hans. Choices persist locally when allowed. The original editor language is unchanged. The shell uses the upstream v0.2.1 default Light palette; editor theme changes do not alter it.

Below 900 CSS pixels, the editor fits a 960 CSS-pixel virtual viewport; 50%/75%/100% overrides persist. Landscape is preferable. This changes display scale, not touch support or feature availability. The official module reads `?file=URL` from the main page; the PDF URL must be same-origin or permit CORS. Readiness requires eframe's initialized canvas and explicit nonzero dimensions, rather than HTML presence alone.

## Web and desktop differences

Static/configuration evidence from the v0.2.1 source; this is not comprehensive runtime parity proof:

| Area | Browser behavior | Upstream source |
|---|---|---|
| Open and save | Async file picker/drop; Save downloads a new PDF instead of overwriting the source | `crates/ui-egui/src/lib.rs:open_dialog`; `crates/ui-egui/src/editing.rs:save_view`, `download` |
| Unfinished web operations | Image insertion/replacement, custom image stamps, attachment downloads, form-data import and spreadsheet-merge pickers have desktop-only paths or web placeholders | `crates/ui-egui/src/content_ui.rs`; `stamps_ui.rs`; `lib.rs:attachment_action`; `files.rs` |
| Recovery | Some preferences persist, but the web build has no document recovery store | `apps/printcraft-web/src/main.rs`; `crates/ui-egui/src/recovery.rs`; `crates/ui-egui/src/lib.rs:persist` |
| OCR | Unavailable in this mirror: model loading uses filesystem paths, not browser fetch; no independent OCR add-on is included | `crates/ocr/src/lib.rs:Models::find`; `crates/ui-egui/src/ocr_ui.rs` |
| Rendering and integration | WebGL2 required; browser memory limits apply; desktop TCP/MCP and filesystem integration are separate | `apps/printcraft-web/src/main.rs`; `crates/ui-egui/src/actions_ui.rs` |

Save explicitly, retain original files and evaluate the official desktop app separately. Browser limitations do not imply the same limitations on desktop; PrintCraft itself remains alpha.

The upstream file picker queues bytes without requesting a repaint; pointer input in the editor wakes the next frame. If a selected PDF does not appear immediately, move the pointer over the editor or tap it. The host preserves this official behavior rather than patching its binary.

## Provenance and licenses

Official release: https://github.com/storytold/printcraft/releases/tag/v0.2.1. The downloaded ZIP is checked against the release's `SHA256SUMS.txt`; extracted original files are recorded in `upstream-files.json` and verified during every build. Packaging and the independent host do not modify those files. This verifies consistency with the published release, not an independent author-signature chain.

PrintCraft is MIT OR Apache-2.0 at your option; preserve LICENSE-MIT, LICENSE-APACHE, [NOTICE](app/NOTICE), [ATTRIBUTION.md](app/ATTRIBUTION.md) and accompanying asset-license texts. Host code adapted from the PhotoCraft community mirror is MIT. ArtCraft marks have [separate terms](app/docs/brand/LICENSE-brand.txt); the host uses plain text attribution and no extracted brand logos. No complete transitive dependency-license audit is claimed.

Validation: `node --test tests/*.test.cjs` checks delivery failures, retries, integrity, caching and the generated package. These are unit/selftest gates; separate isolated-browser checks cover actual PDF import/save, mobile layout and cold/warm startup.

## Single-page hosting and updates

The community bootstrap in `app-loader.js` mounts the official canvas in the main document; no iframe is created. Original JS/Wasm are verified against `upstream-files.json`. The host, loader and layout are community code, not an official build or endorsement. Keep upstream copyright, licenses, NOTICE and third-party attributions; do not extract ArtCraft brand marks into the host.

Wasm and compressed parts are ignored. CI downloads the exact archive pinned by URL + SHA-256, restores the verified Wasm, generates a fresh Pages artifact, and deploys it without committing binaries. WordCraft previews must be archived as GitHub Release assets because upstream CI artifacts expire. Browser asset caches retire prior Wasm hashes on service-worker activation. Existing Git history is not rewritten by this change.

1. Select an explicit official web ZIP and verify its published SHA-256; never silently follow latest.
2. Run `node scripts/update-upstream.cjs --archive HTTPS_ZIP_URL --sha256 SHA256 --version VERSION` to inspect the update, then repeat with `--write`. The command rejects changed archive/bootstrap contracts; review upstream licensing, supplemental notices and web/desktop differences.
3. Run `node scripts/build-site.cjs --output _site-review --write` and `node --test tests/*.test.cjs`; perform fresh-browser import/edit/export, mobile and renderer-failure checks.
4. Commit only source, configuration and provenance metadata, then push. Pages builds its current version from scratch; neither the old nor the new Wasm/parts enter Git.

For a clean checkout, `node scripts/update-upstream.cjs --restore` restores only the pinned Wasm. `--restore --archive LOCAL_ZIP` accepts a local copy with the same pinned archive hash.
