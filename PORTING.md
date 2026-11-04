# PORTING.md — e-gov.info を他プロジェクトへお引越しする際のガイド

このファイル1枚で、本リポジトリの実装パターンを他プロジェクトへ移設できる
ことを目指す(`aruaru-tokyo`/`karu.tokyo`と同じ慣行)。

## 1. GitHub README/CLAUDE.md/PORTING.md 自動クロール・GitHub風表示

`src/github_viewer.rs` に、指定した1つのGitHubリポジトリの
`README.md`/`CLAUDE.md`/`PORTING.md`を`raw.githubusercontent.com`経由で
取得し、`pulldown-cmark`でGitHub風にレンダリングして表示する機能一式を
まとめている。`aruaru-tokyo`の複数リポジトリ切替版(`fetch_repo_file`/
`markdown_to_github_style_html`/`markdown_to_rs`)を単一リポジトリ固定
(`GITHUB_ORG`/`GITHUB_REPO`定数)に簡略化したもの。

移植手順:
1. `src/github_viewer.rs`をそのままコピーする。
2. `GITHUB_ORG`/`GITHUB_REPO`定数を移設先の値に書き換える。
3. `Cargo.toml`に`reqwest`(json, rustls-tls)・`pulldown-cmark`・
   `serde_json`を追加する。
4. 呼び出し側(`main.rs`)で`github_viewer::render_repo_viewer().await`を
   呼び、返ってきたHTML文字列をページに埋め込む。CSSも
   `github_viewer::VIEW_TOGGLE_CSS`定数をコピーする。

## 2. 定期自動調査(日本語・英語、Google/GitHub)

`src/research.rs`。GitHub Search APIは実際に叩いて結果を取得し、Googleは
検索リンクの自動生成に留める(APIキー不要な範囲で自動化する設計判断)。
`TOPICS`定数の日英ペアを書き換えるだけで、他プロジェクトのテーマにも
転用できる。

## 3. 定期自動マーケティングドラフト生成

`src/marketing.rs`。テンプレートベースの告知文生成(外部LLM APIは不使用)。
`CAMPAIGNS`定数を書き換えるだけで転用可能。

## 4. サーバー内蔵の定期実行ループ

`main.rs`の`spawn_periodic_tasks()`。`tokio::time::interval`で
起動時に自動でバックグラウンドループを開始する(VPSのcron/systemd timerに
頼らず、アプリ自身が「常に・定期的に・自動で」実行する設計)。環境変数で
間隔を変更可能。

## 5. LINE Messaging API Webhook(署名検証つき)

`src/line_webhook.rs`。`E_GOV_LINE_CHANNEL_SECRET`環境変数が未設定なら
501を返し「未接続」であることを正直に開示する設計。HMAC-SHA256による
署名検証ロジックはLINE公式SDKと同じ仕組みで実装。

## 注意事項

- 秘密情報(チャネルシークレット・アクセストークン等)は`.env`
  (`.gitignore`対象)またはVPSのsystemd `Environment=`経由で渡すこと。
  コードやコミット履歴に直接書き込まないこと。

## 6. お引越し(別サーバーへの移設)

1. 新サーバーにRust(`rustup`)・nginx・Let's Encrypt証明書を用意する。
2. `git clone https://github.com/aon-co-jp/e-gov /root/e-gov.info`
3. `cargo build --release`(1バイナリ完結・DB不要)。
4. systemd `e-gov-server.service`(`WorkingDirectory=/root/e-gov.info`、
   `ExecStart=.../target/release/e-gov-server`、`Restart=on-failure`)を作成し有効化。
5. nginxで`e-gov.info`/`www.e-gov.info`を`127.0.0.1:4500`へリバースプロキシ(HTTPSへ301)。
6. 秘密情報(`E_GOV_LINE_CHANNEL_SECRET`等)は`.env`/`Environment=`で渡す(コミット禁止)。
7. DNSのAレコードを新サーバーへ切替。更新時は`git pull`→`cargo build --release`→`systemctl restart e-gov-server`。

---

## 6. Migrating to another server

1. Install Rust (`rustup`), nginx and a Let's Encrypt certificate on the new host.
2. `git clone https://github.com/aon-co-jp/e-gov /root/e-gov.info`
3. `cargo build --release` (single self-contained binary, no DB).
4. Create and enable a systemd unit `e-gov-server.service` (`WorkingDirectory=/root/e-gov.info`,
   `ExecStart=.../target/release/e-gov-server`, `Restart=on-failure`).
5. Reverse-proxy `e-gov.info`/`www.e-gov.info` to `127.0.0.1:4500` in nginx (301 to HTTPS).
6. Pass secrets (`E_GOV_LINE_CHANNEL_SECRET`, etc.) via `.env`/`Environment=` — never commit them.
7. Switch the DNS A record. To update: `git pull` → `cargo build --release` → `systemctl restart e-gov-server`.

## API 方針: REST 全廃・gRPC / Connect へ (2026-11-04)

### 日本語
- **REST は全廃する。新しい API は REST で作らない。** ブラウザや `curl` から呼ぶ API は **Connect の単純呼び出し**(`POST /<パッケージ>.<Service>/<Method>`、`content-type: application/json`、本文は JSON)、サービス間は **gRPC / gRPC-web**(RPoem の管理API `open_runo.admin.v1.Admin` と同じ流儀)。ストリーミングは、必要になってから。
- **既存の REST は、触るときに順に移す。** 移したら旧 REST の経路は消す。移行が終わるまで残るが、**新規には追加しない**。
- **JSON は RS-JSON に統一**する(読み込みは `from_str_secure` / `parse_secure` でサイズ・深さ・重複キーの上限つき、書き出しは `rust_json::fast`)。ブラウザの WASM に `prost` は入れない(サイズが増えるため)。
- **例外(自分たちの API ではなく、相手の規格が決めているもの)**: ACME、OAuth/OIDC のコールバック、第三者(GitHub など)からの Webhook、ヘルスチェック、静的ファイル配信、OTLP など。これらは規格どおりの HTTP を受ける。
- 参考実装: `RPoem`(`open_runo.admin.v1.Admin`)、`open-easy-web`(`server/src/connect.rs`、`proto/open_easy_web/admin/v1/admin.proto`)。
- ※ このリポジトリのコードが既に方針どおりかどうかは、各リポジトリの現状による。この節は方針の記録であり、コードの変更を意味しない。

### English
- **REST is being abolished. New APIs are not built as REST.** APIs called from browsers or `curl` use **Connect unary calls** (`POST /<package>.<Service>/<Method>`, `content-type: application/json`, JSON body); service-to-service calls use **gRPC / gRPC-web** (the same style as RPoem's admin API `open_runo.admin.v1.Admin`). Streaming only when it is actually needed.
- **Existing REST endpoints are migrated one by one, when they are touched.** The old REST route is removed once migrated. Until migration is finished they remain, but **no new ones are added**.
- **JSON is unified on RS-JSON**: reading with `from_str_secure` / `parse_secure` (size, depth and duplicate-key limits), writing with `rust_json::fast`. `prost` is not put into browser WASM (it would grow the binary).
- **Exceptions (shapes dictated by someone else's standard, not our own API)**: ACME, OAuth/OIDC callbacks, webhooks from third parties (e.g. GitHub), health checks, static file serving, OTLP, and so on. These accept plain HTTP as the standard requires.
- Reference implementations: `RPoem` (`open_runo.admin.v1.Admin`) and `open-easy-web` (`server/src/connect.rs`, `proto/open_easy_web/admin/v1/admin.proto`).
- Note: whether the code in a given repository already follows this policy depends on that repository. This section records the policy; it does not mean the code has been changed.

### Deutsch (KI-Übersetzung, redaktionell überarbeitet; noch nicht von Muttersprachlern geprüft)
- **REST wird vollständig abgeschafft; neue APIs entstehen nicht mehr als REST.** APIs, die aus dem Browser oder per `curl` aufgerufen werden, verwenden **unäre Connect-Aufrufe** (`POST /<Paket>.<Service>/<Methode>`, `content-type: application/json`, JSON-Body). Zwischen Diensten kommen **gRPC bzw. gRPC-web** zum Einsatz, im selben Stil wie die Admin-API von RPoem (`open_runo.admin.v1.Admin`). Streaming wird erst ergänzt, wenn es tatsächlich gebraucht wird.
- **Bestehende REST-Endpunkte werden schrittweise umgestellt, und zwar immer dann, wenn man ohnehin an ihnen arbeitet.** Sobald ein Endpunkt umgestellt ist, wird die alte REST-Route entfernt. Bis zum Abschluss der Migration bleiben die übrigen bestehen, es werden aber **keine neuen mehr hinzugefügt**.
- **JSON wird einheitlich über RS-JSON verarbeitet**: Gelesen wird mit `from_str_secure` bzw. `parse_secure` (mit Obergrenzen für Größe, Verschachtelungstiefe und doppelte Schlüssel), geschrieben mit `rust_json::fast`. `prost` wird nicht in das Browser-WASM aufgenommen, da es die Datei vergrößern würde.
- **Ausnahmen (Schnittstellen, deren Form ein fremder Standard vorgibt und die nicht zu unserer eigenen API gehören)**: ACME, OAuth/OIDC-Callbacks, Webhooks von Drittanbietern (z. B. GitHub), Health-Checks, die Auslieferung statischer Dateien, OTLP und Ähnliches. Diese nehmen, wie vom jeweiligen Standard verlangt, gewöhnliches HTTP entgegen.
- Referenzimplementierungen: `RPoem` (`open_runo.admin.v1.Admin`) und `open-easy-web` (`server/src/connect.rs`, `proto/open_easy_web/admin/v1/admin.proto`).
- Hinweis: Ob der Code eines einzelnen Repositorys diese Richtlinie bereits erfüllt, hängt vom jeweiligen Repository ab. Dieser Abschnitt dokumentiert lediglich die Richtlinie; er besagt nicht, dass der Code schon angepasst wurde.

### Русский (перевод выполнен с помощью ИИ и отредактирован; носителями языка пока не проверен)
- **От REST мы полностью отказываемся; новые API больше не создаются в стиле REST.** API, к которым обращаются из браузера или через `curl`, используют **унарные вызовы Connect** (`POST /<пакет>.<Service>/<Method>`, `content-type: application/json`, тело запроса — JSON). Для взаимодействия между сервисами применяются **gRPC и gRPC-web** — в том же стиле, что и административный API RPoem (`open_runo.admin.v1.Admin`). Потоковая передача данных добавляется лишь тогда, когда в ней действительно возникает необходимость.
- **Существующие REST-эндпоинты переводятся поэтапно — по мере того, как с ними приходится работать.** После перевода старый REST-маршрут удаляется. Пока миграция не завершена, оставшиеся эндпоинты сохраняются, но **новые не добавляются**.
- **Работа с JSON унифицирована на основе RS-JSON**: чтение — через `from_str_secure` или `parse_secure` (с ограничениями на размер, глубину вложенности и повторяющиеся ключи), запись — через `rust_json::fast`. `prost` в браузерный WASM не включается, поскольку это увеличило бы размер файла.
- **Исключения (интерфейсы, форму которых определяет чужой стандарт, а не наш собственный API)**: ACME, обратные вызовы OAuth/OIDC, вебхуки сторонних сервисов (например, GitHub), проверки работоспособности, раздача статических файлов, OTLP и тому подобное. Такие интерфейсы принимают обычный HTTP, как того требует соответствующий стандарт.
- Эталонные реализации: `RPoem` (`open_runo.admin.v1.Admin`) и `open-easy-web` (`server/src/connect.rs`, `proto/open_easy_web/admin/v1/admin.proto`).
- Примечание: соответствует ли код конкретного репозитория этой политике, зависит от самого репозитория. Данный раздел лишь фиксирует политику и не означает, что код уже изменён.
