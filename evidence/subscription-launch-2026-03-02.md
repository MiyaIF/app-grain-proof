# Subscription Launch Evidence (2026-03-02)

## Scope
- Paid feature boundary: Export (Markdown/JSON)
- Free features: Record / Calendar / Backup
- Price: JPY 500 monthly
- Trial: 7 days

## Technical Proof
- private source commit SHA: `b3ad59ba978ba41e734934b24fdf76c3f80cec7d`
- build artifact sha256 (`tar frontend/dist`): `4b504cdccf3b56ad21a108955ed1a3b31389a36274a7e986f86feb92013828b2`
- built at (UTC): `2026-03-02T13:34:42Z`
- built at (JST): `2026-03-02 22:34 JST`
- proof command: `./scripts/collect_proof.sh`

## Notes
- App-side PII is not stored; billing/auth data processing is delegated to Stripe/Supabase.
- This repository is for publication-timeline proof, not patent/monopoly claims.
