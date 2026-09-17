---
name: before-and-after
description: Captures before/after screenshots of web pages or elements for visual comparison. Use when user says "take before and after", "screenshot comparison", "visual diff", "PR screenshots", "compare old and new", or needs to document UI changes. Accepts two URLs (file://, http://, https://) or two image paths.
allowed-tools:
  - Bash(npx @vercel/before-and-after *)
  - Bash(before-and-after *)
  - Bash(which before-and-after)
  - Bash(npm install -g @vercel/before-and-after)
  - Bash(*/upload-and-copy.sh *)
  - Bash(curl -s -o /dev/null -w *)
  - Bash(gh pr view *)
  - Bash(gh pr edit *)
  - Bash(vercel inspect *)
  - Bash(vercel whoami)
  - Bash(which vercel)
  - Bash(which gh)
---

# Before-After Screenshot Skill

> **Package:** `@vercel/before-and-after`
> Never use `before-and-after` (wrong package).

## Agent Behavior Rules

**DO NOT:**
- Switch git branches, stash changes, start dev servers, or assume what "before" is
- Use `--full` unless user explicitly asks for full page / full scroll capture

**DO:**
- Use `--markdown` when user wants PR integration or markdown output
- Use `--mobile` / `--tablet` if user mentions phone, mobile, tablet, responsive, etc.
- Assume current state is **After**
- If user provides only one URL or says "PR screenshots" without URLs, **ASK**: "What URL should I use for the 'before' state? (production URL, preview deployment, or another local port)"

## Execution Order (MUST follow)

1. **Pre-flight** — `which before-and-after || npm install -g @vercel/before-and-after`
2. **Protection check** — if `.vercel.app` URL: `curl -s -o /dev/null -w "%{http_code}" "<url>"` (401/403 = protected)
3. **Capture** — `before-and-after "<before-url>" "<after-url>"`
4. **Upload** — try `./scripts/upload-and-copy.sh <before.png> <after.png> --markdown` (default host is 0x0.st).
5. **PR integration** — put GitHub-hosted image URLs in the PR body. If step 4 fails (0x0.st often refuses uploads), attach with `gh` instead. Do not leave `./foo.png` or `.artifacts/...` paths in the PR. GitHub cannot fetch your laptop.

**Never skip steps 1-2.**

## Quick Reference

```bash
# Basic usage
before-and-after <before-url> <after-url>

# With selector
before-and-after url1 url2 ".hero-section"

# Different selectors for each
before-and-after url1 url2 ".old-card" ".new-card"

# Viewports
before-and-after url1 url2 --mobile    # 375x812
before-and-after url1 url2 --tablet    # 768x1024
before-and-after url1 url2 --full      # full scroll

# From existing images
before-and-after before.png after.png --markdown

# Via npx (use full package name!)
npx @vercel/before-and-after url1 url2
```

| Flag | Description |
|------|-------------|
| `-m, --mobile` | Mobile viewport (375x812) |
| `-t, --tablet` | Tablet viewport (768x1024) |
| `--size <WxH>` | Custom viewport |
| `-f, --full` | Full scrollable page |
| `-s, --selector` | CSS selector to capture |
| `-o, --output` | Output directory (default: ~/Downloads) |
| `--markdown` | Upload images & output markdown table |
| `--upload-url <url>` | Custom upload endpoint (default: 0x0.st) |

## Image Upload

0x0.st is the CLI default and is often down or blocking uploads. A local path in the PR body will not render.

If `--markdown` / 0x0.st fails, attach files to GitHub with `gh` **2.99+** (`gh pr edit --help` must list `--attach`). Older `gh` (for example 2.67) has no flag. Download a newer binary if needed.

```bash
# Capture only. Keep the PNGs.
npx @vercel/before-and-after "<before-url>" "<after-url>" -o .artifacts/<task>

# After the PR exists, upload to GitHub and rewrite markdown that uses the same paths:
gh pr edit <n> --body-file body.md \
  --attach './before.png#Before' \
  --attach './after.png#After'
```

`body.md` must reference the same paths you pass to `--attach` (`![Before](./before.png)`), or `gh` appends the images at the end instead of filling the table.

`gh gist create` rejects PNG binaries. Do not use the gist adapter for screenshots.

```bash
# Default (0x0.st). Fine when it works. Not required.
./scripts/upload-and-copy.sh before.png after.png --markdown
```

## Vercel Deployment Protection

If `.vercel.app` URL returns 401/403:

1. Check Vercel CLI: `which vercel && vercel whoami`
2. If available: `vercel inspect <url>` to get bypass token
3. If not: Tell user to provide bypass token, take manual screenshots, or disable protection

## PR Integration

```bash
# Check for gh CLI
which gh

# Get current PR
gh pr view --json number,body

# Host images on GitHub so they render in the description
gh pr edit <number> --body-file body.md \
  --attach './before.png#Before' \
  --attach './after.png#After'
```

If no `gh` CLI: output markdown and tell the user to paste and drag the PNGs into the GitHub UI.

## Error Reference

| Error | Fix |
|-------|-----|
| `command not found` | `npm install -g @vercel/before-and-after` |
| `could not determine executable` | Use `npx @vercel/before-and-after` (full name) |
| 401/403 on .vercel.app | See Vercel protection section |
| Element not found | Verify selector exists on page |
| 0x0.st upload refused / empty image in PR | Use `gh pr edit --attach` (gh 2.99+). Never paste local disk paths. |
