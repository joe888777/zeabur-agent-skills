---
name: zeabur-template-publish
description: Use when publishing or updating a Zeabur template to the marketplace. Use when user says "publish template", "update template online", "push template to Zeabur", or "create template on Zeabur". Do NOT use for deploying templates (use zeabur-template-deploy instead). Do NOT use for editing template YAML (use zeabur-template instead).
---

# Zeabur Template Publish

> **Always use `npx zeabur@latest` to invoke Zeabur CLI.** Never use `zeabur` directly or any other installation method. If `npx` is not available, install Node.js first.

Publish a template YAML file to the Zeabur marketplace, or update an existing one.

## Create a New Template

```bash
npx zeabur@latest template create -f <template-file>.yaml
```

This uploads the YAML and returns a template code (e.g., `VTZ4FX`). Save this code — you'll need it for future updates.

## Update an Existing Template

```bash
npx zeabur@latest template update -c <TEMPLATE_CODE> -f <template-file>.yaml
```

The template code is the identifier from the template URL: `https://zeabur.com/templates/<TEMPLATE_CODE>`

## Delete a Template

```bash
npx zeabur@latest template delete -c <TEMPLATE_CODE>
```

## Verify

After publishing or updating, check the template page:

```
https://zeabur.com/templates/<TEMPLATE_CODE>
```

Or fetch the YAML to confirm:

```bash
npx zeabur@latest template get -c <TEMPLATE_CODE> --raw
```

## Flags

### `template create`

| Flag | Description |
|------|-------------|
| `-f, --file` | Template YAML file to publish |

### `template update`

| Flag | Description |
|------|-------------|
| `-c, --code` | Template code to update |
| `-f, --file` | Template YAML file with changes |

### `template delete`

| Flag | Description |
|------|-------------|
| `-c, --code` | Template code to delete |

## Validator Gotchas

The `template create` / `template update` server-side validator is stricter than the inline YAML schema in `zeabur-template`. Common rejections:

| Error from `template create` | Cause | Fix |
|------------------------------|-------|-----|
| `metadata.version: unknown field` | `metadata.version` is **not** in the schema | Remove the field; version your YAML in git instead |
| `variables[i].secret: unknown field` | `secret: true` is not a valid variable property | Use `type: PASSWORD` |
| `variables[i].type: required` | Variable missing `type` | Add `type: STRING` (or `PASSWORD` / `DOMAIN` / `AI_HUB_KEY`) |
| `variables[i].name: required` | Variable shape incomplete | Each variable needs `{key, type, name, description}` |
| `services[i].spec.source.image: invalid reference format` | `image: ${VAR}` placeholder fails docker-ref regex | Inline the literal image ref, or envsubst the file before publishing |
| `services[i].spec.source.command: not allowed at this level` | `command` placed at `spec` instead of `spec.source` | Move `command` inside `source`, alongside `image` |

If `template create` returns a validation error, **do not retry the same YAML**. The validator is deterministic — fix the field shape and re-publish. See the `zeabur-template` skill for the verified variable schema reference.

## Workflow

```bash
# First time: create and note the returned code (use the `zeabur-template` skill to create the YAML first)
npx zeabur@latest template create -f zeabur-template-myapp.yaml
# → Template created: VTZ4FX

# Later: update with changes
npx zeabur@latest template update -c VTZ4FX -f zeabur-template-myapp.yaml

# Verify
npx zeabur@latest template get -c VTZ4FX --raw
```

