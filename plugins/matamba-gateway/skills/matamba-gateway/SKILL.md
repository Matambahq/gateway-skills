---
name: matamba-gateway
description: Integration rules for the Matamba Gateway API, which opens brokerage accounts for a partner's customers in Nigeria, funds them from the partner's wallet, and trades Nigerian equities on the Nigerian Exchange (NGX). Use when writing, reviewing or debugging code that calls the gateway (the host gateway-sandbox.matambaintelligence.com or a live Matamba Gateway host, API keys starting mgw_test_ or mgw_live_), or when the user asks to open or fund a customer's brokerage account, place, estimate or cancel an equity order, read positions, trades or contract notes, or handle Matamba webhooks, errors, idempotency keys, kobo amounts or uncertain orders.
---

# Matamba Gateway integration rules

<!-- BEGIN GENERATED: release -->
Docs release: **1 October 2026** (the newest heading in the [changelog](https://docs.matambaintelligence.com/changelog)).
<!-- END GENERATED: release -->

These rules come from the partner docs at <https://docs.matambaintelligence.com>. Follow them over anything you remember about other brokerage or payments APIs. Each rule links to its source, and `references/` holds those pages in full.

When something isn't covered here:

1. Read the matching file in `references/` (listed at the end).
2. If the `matamba-gateway-docs` MCP server is connected, search it with `search_matamba_gateway`, or read a page with `query_docs_filesystem_matamba_gateway`.
3. Otherwise fetch <https://docs.matambaintelligence.com/llms.txt> and the page it names.

Never invent an endpoint, field, error code or behaviour. If the docs don't say, tell the user so, and point them to partners@matambaintelligence.com ([support](https://docs.matambaintelligence.com/support#how-to-reach-us)).

## 1. Hosts and environments

<!-- BEGIN GENERATED: sandbox-host -->
Sandbox: `https://gateway-sandbox.matambaintelligence.com`
<!-- END GENERATED: sandbox-host -->

- The live host comes with the partner's live keys ([quickstart](https://docs.matambaintelligence.com/quickstart#before-you-begin)). Never guess it or hard-code it: read the base URL from configuration, next to the key.
- Sandbox keys start `mgw_test_` and live keys start `mgw_live_`. Each works only on its own host; a key sent to the other host gets `401 unauthorized` ([keys](https://docs.matambaintelligence.com/keys-and-security#api-keys)).
- The sandbox has the live API's endpoints, test money, and no exchange ([sandbox](https://docs.matambaintelligence.com/sandbox)). HTTPS only, TLS 1.2 or later ([connections](https://docs.matambaintelligence.com/keys-and-security#connections)).
- Every path is under `/v1` ([versioning](https://docs.matambaintelligence.com/versioning)). Bodies are JSON with `Content-Type: application/json`, except document uploads (section 7).
- Set the HTTP client timeout above 115 seconds. `POST /v1/orders` can take up to 95 seconds to answer ([limits](https://docs.matambaintelligence.com/limits#how-long-a-request-can-take)).

## 2. API keys: never in code, never in logs

- Send `Authorization: Bearer <key>` on every request ([how it works](https://docs.matambaintelligence.com/how-it-works)).
- Read the key at runtime from an environment variable or a secrets manager. Never write a key into source code, committed config, tests, fixtures, examples or logs, and never put one in an app or web page that customers download ([keys](https://docs.matambaintelligence.com/keys-and-security#how-keys-are-created)). Never ask the user to paste a key into the conversation.
- Redact the `Authorization` header wherever requests are logged.
- Give each service a key with only the scopes it needs: `read`, `trade`, `fund`, `apply`. A missing scope gets `403 insufficient_scope`, naming it ([scopes](https://docs.matambaintelligence.com/keys-and-security#scopes)).
- A live key works only from the IP addresses registered for it. Any other address gets the same `401` as a bad key ([live keys](https://docs.matambaintelligence.com/keys-and-security#live-keys-and-ip-addresses)).
- If a key leaks, revoke it at once: `POST /v1/keys/revoke`, sent with that key to its own host (`204`). Then create a new key in the partner portal. Don't rotate a leaked key ([if a key leaks](https://docs.matambaintelligence.com/support#if-a-key-leaks)).
- Log the `x-request-id` response header with your record of each request. Support asks for it ([handle errors](https://docs.matambaintelligence.com/handle-errors#1-log-the-request-id)).
- Never log a customer's name, date of birth, BVN, NIN, phone, email, address, next of kin or bank details, and never put personal data in `client_reference` ([customers' data](https://docs.matambaintelligence.com/keys-and-security#your-customers-data), [your own reference](https://docs.matambaintelligence.com/orders#your-own-reference)).

## 3. Idempotency-Key

<!-- BEGIN GENERATED: idempotency-required -->
| Request | Key |
|---|---|
| `POST /v1/orders` | Required |
| `POST /v1/applications` | Required |
| `POST /v1/allocations` | Required |
| Every other request | Not used |
<!-- END GENERATED: idempotency-required -->

- Generate one key per order, application or allocation, such as a UUID. Save it with your own record before you send the request, and send the same key on every retry of that request ([choose a key](https://docs.matambaintelligence.com/idempotency#choose-a-key)).
- A key is 1 to 255 printable ASCII characters with no spaces. Send the header exactly once.
- The same key with the same request returns `200` and the resource as it is now. The same key with a different request returns `409 duplicate_order`, `duplicate_application` or `duplicate_allocation`. `client_reference` is part of the request ([what a retry returns](https://docs.matambaintelligence.com/idempotency#what-a-retry-returns)).
- After a timeout, no response, `500`, `503 service_unavailable` or `503 exchange_unavailable`: send the same request with the same key. After `429 rate_limited` or `503 trading_paused`: wait the `Retry-After` seconds, then the same request with the same key. Never a new key: for an order, a new key is a second order at the exchange ([when to use a new key](https://docs.matambaintelligence.com/idempotency#when-to-use-a-new-key)).
- Use a new key only when the answer says the request was stored as finished and you still want it: `503 order_not_sent`, `application_not_sent` or `allocation_not_sent` (send it again), or `422 order_rejected`, `application_rejected` or `allocation_rejected` (act on its `reject_reason` first: some, such as `rejected_by_exchange`, mean contact support rather than resend).
- `503 outcome_not_recorded`: don't send it again at all. Retrieve the order, application or allocation the error names until it resolves.
- `409 duplicate_*`: the key belongs to another request. Read the resource the error names before sending anything with a new key: it may be the one you intended.
- When nothing was stored (`400`, `401`, `403 insufficient_scope`, `413`, `415`, `422 unknown_symbol`, `422 limit_exceeded`, `422 wallet_too_low`, `409 similar_*_unresolved`), fix the cause and reuse the same key.
- `references/idempotency.md` has every code and what a retry is compared on.

## 4. Money: integer kobo, never floats

- Every amount is an integer in kobo: ₦1 is 100 kobo, so ₦475.00 is `47500` ([how it works](https://docs.matambaintelligence.com/how-it-works)). Send amounts, prices and quantities as JSON integers, never as strings or decimals: `"quantity": "10"` gets `400 invalid_request` ([sandbox errors](https://docs.matambaintelligence.com/sandbox#test-your-error-handling)).
- Convert naira to kobo exactly: parse the naira amount as a decimal (for example Python `Decimal`, Java `BigDecimal`, or integer arithmetic on the digits), multiply by 100, and require a whole number. Never multiply a binary float.
- A field ending `_kobo_decimal` is an exact decimal string of kobo, such as `"530337.5"`. Parse it with a decimal type, never a float, and round only for display.
- The whole-kobo figures are rounded against the customer; the `_kobo_decimal` figures are exact:
  - `estimated_total_kobo`: up for a buy (the most it can cost), down for a sell ([estimate](https://docs.matambaintelligence.com/trade#2-estimate-the-order)).
  - Balance: `opening_kobo` and `available_kobo` down, `pending_debits_kobo` up ([balance](https://docs.matambaintelligence.com/api-reference/accounts/retrieve-an-accounts-balance)).
  - Contract notes: the three charges up; `amount_kobo` and `consideration_kobo` up on a buy and down on a sell. Each is rounded on its own, so for an exact total add the `_kobo_decimal` figures ([after a fill](https://docs.matambaintelligence.com/orders#after-a-fill)).
- `average_fill_price_kobo_decimal` is exact to 12 decimal places ([fills](https://docs.matambaintelligence.com/orders#fills)).
- The API gives no prices or quotes. Never show an estimate as a price ([prices](https://docs.matambaintelligence.com/trade#prices-to-show-your-customers)).

## 5. Orders

- `POST /v1/orders` takes `account`, `symbol`, `side` (`buy` or `sell`), `quantity` (whole shares, 1 to 100,000,000), `type`, `time_in_force`, `price_kobo` for a limit order, and an optional `client_reference`. `references/endpoints.md` has every field's rule.
- `type: "limit"`: send `price_kobo`, the limit price per share in kobo. Shares and ETFs take limit orders.
- `type: "market"`: leave out `price_kobo`. Shares only: an ETF takes limit orders only, and a market order on one is rejected with `reject_reason` `etf_limit_orders_only`. No endpoint lists ETFs; in the sandbox the only one is `STANBICETF30` ([order types](https://docs.matambaintelligence.com/orders#order-types)).
- `time_in_force`: `day`, `gtc`, `ioc` or `fok`. Send a market order as `day`, never `gtc`: only a limit order can be cancelled ([time in force](https://docs.matambaintelligence.com/orders#time-in-force)).
- Estimate first with `POST /v1/orders/estimate`: the same body, no `Idempotency-Key`, nothing stored. A market order is priced at the day's price limit, so before a market buy check that `available_kobo` from `GET /v1/accounts/{account}/balance` covers the estimate ([estimate](https://docs.matambaintelligence.com/trade#2-estimate-the-order)).
- States: `sending`, `pending`, `partially_filled`, `uncertain` are not final; `filled`, `rejected`, `refused`, `cancelled`, `expired` are final. Treat a state you don't recognise as not final ([order states](https://docs.matambaintelligence.com/orders#order-states)).
- Read `filled_quantity`, not the number of events. An order that ends `cancelled`, `expired` or `rejected` can have `filled_quantity` above zero ([fills](https://docs.matambaintelligence.com/orders#fills)).
- Cancel with `DELETE /v1/orders/{id}`: only a limit order that is `pending` or `partially_filled`. `202` means the broker accepted the cancel, not that the order is cancelled: wait for `order.cancelled`, or `order.filled` if it filled first. On `503 cancel_unconfirmed` or `503 exchange_unavailable`, send the cancel again; repeating a cancel is safe. On `409 order_not_cancelable`, retrieve the order ([cancel](https://docs.matambaintelligence.com/trade#5-cancel-an-order)).
- Sell no more than the position's `quantity` from `GET /v1/accounts/{account}/positions` ([sell only what the account holds](https://docs.matambaintelligence.com/trade#sell-only-what-the-account-holds)).
- The exchange opens for order entry at 9:00 WAT, trades continuously from 9:30 until the pre-close at 15:50, and closes at 16:00; it doesn't trade on Nigerian public holidays. No endpoint says whether the market is open. `reject_reason` `market_closed` means send it again with a new key when it opens ([trading hours](https://docs.matambaintelligence.com/orders#trading-hours)).
- No endpoint lists symbols ([find the symbol](https://docs.matambaintelligence.com/trade#1-find-the-symbol)). Charges are on contract notes, not on the order ([after a fill](https://docs.matambaintelligence.com/orders#after-a-fill)).

## 6. Uncertain, sending and unplaced

- `uncertain` means the order was sent and its receipt at the exchange is unconfirmed. It may be at the exchange. Never place it again under a new key: wait for its next event, or retrieve it with `GET /v1/orders/{id}`. An order still `sending` after its request ended is the same ([what to do](https://docs.matambaintelligence.com/handling-uncertain#what-to-do)).
- While you wait, a retry with the same key is safe: it returns `200` and the order as it is now. An uncertain order can't be cancelled.
- `409 similar_order_unresolved` or `409 similar_allocation_unresolved`: an identical one is unresolved and nothing was stored. Retrieve the one the error names until it resolves, then follow [identical orders](https://docs.matambaintelligence.com/handling-uncertain#identical-orders-while-one-is-unresolved) or [identical allocations](https://docs.matambaintelligence.com/handling-uncertain#identical-allocations-while-one-is-unresolved).
- An uncertain application or allocation also resolves without you. An allocation's amount stays reserved until it becomes `booked`, or `rejected` with `not_booked`. An application that ends `unplaced` is final: contact support before sending it again ([applications and allocations](https://docs.matambaintelligence.com/handling-uncertain#applications-and-allocations)).
- The operations team reconciles 9:00 to 17:00 WAT, Monday to Friday, the same or the next business day. `references/uncertain.md` has the full rules.

## 7. Open an account

The flow: `POST /v1/applications` → `documents_required` → send the four documents → `submitted` → the broker's staff review it → `opened`, with the account code in `account` ([open an account](https://docs.matambaintelligence.com/applications)).

1. An account is for one person who is Nigerian and lives in Nigeria. Checking the customer's identity (KYC) is the partner's job, before applying ([who can have an account](https://docs.matambaintelligence.com/applications#who-can-have-an-account)).
2. Look up the codes. `state` and `state_of_origin` come from `GET /v1/reference/states`; `lga_of_origin` from `GET /v1/reference/states/{state}/lgas` for the state of origin; `bank` from `GET /v1/reference/banks`. They take codes, not free text ([look up the codes](https://docs.matambaintelligence.com/applications#1-look-up-the-codes)).
3. Send `POST /v1/applications` with a new `Idempotency-Key` ([send the application](https://docs.matambaintelligence.com/applications#2-send-the-application)).
   <!-- BEGIN GENERATED: application-fields -->
   - Required fields: `first_name`, `last_name`, `gender`, `date_of_birth`, `email`, `phone`, `mothers_maiden_name`, `street`, `city`, `state`, `postcode`, `state_of_origin`, `lga_of_origin`, `next_of_kin_name`, `next_of_kin_phone`, `bank`, `bank_account_name`, `bank_account_number`, `bank_account_opened`, `bvn`.
   - Optional fields: `middle_name`, `next_of_kin_relationship`, `next_of_kin_address`, `nin`, `tax_id`, `client_reference`.
   <!-- END GENERATED: application-fields -->
   - `phone` and `next_of_kin_phone`: a Nigerian number, `0` and 10 digits (`08012345678`), or `+234` and the same 10 digits. Any other form gets `400 invalid_request`, so normalise the number before sending.
   - `mothers_maiden_name` is required: without it, `400 missing_required_field`.
   - `gender` is `M` or `F`; dates are `yyyy-MM-dd`; `bvn` and `nin` are 11 digits; each field is at most 200 characters.
4. Read the answer: `201` `documents_required` (the broker has it); `201` `uncertain` (wait, don't send it again under a new key); `422 application_rejected` (read `reject_reason` and `param`: for `invalid_field`, correct that field and send with a new key); `503 application_not_sent` (send again with a new key); `503 outcome_not_recorded` (retrieve it) ([read the answer](https://docs.matambaintelligence.com/applications#3-read-the-answer)).
5. Send the four documents, one request each: `PUT /v1/applications/{id}/documents/{type}` with `type` `photo`, `government_id`, `utility_bill` and `signature` ([send the documents](https://docs.matambaintelligence.com/applications#send-the-documents)).
   - The body is the file's raw bytes: not JSON, not multipart, not base64. Set `Content-Type` to `image/jpeg` or `image/png`, matching the file.
   - Each file must be a whole JPEG or PNG, smaller than 1,000,000 bytes and at most 8,000 pixels wide and high. Check the size before sending. A PDF gets `415 unsupported_media_type`: convert it to an image first.
   - No `Idempotency-Key`. Send the same file again to retry or to check: it isn't sent twice.
   - Keep each file, byte for byte, until it's `delivered`. A retry must send the same bytes: a file compressed again is a different file.
   - At most two documents on their way at once. A third gets `429 rate_limited`.
   - The answer is `200` with `state` `delivered`. The fourth makes the application `submitted`. `references/applications.md` lists every other answer.
6. Wait for the `application.opened` event, which carries `account`, or retrieve `GET /v1/applications/{id}`. The broker's review has no promised time and can take days ([wait for the account](https://docs.matambaintelligence.com/applications#4-wait-for-the-account-to-open)).

- `reject_reason` `already_at_broker`: don't send it again; contact support ([a customer the broker already knows](https://docs.matambaintelligence.com/applications#a-customer-the-broker-already-knows)).
- In the sandbox, use a new `bvn` and `email` for each application. `bvn` `00000000000` makes the broker reject it at once ([in the sandbox](https://docs.matambaintelligence.com/applications#in-the-sandbox)).

## 8. Fund an account

- The partner pays into its wallet by bank transfer, with its payer reference in the narration. There is no API call for it, and no event: the credit appears in `GET /v1/wallet/transfers` as type `credit` ([pay into your wallet](https://docs.matambaintelligence.com/funding#1-pay-into-your-wallet)).
- Move money to one account with `POST /v1/allocations`, body `{"account": ..., "amount_kobo": ...}` and a new `Idempotency-Key`. `201` with `state` `booked` means the money is in the account's balance, and the customer can trade ([allocate](https://docs.matambaintelligence.com/funding#3-allocate-to-the-account)).
- `422 wallet_too_low`: nothing stored; pay in more and send it again with the same key. `422 allocation_rejected`: the amount is back in the wallet; act on `reject_reason`. `503 allocation_not_sent`: send again with a new key. `uncertain`, or `sending` after the request ended: wait; don't send it again under a new key.
- `available_kobo` on `GET /v1/wallet` is `credited_kobo` minus `allocated_kobo`, `reserved_kobo`, `withdrawing_kobo` and `withdrawn_kobo` ([the figures](https://docs.matambaintelligence.com/wallet#the-figures)).
- There is no API for withdrawals ([take money out](https://docs.matambaintelligence.com/wallet#take-money-out)).

## 9. Webhooks and events

- Every state change after `sending` of an order, application or allocation creates an event. It is sent to the partner's HTTPS endpoint, which Matamba registers on request (no API), and listed by `GET /v1/events` for 30 days ([webhooks](https://docs.matambaintelligence.com/webhooks)).
- Verify every event before acting on it ([verify each event](https://docs.matambaintelligence.com/webhooks#verify-each-event)). It follows Standard Webhooks:
  1. Use the raw body bytes exactly as they arrived. Never re-serialise parsed JSON.
  2. Reject a `webhook-timestamp` more than 5 minutes from your clock.
  3. Signed content: `{webhook-id}.{webhook-timestamp}.{raw body}`.
  4. Key: base64-decode the secret after its `whsec_` prefix. HMAC-SHA256, then base64.
  5. Accept if any space-separated `v1,<signature>` entry in `webhook-signature` matches, using a constant-time compare.
- Test verification against the test vector in `references/webhooks.md`. Keep the signing secret like an API key, one per environment.
- Answer `2xx` within 10 seconds. Save the event, answer, and only then do the work, such as updating your order records, from a queue or a background job. Never run business logic before answering: a slow task must not become a failed delivery ([respond](https://docs.matambaintelligence.com/webhooks#respond)). If saving the event fails, answer `5xx` and the event is sent again ([retries](https://docs.matambaintelligence.com/webhooks#retries)).
- Delivery is at least once and can be out of order: ignore a repeated `webhook-id`, and per object ignore an event whose `version` is lower than the last one applied ([retries](https://docs.matambaintelligence.com/webhooks#retries)). Ignore event types you don't handle.
- To catch up, or instead of webhooks, read `GET /v1/events` from a stored `next_cursor`; poll every 30 seconds ([catch up](https://docs.matambaintelligence.com/webhooks#catch-up-on-missed-events)).

## 10. Errors

- Every error is `{"error": {"type", "code", "message", "doc_url", ...}}`, with `param`, `reject_reason`, and `order`, `application` or `allocation` when they apply. Branch on `error.code`, never on `message` ([errors](https://docs.matambaintelligence.com/errors)).
- When a response has `Retry-After`, wait that many seconds.
- For a `5xx` without `Retry-After`, back off: 1 second, doubling with random jitter, at most 60 seconds apart. After about 5 minutes, stop and establish the outcome: retrieve the object, or list by `client_reference` ([back off](https://docs.matambaintelligence.com/handle-errors#4-back-off-between-retries)).
- Default for a code you don't recognise ([defaults](https://docs.matambaintelligence.com/handle-errors#5-keep-a-default-for-unrecognised-codes)): `409`, don't send it with a new key and retrieve the object it names; `429`, wait `Retry-After` and send the same request with the same key; any other `4xx`, don't send it again as it is; `5xx`, the same request with the same key, backing off.
- `references/errors.md` has every code, its status and type, what to do, and whether your key was stored.

## 11. Limits

<!-- BEGIN GENERATED: limits -->
| Limit | Default | What it counts |
|---|---|---|
| Requests per second | 20, with bursts up to 40 | Every request made with any of your keys, counted together. |
| Requests in progress at once | 5 | `POST /v1/orders`, `POST /v1/orders/estimate`, `DELETE /v1/orders/{id}`, `POST /v1/applications`, `PUT /v1/applications/{id}/documents/{type}` and `POST /v1/allocations`, across all your keys. |
| Order value per day | ₦10,000,000 | The value of the orders you place in one day, West Africa Time (WAT). |
| Allocations per day | ₦50,000,000 | The amount you allocate in one day, WAT. |

| Limit | Response | What to do |
|---|---|---|
| Requests per second | `429 rate_limited`, with `Retry-After: 1` | Wait 1 second, then send the same request again. For a request that takes an `Idempotency-Key`, keep the same key. |
| Requests in progress at once | `429 rate_limited`, with `Retry-After: 1` | Wait 1 second, then send the same request again. For a request that takes an `Idempotency-Key`, keep the same key. |
| Order value per day | `422 limit_exceeded` | Nothing was stored, so the key is still free. The limit resets at midnight, WAT. A single order larger than the whole daily limit never fits: split it, or [ask us](https://docs.matambaintelligence.com/support) to change the limit. |
| Allocations per day | `422 limit_exceeded` | Nothing was stored, so the key is still free. The limit resets at midnight, WAT. A single allocation larger than the whole daily limit never fits: split it, or [ask us](https://docs.matambaintelligence.com/support) to change the limit. |
<!-- END GENERATED: limits -->

- `GET /v1/limits` shows each limit, what's used and what's left ([see what's left](https://docs.matambaintelligence.com/limits#see-whats-left)).
- Balances, positions, trades, contract notes and reference lists are read from the broker, 2 at a time per partner. Keep your own copy of balances and positions, refresh it when an event changes the account, and read the broker when you need the latest figure, such as before a sell ([reads that ask the broker](https://docs.matambaintelligence.com/limits#reads-that-ask-the-broker)).

## 12. Lists and versioning

- A list is `{"data", "has_more", "next_cursor"}`. Pass `limit` (1 to 100, default 25) and the previous `next_cursor` as `cursor`; stop when `has_more` is false; keep the same filters on every page ([how it works](https://docs.matambaintelligence.com/how-it-works)).
- `v1` grows without notice: ignore fields and event types you don't know, and give unknown states, reject reasons and codes a safe default. A client generated from the OpenAPI file should accept unknown enum values ([versioning](https://docs.matambaintelligence.com/versioning#changes-we-make-within-a-version)).

## 13. What the API doesn't do

There is no endpoint to list symbols or ETFs, give prices, say whether the market is open, create API keys, register a webhook endpoint, withdraw money, or change or close a customer's account. Don't invent one. Keys are made in the partner portal, and the rest go through support ([support](https://docs.matambaintelligence.com/support#what-to-send-for-each-request), [what the API doesn't do yet](https://docs.matambaintelligence.com/applications#what-the-api-doesnt-do-yet)).

## Reference files

Read the one you need; each is generated from the docs.

| File | Holds |
|---|---|
| `references/endpoints.md` | Every endpoint with its scope and `Idempotency-Key`, request fields and rules, money fields, and value lists (from `openapi.json`) |
| `references/errors.md` | Every error code, reject reason, what to do and whether the key was stored; troubleshooting |
| `references/idempotency.md` | Which requests take a key, what a retry is compared on, when to use a new key |
| `references/uncertain.md` | Uncertain and unresolved orders, applications and allocations |
| `references/trading.md` | Placing, estimating and cancelling orders; states, hours, fills, contract notes |
| `references/applications.md` | Opening an account, document uploads, application states |
| `references/funding.md` | Wallet, payments in, allocations, ledger, withdrawals |
| `references/webhooks.md` | Event format and types, verification code in Node.js, Python and Go, test vector, retries |
| `references/limits.md` | Rate and daily limits, fixed limits, timeouts |
| `references/sandbox.md` | Sandbox behaviour, requests for each error, test values |
| `references/keys-and-security.md` | Keys, scopes, rotation, IP addresses, TLS, stored data |
| `references/concepts.md` | How it works, versioning, glossary |
| `references/support.md` | Contacting Matamba, what to send, leaked keys and secrets |
| `references/changelog.md` | What changed in the newest docs release |
