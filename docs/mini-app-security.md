# Mini App security notes

These notes are for people building or reviewing a Telegram Mini App. They are written at a general level, from lessons we learned hardening AntMill’s own app. They are not an audit report, and they are not a recipe for attacking any system.

If you are looking for official platform documentation, start with Telegram’s [Mini Apps guide](https://core.telegram.org/bots/webapps).

## Trust Telegram `initData` on the server, not in the browser

A Mini App can read launch data in the client, but authorization decisions belong on the server.

- Verify the signature with the rules Telegram publishes. Do not invent a shortcut.
- Reject data that is too old, and reject timestamps that sit in the future.
- If verification fails, ask the user to reopen the app from Telegram rather than silently accepting a stale session.

The client can always be modified. Anything important — identity, quotas, payments, admin flags — must be checked again after a valid server-side verification.

## Treat quotas and rate limits as security controls

Free AI calls, login attempts, and other scarce actions will be hammered if they are cheap to retry.

- Count usage in a way that stays correct when several requests arrive at the same time. Check-then-increment patterns lose that race.
- Apply limits per user and per network neighbor, not only per exact IP address. On IPv6, a single subscriber often has a large prefix, so a per-address cap is weaker than it looks.
- Keep a portion of shared capacity for trusted or paying users so a burst of free traffic cannot lock them out.
- Cap request body size before you parse it.

## Do not let the bot become a phishing channel

If a bot can be mentioned in a public group, people will try to make it repeat links and “official” sounding claims.

- Decide, in advance, which chats the bot is allowed to answer in.
- Be conservative about echoing user-supplied URLs in group replies.
- Keep tools and secrets away from the model. If the model cannot take actions or read private credentials, a prompt injection has less to steal — but it can still be used to impersonate you in a chat, so the wording of group replies still matters.

## Referrals and rewards attract abuse

If you grant benefits for inviting friends, expect automated sign-ups.

- Credit a referral only after simple, observable activity, not at the moment the link is opened.
- Ignore self-invites.
- Bound how many grants one account can receive in a short window, and review clusters that arrive together from the same network neighborhood.

Describe rewards honestly. Do not promise financial returns for inviting people.

## Payments, pins, and secrets

- Keep different secrets for different jobs. A webhook secret is not a good stand-in for a PIN pepper or a session key.
- Store user PINs or passcodes with a slow hash and a unique salt. Lock out guessing, and count attempts reliably.
- Make payment grants and refunds idempotent, so a retry cannot apply the same charge twice or leave access behind after a refund.

## Test the failure modes

Write tests that describe the behavior you want under load and under invalid input: stale launch data, oversized bodies, repeated attempts, and concurrent quota requests. A mocked backend is enough for this. You do not need to publish attack scripts to prove that a control holds.

## What this document is not

It is not a penetration-test checklist, and it is not permission to probe apps you do not operate. If you find a problem in AntMill, see [SECURITY.md](../SECURITY.md).
