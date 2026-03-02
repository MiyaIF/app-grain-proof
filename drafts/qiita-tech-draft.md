# PWAにStripe+Supabaseでサブスク課金を実装した設計メモ（エクスポート機能のみ有料化）

## この記事で書くこと
- ローカルPWAへの最小課金導入方針
- Netlify FunctionsでのCheckout/Portal/Webhook実装
- Webhook冪等化（event_id一意）

## システム構成
- Frontend: React + Vite + IndexedDB(Dexie)
- Auth: Supabase Anonymous Auth（PII非保持）
- Billing: Stripe Subscription
- API: Netlify Functions

## 実装ポイント
1. 起動時に匿名セッションを自動作成
2. `trialing/active` は利用可、`inactive/past_due` はロック
3. `canceled` でも `current_period_end` まで利用可
4. 復旧コード（ハッシュ保存）で端末移行を可能化
5. Webhook重複イベントを `webhook_events` で無害化

## 参考リンク
- GitHub Release: `https://github.com/MiyaIF/app-grain-proof/releases/tag/v0.1.0-subscription-launch`
- デモURL: 公開準備中

## 著作権
- 著作権: © 2026 Fiso
