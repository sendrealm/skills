# Email Sending

Use this reference for direct outbound email sends from trusted backend code.

Source docs:

- JavaScript SDK: `/sdks/javascript`
- Send Email API: `/api-reference/endpoint/create`
- API Keys: `/tutorials/api-keys`
- Domains: `/tutorials/domains`

## Contents

- Install And Configure
- Send Parameters
- Attachments
- Next.js Route Example
- Error Handling
- Domain And Safety Checks

## Install And Configure

```bash
npm install @sendrealm/sdk
```

Store the API key in the project's existing secret manager or environment configuration:

```bash
SENDREALM_API_KEY=sk_...
```

Create the client in server-only code:

```ts
import Sendrealm from "@sendrealm/sdk";

const client = new Sendrealm();
```

The client reads `SENDREALM_API_KEY` automatically. Pass `apiKey` only when the project already centralizes secrets that way.

## Send Parameters

```ts
await client.emails.send({
  from: "hello@example.com",
  to: ["user@example.com"],
  cc: ["cc@example.com"],
  bcc: ["audit@example.com"],
  reply_to: ["support@example.com"],
  subject: "Welcome",
  text: "Welcome to Sendrealm.",
  html: "<p>Welcome to Sendrealm.</p>",
  tags: [
    { name: "workflow", value: "welcome" },
    { name: "tenant", value: "acme" },
  ],
  headers: {
    "X-Customer-ID": "cus_123",
  },
});
```

The Send Email API requires `from`, `to`, `subject`, and `text`. Add `html` for rich email; keep `text` as the plain-text fallback.

## Attachments

Attachments use base64 data.

```ts
await client.emails.send({
  from: "billing@example.com",
  to: ["user@example.com"],
  subject: "Invoice",
  text: "Your invoice is attached.",
  attachments: [
    {
      name: "invoice.pdf",
      type: "application/pdf",
      data: base64Pdf,
    },
  ],
});
```

Do not read large files into memory casually. Follow the host application's existing file-size limits, streaming/download policies, and privacy rules.

## Next.js Route Example

```ts
import { NextResponse } from "next/server";
import Sendrealm, { APIError, RateLimitError } from "@sendrealm/sdk";

const sendrealm = new Sendrealm({
  apiKey: process.env.SENDREALM_API_KEY,
  maxRetries: 2,
});

export async function POST(request: Request) {
  const { email, firstName } = await request.json();

  try {
    const result = await sendrealm.emails.send({
      from: "hello@example.com",
      to: [email],
      subject: "Welcome",
      text: `Hi ${firstName}, welcome.`,
      html: `<p>Hi ${firstName}, welcome.</p>`,
      tags: [{ name: "workflow", value: "welcome" }],
    });

    return NextResponse.json({ id: result.id });
  } catch (error) {
    if (error instanceof RateLimitError) {
      return NextResponse.json(
        { error: "Rate limited" },
        { status: 429 }
      );
    }

    if (error instanceof APIError) {
      console.error("Sendrealm email failed", {
        status: error.status,
        code: error.code,
        message: error.message,
      });
      return NextResponse.json({ error: "Email failed" }, { status: 502 });
    }

    throw error;
  }
}
```

Validate untrusted request input before sending. Never let arbitrary client input become `from`, headers, attachment data, or unsanitized HTML.

## Error Handling

Sendrealm throws typed errors. Use `try/catch`.

```ts
import { APIError, RateLimitError } from "@sendrealm/sdk";

try {
  await client.emails.send(payload);
} catch (error) {
  if (error instanceof RateLimitError) {
    const retryAfter = error.headers.get("retry-after");
    // Retry later according to project queue policy.
  }

  if (error instanceof APIError) {
    console.error(error.status, error.code, error.message);
  }
}
```

Common categories:

| Status | Action |
| --- | --- |
| 400 or 422 | Fix request validation, including required `from`, `to`, `subject`, and `text`; do not blindly retry. |
| 401 | Check `SENDREALM_API_KEY`. |
| 403 | Check project access, key scope, and verified sending domain. |
| 429 | Retry later using `retry-after` when present. |
| 500+ | Retry with bounded exponential backoff through the app's queue policy. |

## Domain And Safety Checks

- Verify the sending domain and DNS records before production.
- Keep SPF, DKIM, and DMARC aligned with the sending domain.
- Respect unsubscribe/compliance requirements for non-transactional email.
- Avoid fake real-provider test addresses such as random Gmail addresses.
- Log message IDs and sanitized errors, not API keys or sensitive email bodies.
- If the same business event can be retried, deduplicate in your own application unless Sendrealm exposes an email idempotency feature for that path.
