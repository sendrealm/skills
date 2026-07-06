---
name: sendrealm-email
description: Use when building, reviewing, or debugging Sendrealm outbound email integrations, including trusted-backend use of @sendrealm/sdk, transactional email sends, attachments, tags, custom headers, domain and API-key setup, email templates for broadcasts or automations, typed SDK errors, and safe testing. Always use this skill when the user asks an agent to add email sending with Sendrealm or migrate a Resend-like email workflow to Sendrealm.
---

# Sendrealm Email

Use this skill to implement Sendrealm email from trusted server-side code. Keep it guidance-first: write or review integration code, validate payload shapes, and ask the human for missing credentials or sending-domain choices instead of inventing them.

## Quick Send

Use `@sendrealm/sdk` from backend, worker, queue, or serverless code.

```ts
import Sendrealm from "@sendrealm/sdk";

const client = new Sendrealm({
  apiKey: process.env.SENDREALM_API_KEY,
});

const email = await client.emails.send({
  from: "hello@example.com",
  to: ["user@example.com"],
  subject: "Welcome",
  text: "Welcome to Sendrealm.",
  html: "<p>Welcome to Sendrealm.</p>",
});
```

Before production sends, confirm `SENDREALM_API_KEY` is stored server-side and the `from` domain is verified in Sendrealm.

## Non-Negotiables

- Never put a Sendrealm API key in browser JavaScript, mobile apps, React Web Push code, React Native, Android, or iOS SDK setup.
- Prefer the official `@sendrealm/sdk` for Node/TypeScript projects; use REST only when the project cannot install the SDK.
- Handle errors with `try/catch` and typed SDK errors such as `APIError` and `RateLimitError`. Sendrealm does not use Resend's Node pattern of returning `{ data, error }`.
- Do not add Resend-only behavior unless Sendrealm exposes it. In particular, do not claim outbound email sends support Resend-style email idempotency keys.
- Ask for missing `from`, recipients, subject, content, API key storage location, and verified domain details.

## What To Load

- Read [email-sending.md](references/email-sending.md) for SDK setup, direct sends, attachments, tags, headers, typed errors, rate limits, and safe testing.
- Read [email-templates.md](references/email-templates.md) when the task mentions email templates, drafts, publishing, broadcasts, or automations.

## Implementation Workflow

1. Identify where trusted backend code lives: API route, server action, worker, queue consumer, cron job, or edge runtime.
2. Install or reuse `@sendrealm/sdk`; configure `SENDREALM_API_KEY` through the project's existing secret pattern.
3. Build the email payload with API-required `from`, `to`, `subject`, and `text`. Add `html` when sending rich email.
4. Add optional `cc`, `bcc`, `reply_to`, `attachments`, `headers`, and `tags` only when the user or existing code needs them.
5. Wrap sends in error handling that logs status, code, and message without leaking secrets.
6. Use safe test recipients or internal addresses first, then verify delivery and domain settings before broad rollout.

## Common Mistakes

| Mistake | Fix |
| --- | --- |
| Calling Sendrealm from a client component or browser bundle | Move the call to a server route or trusted backend function. |
| Copying Resend SDK error handling | Use `try/catch`; Sendrealm throws typed errors. |
| Assuming templates can be used in direct email sends | Direct sends take `text`/`html`; Sendrealm templates are reusable assets for dashboard, broadcasts, and automations. |
| Sending from an unverified domain | Verify the sending domain and DNS records before production. |
| Logging API keys or full payloads with sensitive content | Log message IDs, status, and sanitized errors. |
