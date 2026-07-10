# Sendrealm Skills

Portable agent skills for adding Sendrealm email and push notification integrations, plus safe dashboard workflows through MCP.

## Install

```bash
npx skills add sendrealm/skills
```

## Included Skills

- `sendrealm-email`: server-side email sends, templates, domains, API keys, attachments, tags, headers, and testing patterns.
- `sendrealm-push`: push setup for Web, React Native, iOS, and Android, including device registration, backend sends, provider configuration, and delivery diagnostics.
- `sendrealm-dashboard`: API-key-scoped project, domain, audience, contact lookup/upsert, template, correlated automation, sandbox test-run, event, and campaign-draft workflows through the Sendrealm MCP server.

## Package Layout

```text
skills/
  sendrealm-email/
    SKILL.md
    agents/openai.yaml
    references/
  sendrealm-push/
    SKILL.md
    agents/openai.yaml
    references/
  sendrealm-dashboard/
    SKILL.md
    agents/openai.yaml
```

The dashboard skill describes MCP workflows for account resources. Campaign scheduling and production sends remain dashboard-only.

## Maintenance

Keep the examples aligned with the Sendrealm docs and API reference pages linked from each skill. Skill frontmatter should stay portable: only `name` and `description`.
