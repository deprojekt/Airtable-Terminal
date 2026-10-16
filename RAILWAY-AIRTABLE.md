# Airtable Trade Book + Railway

## Verified Airtable schema (read-only inspection, 2026-10-16)

Base `appEJAdiQAm0Ooniq`; model trade table `tblq56obY4a8zXSsn` (`MODEL TRADE BOOK`). Fields read from existing base: Trade ID, Ticker, Mode, Status, Entry Date, Entry Price, Position Size IDR, Current Price, Exit Plan Tomorrow, Exit Date, Exit Price, Realized P&L %, Realized P&L IDR, Unrealized P&L %, Unrealized P&L IDR, Entry Thesis, Exit Reason, Last Evaluated. No schema modification or trade mutation was performed.

## Deployment

1. Push this project to a **private** GitHub repository (do not commit secrets).
2. In Railway, create **two services** from that repo. For the API use `server/Dockerfile`, for the web use `web/Dockerfile`. Configure Dockerfile paths manually in the service build settings; the included `railway-api.json` and `railway-web.json` are reference configs.
3. Configure environment variables per `.env.railway.example`. Use the same long random `API_KEY` in both services. API has **private networking only**. Web gets the public HTTPS domain.
4. Set Airtable token scope `data.records:read` and `data.records:write` for this base only. Keep it on API service exclusively, never in `NEXT_PUBLIC_` variables.
5. Enable the web access user/password and `REQUIRE_TERMINAL_AUTH=1`. Basic authentication requires HTTPS; for multiuser systems replace it with proper identity-aware authentication.
6. Attach Railway persistent volume `/app/data` to API to preserve existing SQLite portfolios. Airtable trades are loaded live and not mirrored into SQLite.
7. Open `https://<your-web-domain>/trades` to see the Airtable trade editor.

## Safety / boundaries

- The **ACTUAL** Mode is server-enforced **read-only**. Unrecognized/blank modes are also read-only. Only `MODEL` / `PAPER` allow modifications. `Mode`, `Ticker` and `Trade ID` cannot be modified through the API.
- Only a narrow allowlist of trade fields can be patched; no Airtable record deletions or creation, no order execution, and no edits to SIGNAL LEDGER, SIGNAL PERFORMANCE or DAILY MARKS.
- The UI loads fresh Airtable records on refresh. Writes PATCH existing Airtable records directly; a version/fingerprint detects stale edits under ordinary use, but Airtable lacks a conditional update primitive here, so concurrent writes can still race between read and PATCH. Do not treat this as transactional synchronization.
- P&L % is fractional in Airtable (0.1 = 10%). Exit P&L is only computed with supplied entry/exit prices; realized IDR is only populated with positive Position Size IDR. No inferred fills.
- Existing local Portfolio widget operates independently on SQLite. The new Airtable trade editor is accessible via the Terminal header. Transactions in local SQLite are **not** bidirectionally synchronized to Airtable, to prevent duplicates and mixing model plans with brokerage transactions.
- Never expose Railway API publicly without additional controls. Consider backup, access logs, strong identity, and read-after-write verification before using with real capital.
