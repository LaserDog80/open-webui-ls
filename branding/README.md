# Branding Assets

Placeholder assets for LLM Philosopher Chat. Replace before production.

| File          | Purpose                        | Notes                          |
|---------------|--------------------------------|--------------------------------|
| `favicon.svg` | Vector icon (placeholder)      | Dark square + gold φ glyph     |
| `favicon.png` | 64x64 raster icon              | **TODO: add real PNG**         |
| `favicon.ico` | Legacy browser icon            | **TODO: generate from PNG**    |
| `custom.css`  | Theme overrides (colors, hide) | Applied via Admin > Interface  |

## Generating the raster placeholders

```bash
# Requires ImageMagick or rsvg-convert
rsvg-convert -w 64 -h 64 favicon.svg -o favicon.png
convert favicon.png favicon.ico
```

## Applying custom.css

1. Start the container.
2. Admin Panel → Settings → Interface → Custom CSS.
3. Paste contents of `custom.css`, save.

(A future iteration can inject it directly via the Dockerfile into `index.html`.)
