---
name: quickstart
description: Walk the user through the Matamba Gateway sandbox quickstart (list accounts, estimate, place a market buy of 10 DANGCEM, check it fills, view the position) with code in their own programming language, reading the sandbox API key from an environment variable. Use only when the user asks to run or try the Matamba Gateway sandbox quickstart.
disable-model-invocation: true
---

# Matamba Gateway sandbox quickstart

Take the user from their sandbox API key to a filled order in five requests, exactly as the docs' quickstart does. `references/quickstart.md` is that page in full: follow it, and don't add steps it doesn't have.

Write the code in the programming language the user named when they asked for this. If they named none, use the main language of the project in the working directory; with no project, use `curl` in a shell script. Talk to the user in the language they write in.

## Rules

1. **Sandbox only.** Every request goes to this host and no other:

   <!-- BEGIN GENERATED: sandbox-host -->
   Sandbox: `https://gateway-sandbox.matambaintelligence.com`
   <!-- END GENERATED: sandbox-host -->

2. **The key stays out of the conversation and out of files.** The code reads it at runtime from the environment variable `MATAMBA_API_KEY`. Never ask for the key in chat. Never print, echo or log it, and never write it to any file: not the script, not a `.env`, not a config file. If the user pastes a key into the chat anyway, don't use or repeat it: tell them it is now in the transcript, that they should revoke it (`POST /v1/keys/revoke` sent with that key, or in the partner portal) and create a new one.
3. **Refuse anything but a sandbox key.** Before the first request, the code checks that `MATAMBA_API_KEY` is set and starts with `mgw_test_`, without displaying it, and stops with a clear message if not. The Postman collection refuses the same way.
4. If the variable isn't set in your environment, don't work around it. Tell the user to set it in their own terminal from wherever they keep secrets, for example `read -rs MATAMBA_API_KEY && export MATAMBA_API_KEY` (it isn't echoed or saved in shell history), then either run the script there themselves or restart this session from that terminal.
5. **Money is integer kobo.** Send `quantity` as a JSON integer. Show kobo figures as naira by decimal division (`47500` kobo is ₦475.00); parse `_kobo_decimal` strings as decimals, never floats.
6. **One `Idempotency-Key` per run.** The code generates a UUID for the order at the start of the run and sends it on `POST /v1/orders` and on every retry of that request. After a timeout or a `5xx`, retry with the same key, never a new one. The estimate and the reads take no key.
7. Print each step's HTTP status, the `x-request-id` header and the fields that matter. Never print request headers.

## Steps

Write one new file, such as `matamba_quickstart.py`, in the working directory, unless the user wants it elsewhere. Show it to the user, then run it only if the key is already in your environment and the user agrees; otherwise give them the command to run it.

1. **List your accounts.** `GET /v1/accounts`. The sandbox gives two account codes: one to trade with, and one ending in `99` with a zero balance. Trade with the first account whose code doesn't end in `99`. A `401` means the key is wrong or isn't a sandbox key.
2. **Estimate the cost.** `POST /v1/orders/estimate` with `{"account": <account>, "symbol": "DANGCEM", "side": "buy", "quantity": 10, "type": "market", "time_in_force": "day"}`. Show `estimated_total_kobo` and `estimated_total_kobo_decimal`. A market buy is estimated at the day's upper price limit, plus charges, so the fill costs less.
3. **Place the order.** `POST /v1/orders` with the same body and the run's `Idempotency-Key`. Expect `201` with `state` `pending`. Keep its `id`.
4. **Check the order status.** Wait 30 seconds, then `GET /v1/orders/{id}`. If it still reads `pending`, retrieve it again a few seconds later, up to about two minutes. Expect `filled`, with `filled_quantity` `10` and `average_fill_price_kobo_decimal`.
5. **View the position.** `GET /v1/accounts/{account}/positions`. Show the `DANGCEM` position's `quantity`, `average_cost_kobo_decimal`, `market_price_kobo` and `market_value_kobo`.

If the order comes back `uncertain`, don't place it again: retrieve it until it resolves. For any error, read `error.code`, never `message`, and look it up in the error reference.

## Then

Offer the user the next step, with its docs page:

- See a rejected order: place the same order on the account ending in `99` with a new key. It returns `422 order_rejected` with `reject_reason` `insufficient_funds`.
- Rehearse other errors and rare outcomes: <https://docs.matambaintelligence.com/sandbox>.
- Open an account for a customer: <https://docs.matambaintelligence.com/applications>.
- Fund an account from the wallet: <https://docs.matambaintelligence.com/funding>.
- Receive events: <https://docs.matambaintelligence.com/webhooks>.
