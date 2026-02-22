---
name: playwright-screenshot-safe
description: Use this skill when taking or validating Playwright MCP screenshots, especially when image-returning screenshot calls cause context churn or compaction. It enforces a safe file-first capture flow and cross-platform path handling.
---

# Playwright Screenshot Safe

Use this when a user asks to capture/inspect browser state and prior attempts with
`browser_take_screenshot` or image-returning responses caused unexpected context
compaction.

## Safe Workflow

1. Choose target URL and output file path first.
2. Capture screenshot with `browser_run_code` and write directly to file.
3. Inspect the saved image from disk (for example with `view_image` in Codex).

## Preferred Capture Method

Avoid image-returning screenshot tool paths during this bug window.

Use this Playwright MCP code pattern:

```js
async (page) => {
  const path = "/tmp/playwright-mcp-output/manual-shot.png";
  await page.screenshot({ path, fullPage: false });
  return { ok: true, path, url: page.url(), title: await page.title() };
}
```

## Path Rules

- Use a path format that matches your runtime environment.
- Linux/macOS example: `/tmp/.../file.png` or `/home/<user>/.../file.png`.
- Windows-native example: `C:/Users/<username>/.../file.png`.
- `view_image` or local file viewers should point to the same saved file, using
  whatever path style that tool expects.
- Replace placeholders like `<username>` with the real user/profile directory name.

## Windows/WSL Interop (Optional)

If the screenshot is written by a Windows Playwright process but inspected from WSL:

- Windows path example: `C:/Users/<username>/.../file.png`
- WSL path example: `/mnt/c/Users/<username>/.../file.png`
- Convert by replacing:
  - `C:/` -> `/mnt/c/`
  - backslashes `\` -> `/`

## Validation Checklist

- Confirm returned `url` and `title` match expected page/project.
- Open the saved file with `view_image` and verify visible UI state.
- Prefer changing one variable at a time (tool method, MCP version, flags, URL).

## Quick Example Sequence

1. `browser_navigate` to target page.
2. Run `browser_run_code` screenshot-to-file snippet.
3. Open the saved file with `view_image` (or equivalent local viewer).
