# Stripe本番環境有効化 Runbook (2026-03-02)

## Summary
`app-grain` のStripe課金設定を test値から live値へ切り替えるための実行手順です。  
本Runbookは実行順固定で、各手順のタイトルと説明をそのまま運用に利用できます。

## Public APIs / Interfaces / Types
- 変更なし
- 既存Functions（`create-checkout-session` / `create-portal-session` / `stripe-webhook`）の実装変更なし
- 変更対象は環境変数とStripe Dashboard設定のみ

## 事前固定値
- 料金: 月額500円（税込）
- トライアル: 7日
- 有料機能: エクスポート（Markdown/JSON）
- 無料機能: 記録 / カレンダー / バックアップ
- 本番サイトURL: `https://<site>.netlify.app`

## 手順タイトルと説明（実行順固定）
### 1. 本番切替条件の凍結
説明: 料金を「月額500円・7日トライアル」に固定し、本番切替中に価格・試用日数・対象機能（有料: エクスポート、無料: 記録/カレンダー/バックアップ）を変更しない運用を先に確定する。  
実施チェック:
- [ ] 固定値をチーム内で合意した
- [ ] 切替完了まで価格/試用日数変更を行わないと決定した

### 2. Stripe LiveモードでProduct/Priceを作成
説明: Stripe DashboardをLiveモードに切り替え、月額500円・7日トライアルのProduct/Priceを新規作成する。作成した `price_...` を本番用 `STRIPE_PRICE_ID_MONTHLY` に採用する。  
実施チェック:
- [ ] Live modeでProduct/Priceを作成した
- [ ] 本番用 `price_...` を記録した

### 3. 本番用APIキーの取得と管理台帳への記録
説明: Liveの `sk_live_...` を取得し、testキーと混在しないよう識別ラベル（`prod` / `preview`）付きで管理する。キー値はGit管理に含めず、Netlify Environment Variablesのみで扱う。  
実施チェック:
- [ ] `sk_live_...` を取得した
- [ ] `prod` と `preview` の識別を管理台帳へ記録した
- [ ] Git管理対象に秘密値が含まれていない

### 4. Netlify Production環境変数へlive値を投入
説明: Production contextに `STRIPE_SECRET_KEY=sk_live_...`、`STRIPE_PRICE_ID_MONTHLY=price_...`、`APP_BASE_URL=https://<site>.netlify.app` を設定する。既存のSupabase本番値との組み合わせを維持する。  
実施チェック:
- [ ] Productionの `STRIPE_SECRET_KEY` を live値へ更新した
- [ ] Productionの `STRIPE_PRICE_ID_MONTHLY` を live値へ更新した
- [ ] Productionの `APP_BASE_URL` を本番URLへ設定した

### 5. Preview/Branch環境のtest値維持を明示
説明: Deploy Preview / Branch Deployは `sk_test_...` とtest用 `price_...` を維持し、Productionへ誤流入しない境界を固定する。必要なら説明コメントをNetlify側に追加する。  
実施チェック:
- [ ] Preview/Branchが `sk_test_...` のままである
- [ ] Preview/Branchがtest用 `price_...` のままである
- [ ] 役割境界を運用コメントで明示した

### 6. Stripe Live Webhook endpointを作成
説明: LiveモードのWebhook endpointを `https://<site>.netlify.app/.netlify/functions/stripe-webhook` で作成する。購読イベントは `customer.subscription.created` / `customer.subscription.updated` / `customer.subscription.deleted` を設定する。  
実施チェック:
- [ ] Live endpoint URLを設定した
- [ ] 購読イベント3種を設定した

### 7. WebhookシークレットをProductionへ反映
説明: Live endpointで払い出された `whsec_...` を `STRIPE_WEBHOOK_SECRET` としてProduction contextへ登録する。`stripe listen` 由来のローカル `whsec_...` を使わないことを確認する。  
実施チェック:
- [ ] Productionの `STRIPE_WEBHOOK_SECRET` をlive endpoint値へ更新した
- [ ] ローカル用 `whsec_...` を使用していない

### 8. 本番デプロイ実行（環境変数反映）
説明: NetlifyでProduction deployを実行し、Functionsが最新環境変数を読む状態にする。必要なら再デプロイを行い、旧設定キャッシュを排除する。  
実施チェック:
- [ ] Production deployを実行した
- [ ] 最新デプロイで環境変数反映を確認した

### 9. 本番ヘルスチェック（500防止）
説明: 認証済みで `/.netlify/functions/subscription-status` と `/.netlify/functions/create-checkout-session` を実行し、HTTP 500が出ないことを確認する。失敗時は `env:check`、`supabase:probe`、`price_...` 形式を優先確認する。  
実施チェック:
- [ ] `subscription-status` が500を返さない
- [ ] `create-checkout-session` が500を返さない
- [ ] 異常時の優先切り分け手順を実施した

### 10. 本番課金スモークテスト
説明: 実際にCheckoutを開始し、契約直後にエクスポートが有効化されることを確認する。続いてStripe Portal遷移が成功することを確認する。  
実施チェック:
- [ ] Checkout開始から購読状態反映まで成功した
- [ ] 購読後にエクスポート機能が有効化された
- [ ] Stripe Portal遷移に成功した

### 11. 解約/期限内利用の回帰確認
説明: 解約後も `current_period_end` まで利用できる仕様が維持されることを確認し、期限経過後にロックへ遷移する挙動を検証する。  
実施チェック:
- [ ] 解約後の期限内利用が可能だった
- [ ] 期限経過後にロックへ遷移した

### 12. 公開証跡とリリース本文の同期
説明: 本番URL、料金、課金仕様境界をRelease/Qiita/Zenn草稿へ反映し、`commit_sha` と `build_tar_sha256` の証跡と矛盾がないことを最終確認する。  
実施チェック:
- [ ] Release本文へ本番URLを追記した
- [ ] Qiita/Zenn草稿へ本番URLを追記した
- [ ] 料金/機能境界/証跡値の不一致がない

## Test Cases and Scenarios
1. Productionで `STRIPE_SECRET_KEY` が `sk_live_...`、Previewで `sk_test_...` になっている。  
2. Productionで `STRIPE_PRICE_ID_MONTHLY` がLive作成の `price_...` に一致する。  
3. Live webhookから `customer.subscription.*` が到達し、`stripe-webhook` が署名検証を通過する。  
4. 購入後に有料機能（エクスポート）が即時利用可能になる。  
5. Stripe Portal遷移が成功し、解約後の期限内利用が継続する。  
6. `subscription-status` / `create-checkout-session` が認証時に500を返さない。  
7. Release本文・証跡ファイル・料金表記に不一致がない（500円/7日で統一）。

## Assumptions / Defaults
1. 本番サイトURLは `https://<site>.netlify.app` を採用する。  
2. 課金プランは単一（月額500円、7日無料トライアル）で運用する。  
3. Webhook購読イベントは `customer.subscription.created` / `customer.subscription.updated` / `customer.subscription.deleted` の3種を標準とする。  
4. 既存Supabaseスキーマ（`subscriptions` / `webhook_events`）は適用済みとする。  
5. 実装コードの改修は行わず、設定変更と運用手順のみで本番化する。  
