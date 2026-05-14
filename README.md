# Kaleido Icon Explorer

A single-page, mobile-responsive icon explorer for the Kaleido Design System. Mirrors the patterns of the Kaleido Type Explorer.

## Contents

```
kaleido-icon-explorer/
├── index.html        — the whole app (HTML + CSS + JS, no build step)
├── manifest.json     — generated icon catalog (1,606 entries)
└── icons/
    ├── Cirro/<use-case>/<category>/<name>.svg
    ├── DE/...
    ├── DP/...
    ├── GME/...
    └── Reliant/...
```

## Features

- Search by name (debounced; matches both full name and base name)
- Filter by brand (multi-select with brand-color dots)
- Filter by use-case → category (use-case selection reveals categories)
- Filter by type: Utility vs Illustrative (detected by `-illus` suffix)
- Grid and list views
- Light / Dark theme toggle (respects system preference, persists in localStorage)
- Click an icon to open a modal with full metadata, copy buttons (name, path, SVG source), and download link
- Lazy-loads icons via native `loading="lazy"` and a "Load 200 more" pager when no search is active
- Keyboard: `Esc` closes the modal

## Deploy

1. Drop the entire folder onto static hosting (Netlify, Vercel, GitHub Pages, Figma Sites, S3+CloudFront, etc.).
2. The app fetches `manifest.json` and SVGs by relative path, so no config needed.
3. Embed via `<iframe>` the same way you embed the Type Explorer.

## Regenerating the manifest

If you add/rename/remove icons in the `icons/` folder, regenerate `manifest.json` with the script below (Python 3, no deps):

```python
import os, json, re
ICONS_DIR = "icons"
manifest = {"brands": [], "icons": []}
brand_meta = {
  "Reliant":      {"color": "#0a2540", "label": "Reliant"},
  "DE":           {"color": "#ee5a24", "label": "Direct Energy"},
  "GME":          {"color": "#22c55e", "label": "Green Mountain"},
  "DP":           {"color": "#0f4c81", "label": "Discount Power"},
  "Cirro":        {"color": "#3b82f6", "label": "Cirro"},
  "GME-DP-Cirro": {"color": "#a855f7", "label": "GME / DP / Cirro (Shared)"},
}
# Custom display order (shared last)
order = ["Reliant", "DE", "GME", "DP", "Cirro", "GME-DP-Cirro"]
for brand in sorted(os.listdir(ICONS_DIR)):
  bpath = os.path.join(ICONS_DIR, brand)
  if not os.path.isdir(bpath): continue
  meta = brand_meta.get(brand, {"color": "#888", "label": brand})
  manifest["brands"].append({"key": brand, "label": meta["label"], "color": meta["color"]})
  for use_case in sorted(os.listdir(bpath)):
    ucpath = os.path.join(bpath, use_case)
    if not os.path.isdir(ucpath): continue
    for category in sorted(os.listdir(ucpath)):
      catpath = os.path.join(ucpath, category)
      if not os.path.isdir(catpath): continue
      for f in sorted(os.listdir(catpath)):
        if not f.endswith(".svg"): continue
        name = f[:-4]
        is_illus = name.endswith("-illus") or name.endswith("-illustrative")
        base = re.sub(r'-illus(trative)?$', '', name)
        manifest["icons"].append({
          "brand": brand, "useCase": use_case, "category": category,
          "name": name, "baseName": base, "isIllus": is_illus,
          "path": f"icons/{brand}/{use_case}/{category}/{f}"
        })
with open("manifest.json", "w") as fp:
  json.dump(manifest, fp, separators=(",", ":"))
print(f"Wrote {len(manifest['icons'])} icons")
```

## Brand colors

Edit the `brand_meta` map above (or `state.manifest.brands[*].color` at runtime) to match the official Kaleido brand palette if these placeholders don't match.

Current defaults:
- Reliant `#0a2540`
- DE (Direct Energy) `#ee5a24`
- GME (Green Mountain) `#22c55e`
- DP (Discount Power) `#0f4c81`
- Cirro `#3b82f6`
- GME / DP / Cirro (Shared) `#a855f7`

## Notes on the icon catalog

- 1,645 SVGs across 6 brand entries (5 distinct brands + 1 "Shared" set for GME/DP/Cirro)
- The "GME-DP-Cirro" folder holds icons that are intentionally shared across those three smaller brands. The folder is named with hyphens (`GME-DP-Cirro`) rather than slashes for URL safety; the UI label is "GME / DP / Cirro (Shared)".
- Naming convention: `use-case/category/name` (utility) or `use-case/category/name-illus` (illustrative)
- Use-cases: `general-ui`, `electricity-service`, `promo`
- Categories vary per use-case (e.g. `billing`, `energy`, `home`, `plan`, `campaign`, `support`, etc.)

## Browser support

Modern evergreen browsers. Uses native ES modules, `fetch`, `IntersectionObserver` is not required (lazy load uses `loading="lazy"` only), and `navigator.clipboard.writeText` for copy.
