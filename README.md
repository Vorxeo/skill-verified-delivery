# Verified Delivery

Claude Code skill: `verified-delivery`

## What

Distinguishes **Written** (file on disk) from **Verified** (command ran; output and exit code read from a result file). Stops agents from claiming green without evidence.

## When to use

End of every coding chunk before claiming done; before Faber handoffs; before bench or gate claims.

## Install

Copy this folder into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/verified-delivery
cp SKILL.md ~/.claude/skills/verified-delivery/
```

Claude Code loads `SKILL.md` from `~/.claude/skills/<name>/`.

## Sibling skills

`karpathy-method`, `handoff-faber-rigor`, `bench-loop`, `adversarial-qa`, `fail-closed-review`, `migration-and-data-safety`, `code-that-holds`

Proposed repos: see [Vorxeo](https://github.com/Vorxeo) `skill-*` packs.

## License

MIT — Copyright (c) 2026 Vorxeo. See [LICENSE](./LICENSE).
