# Hugo Academic Site: Copilot Instructions

## Project Architecture

This is a **dual-layer Hugo project** combining a custom theme with an academic website:

- **Root level** (`/`): "Black & Light" theme source files - a high-performance, JavaScript-free Hugo theme
- **Spock directory** (`/Spock/`): The active Hugo site using the theme, containing all content and configuration
- **Build target**: GitHub Actions deploys from `Spock/public/` to GitHub Pages at `anupanand.space`

## Critical Workflow

### Development & Testing
```bash
cd Spock
hugo server -D  # Preview with drafts at http://localhost:1313
hugo --minify   # Build production site to Spock/public/
```

### File Organization Rules
1. **Content files** → `Spock/content/` (research.md, publications.md, etc.)
2. **Site-specific layouts** → `Spock/layouts/` (override theme defaults)
3. **Theme base layouts** → `layouts/` and `layouts/partials/` (root level)
4. **Configuration** → `Spock/config.toml` only
5. **Static files** → `Spock/static/files/` for PDFs, images, etc.

### Layout Override Hierarchy
The site uses Hugo's template hierarchy where `Spock/layouts/` overrides theme layouts in root `/layouts/`:
- Custom header: `Spock/layouts/partials/header.html` (adds KaTeX, dropdown menus, theme toggle)
- Theme header: `layouts/partials/header.html` (simpler, base version)
- Custom single page: `Spock/layouts/_default/single.html` (adds subtitle support)

**Rule**: To modify site behavior, edit files in `Spock/layouts/` - never directly modify root theme files.

## Project-Specific Patterns

### Math Rendering
The site uses **KaTeX** for LaTeX math. Enable per-page or globally:
```markdown
+++
title = "Research"
math = true  # Enable KaTeX for this page
+++

Inline math: $E = mc^2$
Display math: $$\int_{a}^{b} f(x) dx$$
```

Global enable in [Spock/config.toml](Spock/config.toml): `[params] math = true`

### Menu System
[Spock/config.toml](Spock/config.toml) defines a nested menu with dropdown support:
```toml
[[menu.main]]
    name = "More"
    identifier = "more"  # Parent identifier
    weight = 40

[[menu.main]]
    name = "Publications"
    url = "/publications/"
    parent = "more"      # Makes this a dropdown child
    weight = 41
```

Dropdowns are rendered by custom logic in [Spock/layouts/partials/header.html](Spock/layouts/partials/header.html#L65-L77) using `.HasChildren` checks.

### Theme Toggle Implementation
The site has a custom dark/light mode toggle ([Spock/layouts/partials/header.html](Spock/layouts/partials/header.html#L81-L93)) that:
- Persists preference to `localStorage` 
- Sets `data-theme` attribute on `<html>` element
- CSS in [Spock/layouts/partials/styles.html](Spock/layouts/partials/styles.html#L3-L28) uses `[data-theme='dark']` and `[data-theme='light']` selectors
- Falls back to system `prefers-color-scheme` if no preference set

### Subtitle Support
Pages can have optional subtitles with HTML content:
```markdown
+++
title = "Research"
subtitle = 'Find my publications <a href="/publications/">here</a>'
+++
```

Rendered in [Spock/layouts/_default/single.html](Spock/layouts/_default/single.html#L5-L7) using `safeHTML` to allow inline links.

## Deployment & Build

**GitHub Actions workflow** ([.github/workflows/hugo.yml](.github/workflows/hugo.yml)):
- Triggers on push to `main` or manual dispatch
- Runs `cd Spock && hugo --minify` to build from Spock directory
- Adds `CNAME` file for custom domain (`anupanand.space`)
- Deploys `Spock/public/` to GitHub Pages
- Uses Hugo version `0.152.2` (extended)

**Important**: All `hugo` commands must run from `Spock/` directory, not project root.

## Content Conventions

### Travel Pages
Travel content uses a dedicated section at `Spock/content/travels/` with simple frontmatter:
```markdown
+++
title = "Iceland Adventure"
date = 2023-09-20
+++
```

### Academic Content (Research/Publications)
Research pages use raw HTML for formatting with specific patterns:
- Publication entries: `<b>Title</b><br>Authors<br><a href="journal-link">Journal</a><br><a href="arxiv">arXiv</a><br>Date`
- Section dividers: `<hr>` between research areas
- External links: `target="_self"` for internal-like navigation

## Performance Philosophy

This theme prioritizes **zero-JavaScript operation** and extreme performance:
- Styles inlined in `<head>` via [layouts/partials/styles.html](layouts/partials/styles.html)
- Web fonts load with `media="print" onload="this.media='all'"` for async loading
- No build-time JS bundling or asset pipeline
- Only optional JS: KaTeX (when `math = true`) and theme toggle

When adding features, maintain these constraints - prefer CSS-only solutions.

## Hugo Configuration Notes

Key settings in [Spock/config.toml](Spock/config.toml):
- `baseurl = "https://anupanand.space/"` - custom domain
- `canonifyurls = true` - all URLs absolute
- `disableKinds = ["taxonomy", "term"]` - no tags/categories pages
- `[markup.goldmark.renderer] unsafe = true` - allows raw HTML in markdown
- `theme = "black-and-light"` - references root theme (not submodule)

## Common Tasks

**Add new page**: Create `Spock/content/newpage.md` → auto-generates `/newpage/` URL
**Modify navigation**: Edit `[[menu.main]]` entries in `Spock/config.toml`
**Override theme layout**: Copy from `/layouts/` to `Spock/layouts/` and modify
**Add static files**: Place in `Spock/static/` → accessible at site root
**Change styles**: Edit CSS in `Spock/layouts/partials/styles.html` (overrides theme)
