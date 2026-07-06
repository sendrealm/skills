# Email Templates

Use this reference when a task mentions Sendrealm email templates, drafts, publishing, broadcasts, or automations.

Source docs:

- Templates tutorial: `/tutorials/templates`
- Create Template API: `/api-reference/endpoint/create-template`
- Publish Template API: `/api-reference/endpoint/publish-template`
- Template versions API: `/api-reference/endpoint/list-template-versions`

## Important Distinction

Sendrealm direct email sends currently use `client.emails.send({ text, html, ... })`. Do not invent a direct-send `template_id` or `template_version_id` option unless the installed SDK or API docs expose one.

Sendrealm email templates are channel-specific reusable assets for dashboard workflows, broadcasts, and automations. They have drafts and published versions.

## Create A Draft

```ts
import Sendrealm from "@sendrealm/sdk";

const client = new Sendrealm();

const template = await client.templates.create({
  name: "Welcome Series Email",
  description: "First email after signup",
  channel: "email",
  subject_template: "Welcome to {{company_name}}",
  preview_text: "Start with your first delivery in minutes.",
  rendered_html: "<p>Hi {{first_name}}, thanks for joining.</p>",
  rendered_text: "Hi {{first_name}}, thanks for joining.",
  editor_json: {
    type: "doc",
    content: [],
  },
});
```

Use `rendered_html` and `rendered_text` as the rendered caches that Sendrealm stores for the draft. If the app uses a structured editor, preserve its JSON in `editor_json`.

## Publish A Template

```ts
await client.templates.publish(template.id);
```

Publish only after content, subject, preview text, and variables have been reviewed. Automations and broadcasts should use published template versions.

## List And Retrieve

```ts
const templates = await client.templates.list({
  channel: "email",
  status: "active",
  search: "welcome",
});

const details = await client.templates.retrieve(template.id);
```

## Update A Draft

```ts
await client.templates.update(template.id, {
  subject_template: "Welcome back, {{first_name}}",
  rendered_html: "<p>Welcome back.</p>",
  rendered_text: "Welcome back.",
});
```

## Agent Guardrails

- Ask for required variable names and example data before creating templates with placeholders.
- Keep variable names stable and case-consistent.
- Do not overwrite a template draft without first checking whether the task is to update, publish, or create a new template.
- Do not publish automatically unless the user explicitly asks for a publish flow.
- If the user asks for a Resend-style template migration, map the content and variables to Sendrealm template fields instead of copying unsupported send-time options.
