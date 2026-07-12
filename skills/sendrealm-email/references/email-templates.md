# Email Templates

Use this reference when a task mentions Sendrealm email templates, drafts, publishing, broadcasts, or automations.

Source docs:

- Templates tutorial: `/tutorials/templates`
- List Templates API: `/api-reference/endpoint/list-templates`
- Template versions API: `/api-reference/endpoint/list-template-versions`

## Important Distinction

Sendrealm direct email sends currently use `client.emails.send({ text, html, ... })`. Do not invent a direct-send `template_id` or `template_version_id` option unless the installed SDK or API docs expose one.

Sendrealm email templates are channel-specific reusable assets for dashboard workflows, broadcasts, and automations. They have drafts and published versions. Authoring and lifecycle management happen only in the dashboard; the public API and MCP expose templates as read-only resources.

## Create A Draft In The Dashboard

In `Templates > Emails`, choose **Visual editor** to start from a blank dashboard canvas or **Import HTML** to paste or drop an existing `.html` email. Both flows continue in the dashboard editor. Imported markup opens as a raw HTML block with preview and source views so the original document remains intact. Publish only after content, subject, preview text, and variables have been reviewed.

## List And Retrieve

```ts
const templates = await client.templates.list({
  channel: "email",
  status: "active",
  search: "welcome",
});

const details = await client.templates.retrieve("template_id");
```

## Agent Guardrails

- Ask for required variable names and example data before preparing template content with placeholders.
- Keep variable names stable and case-consistent.
- Hand template creation, updates, publishing, restoring, and archiving to a human in the dashboard.
- If the user asks for a Resend-style template migration, prepare the content and variables for dashboard import instead of calling an unsupported template mutation API.
