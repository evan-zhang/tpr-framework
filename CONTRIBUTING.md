# Contributing to TPR Framework

Thank you for your interest in contributing to TPR Framework!

## How to Contribute

### Reporting Issues

- Open an issue at [GitHub Issues](https://github.com/evan-zhang/tpr-framework/issues)
- Include: what you expected, what happened, and how to reproduce it
- For feature requests, describe the use case and why it matters

### Making Changes

1. Fork the repository
2. Create a feature branch (`git checkout -b feat/your-feature`)
3. Make your changes
4. Ensure changes follow the existing document structure and conventions
5. Submit a Pull Request

### Document Conventions

- **SKILL.md** is the entry point. Keep it concise; details go to `references/`
- **references/** contains load-on-demand protocol documents
- **references/templates/** contains project directory templates
- Each reference document has a single responsibility (stated in the opening `>` block)
- Version numbers are tracked in `CHANGELOG.md` (not in individual files)

### Design Philosophy

- **Minimal** — Every file should answer one question (stated at the top)
- **Load on demand** — SKILL.md is always loaded; references/ only when needed
- **Structured output over attitude** — Prefer "write out the consensus and divergences" over "think like an expert"
- **YAGNI** — Don't add configuration layers until the current scale demands it

### Commit Messages

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add Consensus-Divergence Mapping
fix: correct Battle state machine transition
docs: update README installation guide
```

## Questions?

Open an issue or start a discussion on GitHub.
