# HomeLight Sales-Team Knowledge Base

A curated markdown corpus for HomeLight sales context — designed for humans and bots (Knowledge Bot, Cursor agents) to read and contribute to.

## Purpose

This repository holds **expansive sales context** for HomeLight programs, processes, and partner relationships. It is **not** a personal vault dump or a place for meeting notes. Content here should be relevant to the sales team's day-to-day work.

## What's Here

| Folder | Contents |
|--------|----------|
| `product/` | Program details: BBYS, DTI Drop, HELOC, other Homes programs |
| `sales/` | Sales process, playbooks, objection handling, talk tracks |
| `partners/` | Partner context useful to sales (lenders, agents, integrations) |
| `ops/` | Operations context that intersects with sales workflows |
| `glossary/` | Term definitions, acronyms, and quick-reference material |

See [SCOPE.md](SCOPE.md) for detailed inclusion/exclusion criteria.

## Citing Paths

When referencing content from this repo:

- Use relative paths from the repo root: `product/bbys/README.md`
- In conversations or docs, cite as: `homelight-knowledgebase/product/bbys/...`
- Bots should include the full path when quoting or summarizing

## How Bots Should Write

**Read freely. Write only when explicitly asked.**

- Bots (Knowledge Bot, Cursor agents, etc.) may read any file at any time
- Bots should **not** create or edit files unless a human explicitly requests it
- When asked to write, bots should:
  - Place content in the appropriate folder per SCOPE.md
  - Use clear, factual language — no invented policy or speculation
  - Preserve existing structure and formatting conventions
  - Note the source if importing from another system

## Contributing (Humans)

1. Add markdown files to the appropriate folder
2. Keep content factual and sourced where possible
3. Avoid PII, secrets, and sensitive internal data
4. Update folder READMEs when adding new categories of content
