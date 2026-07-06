# Sendrealm Skills

Portable agent skills for adding Sendrealm email and push notification integrations.

## Install

```bash
npx skills add sendrealm/skills
```

## Included Skills

- `sendrealm-email`: server-side email sends, templates, domains, API keys, attachments, tags, headers, and testing patterns.
- `sendrealm-push`: push setup for Web, React Native, iOS, and Android, including device registration, backend sends, provider configuration, and delivery diagnostics.

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
```

Version 1 is guidance-only. These skills help coding agents write and review integration code; they do not include account-management tools or commands that send production messages.

## Maintenance

Keep the examples aligned with the Sendrealm docs and API reference pages linked from each skill. Skill frontmatter should stay portable: only `name` and `description`.
