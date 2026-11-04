# e-gov.info

行政(お役所・政府)のデジタル化と、個人〜貿易商社まで対応するオンライン貿易・
不動産プラットフォームを、LINEアプリ・WEBサイト・コンビニ端末という複数の
入り口から利用できる形で統合するプロジェクトです。

> ⚠️ **THIS IS STILL A SAMPLE / DEMONSTRATION ONLY — NOT A LIVE SERVICE**
> ⚠️ **これはまだサンプル・デモンストレーションです(実際のサービスでは
> ありません)**。WEBサイト・LINE友だちどちらから利用しても、このお断りが
> 毎回表示されます。電子公証・電子契約(法的拘束力のある売買・賃貸契約の
> 締結)は正式な許可が下りるまで現段階では一切実装していません。

## 2本柱

### 1. eガバメント(デジタルガバメント)

- **入り口の多様化**: LINEアプリ・WEBサイトに加えて、WEBサイトやスマホの
  操作が分からない方向けに、コンビニのマルチメディア端末(Loppi/Famiポート/
  セブン-イレブンのマルチコピー機等)を申請窓口として使えるようにします。
- **段階的本人確認**: 手続きの金額・重要度に応じて認証の強度を変えます
  (ナンバーディスプレイ+折り返し電話 → メールOTP → マイナンバーカードの
  スマホNFCスキャン)。
- ペーパーレス・オンライン申請、複数省庁・自治体をまたぐワンストップ化を
  目指します。

### 2. オンライン貿易プラットフォーム(AIチャットコマース)

- 食料品から自動車・オーディオ機器まで、幅広い商材をLINE・WEBの両方から
  AIとの対話で発見・注文できる総合通販です。
- 個人〜貿易商社まで対応するマーケットプレイス型。既存の大手プラット
  フォームより手数料を大幅に引き下げます。
- **AI与信調査・掛け仕入れ・売掛保証**: 与信スコアに応じた後払い仕入れ、
  電子請求書の重複調査、売掛債権の保証機能。
- **不動産投資・AI工務店**: 土地情報からAIが間取りを提案。投機的資金の
  過剰流入を助長しない設計方針。
- 初期フェーズは実際の在庫仕入れを行わず、アドセンスによるサンプル運用
  から開始します。

## 参考にした海外事例

- エストニア e-Estonia / X-Road(分散型データ連携基盤)
- アゼルバイジャン ASAN xidmet(ワンストップ窓口・モバイル巡回サービス)

## 自動調査・自動マーケティング機能

デジタルガバメント関連のテーマについて、日本語・英語で定期的に自動調査
(GitHub Search APIでの実データ取得、Google検索リンクの自動生成)を行い、
各機能の告知ドラフトも定期的に自動生成します(`/research`ページ参照)。

## LINE連携

WEBサイトの各ページを見るだけでなく、LINE公式アカウントを友だち追加して
チャットで問い合わせることもできます(ルールベースのAIチャットコマース、
「申請したい」「買いたい」「仕入れたい」「土地を探したい」等のキーワードに
反応)。友だち追加用のQRコード掲載は、LINE公式アカウントのBasic ID発行後に
対応予定です。

## GitHubからの自動表示

TOPページは、このリポジトリの`README.md`・`CLAUDE.md`・`PORTING.md`を
GitHubから自動取得し、GitHub風にレンダリングして表示します
(`.rs`形式への変換表示も切替可能)。

## TOPページのお知らせリンク(2026-10-14)

ナビ直下に「政府は、LINEアプリ化して、99％リストラして少子高齢化と年金と後期高齢者無償化予算確保。」
(https://ameblo.jp/www-aon/entry-12981307682.html)へのリンクを表示(`src/main.rs`、全言語共通)。VPS反映済み。

## お引越し(別サーバーへの移設)

1. 新サーバーにRust(`rustup`)・nginx・Let's Encrypt証明書を用意する。
2. `git clone https://github.com/aon-co-jp/e-gov /root/e-gov.info`
3. `cargo build --release`(1バイナリ完結・DB不要)。
4. systemd `e-gov-server.service`(`WorkingDirectory=/root/e-gov.info`、
   `ExecStart=.../target/release/e-gov-server`、`Restart=on-failure`)を作成し有効化。
5. nginxで`e-gov.info`/`www.e-gov.info`を`127.0.0.1:4500`へリバースプロキシ(HTTPSへ301)。
6. 秘密情報(`E_GOV_LINE_CHANNEL_SECRET`等)は`.env`/`Environment=`で渡す(コミット禁止)。
7. DNSのAレコードを新サーバーへ切替。更新時は`git pull`→`cargo build --release`→`systemctl restart e-gov-server`。

## 技術スタック

Rust + [Poem](https://github.com/poem-web/poem)。DB非依存・1バイナリ完結。
`aruaru-tokyo`/`karu.tokyo`と同じ技術スタックです。

詳細な設計思想は [CLAUDE.md](CLAUDE.md) を参照してください。

## 関連プロジェクト

- [open-raid-z](https://github.com/aon-co-jp/open-raid-z) — 開発ルールの正本
- [aruaru-db](https://github.com/aon-co-jp/aruaru-db) — 将来のDB接続候補
- [aruaru-tokyo](https://github.com/aon-co-jp/aruaru-tokyo-server) / [karu.tokyo](https://github.com/aon-co-jp/karu.tokyo) — 同じ技術スタックの姉妹サイト

---

作者 / Author: 石 塚 正 浩（MASAHIRO ISHIZUKA） (https://github.com/aon-co-jp)

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
