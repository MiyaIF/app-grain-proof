# Subscription Launch Evidence (2026-03-02)

## Scope
- Paid feature boundary: Export (Markdown/JSON)
- Free features: Record / Calendar / Backup
- Price: JPY 680 monthly
- Trial: 7 days

## Technical Proof
- private source commit SHA: `bf37631c9dad67b8b06cce22f90418d8f51f5ce2`
- build artifact sha256 (`tar frontend/dist`): `c2a73aae0d0b0f35153c42f52d6c7b521e192c4b32b5e02ad11e8fbc4110dbd5`
- built at (UTC): `2026-03-02T07:38:24Z`
- built at (JST): `2026-03-02 16:38 JST`
- proof command: `./scripts/collect_proof.sh`

## Notes
- App-side PII is not stored; billing/auth data processing is delegated to Stripe/Supabase.
- This repository is for publication-timeline proof, not patent/monopoly claims.
