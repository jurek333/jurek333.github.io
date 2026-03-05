# Installation Guide - Retro Tech Theme

## Quick Start

Follow these steps to get the Retro Tech theme running on your Hugo site:

### Step 1: Install Hugo

If you haven't already, install Hugo Extended:

```bash
# macOS
brew install hugo

# Windows (with Chocolatey)
choco install hugo-extended

# Linux
snap install hugo
```

### Step 2: Create a New Site (or use existing)

```bash
# Create new site
hugo new site my-tech-blog
cd my-tech-blog

# Initialize git (optional but recommended)
git init
```

### Step 3: Install the Theme

Choose one method:

#### Method A: Git Submodule (Recommended)

```bash
git submodule add https://github.com/yourusername/retro-tech-theme themes/retro-tech-theme
```

#### Method B: Download and Extract

1. Download the theme as a ZIP
2. Extract to `themes/retro-tech-theme`

### Step 4: Configure Your Site

Create or edit `hugo.toml` in your site root:

```toml
baseURL = "https://your-site.com/"
languageCode = "en-us"
title = "My Retro Tech Blog"
theme = "retro-tech-theme"

[markup]
  [markup.highlight]
    style = "monokai"
    lineNos = true
    lineNumbersInTable = true

[params]
  description = "Adventures in retro computing and low-level programming"
  tagline = "cd ~/adventures && ./explore.sh"
  intro = "Welcome to my technical blog about **retro electronics** and **programming**!"
  syntaxHighlight = true
  math = true  # Enable LaTeX math rendering with MathJax
  mainSections = ["posts"]
  
  # Logo images (120x120px recommended)
  logo = "/images/logo-light.png"
  logoDark = "/images/logo-dark.png"

[[menu.main]]
  name = "Posts"
  pre = "📝 "
  url = "/posts/"
  weight = 1

[[menu.main]]
  name = "Tags"
  pre = "🏷️ "
  url = "/tags/"
  weight = 2

[[menu.main]]
  name = "About"
  pre = "ℹ️ "
  url = "/about/"
  weight = 3
```

### Step 5: Create Your First Post

```bash
hugo new posts/my-first-post.md
```

Edit the file in `content/posts/my-first-post.md`:

```markdown
---
title: "My First Post"
date: 2026-01-29
author: "Your Name"
tags: ["getting-started", "hello-world"]
draft: false
image: "/images/my-featured-image.jpg"  # Optional: square image for text wrapping
---

Welcome to my blog! This is my first post using the Retro Tech theme.

## Code Example

```python
def hello_world():
    print("Hello, Retro Tech!")
    
hello_world()
```

Stay tuned for more content!
```

### Step 6: Add Your Logo (Optional)

Create logo images (120x120px recommended) and place them in `static/images/`:
- `static/images/logo-light.png` - Logo for light mode
- `static/images/logo-dark.png` - Logo for dark mode

If you only have one logo, just set `logo` in your config and omit `logoDark`.

### Step 7: Run the Development Server

```bash
hugo server -D
```

Visit `http://localhost:1313` to see your site!

### Step 8: Build for Production

When ready to deploy:

```bash
hugo
```

This creates a `public/` directory with your static site.

## Customization

### Change Colors

Edit `themes/retro-tech-theme/static/css/style.css`:

```css
:root {
    --color-primary: #7ba7d6;  /* Change to your preferred blue */
    --color-bg: #f5f7fa;       /* Background color */
}
```

### Add Social Links

In `hugo.toml`:

```toml
[[params.social]]
  name = "GitHub"
  url = "https://github.com/yourusername"

[[params.social]]
  name = "Twitter"
  url = "https://twitter.com/yourusername"

[[params.social]]
  name = "Email"
  url = "mailto:you@example.com"
```

### Customize Terminal Header

Edit `themes/retro-tech-theme/layouts/partials/header.html` to change the ASCII art or terminal appearance.

### Using LaTeX Math

Enable math in your config:
```toml
[params]
  math = true
```

Then use in your posts:
```markdown
Inline math: $E = mc^2$

Display math:
$$
\sum_{i=1}^{n} i = \frac{n(n+1)}{2}
$$
```

### Dark Mode

The theme includes automatic dark mode support! Users can toggle between light and dark modes using the 🌙/☀️ button in the navigation. The preference is automatically saved and will persist across visits.

No configuration needed - it works out of the box!

## Deployment

### GitHub Pages

1. Create `.github/workflows/hugo.yml`:

```yaml
name: Deploy Hugo site

on:
  push:
    branches: ["main"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: recursive
      
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true
      
      - name: Build
        run: hugo --minify
      
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

### Netlify

1. Push your site to GitHub
2. Connect Netlify to your repository
3. Build command: `hugo --minify`
4. Publish directory: `public`

### Vercel

1. Push your site to GitHub
2. Import project in Vercel
3. Framework preset: Hugo
4. Deploy!

## Troubleshooting

### Theme not found

Make sure the theme is in `themes/retro-tech-theme` and `hugo.toml` has:
```toml
theme = "retro-tech-theme"
```

### Styles not loading

Clear your browser cache or run:
```bash
hugo server --noHTTPCache
```

### Syntax highlighting not working

Ensure you have this in `hugo.toml`:
```toml
[params]
  syntaxHighlight = true

[markup]
  [markup.highlight]
    style = "monokai"
```

## Getting Help

- Check the [README](README.md) for full documentation
- Open an issue on GitHub
- Review the example site in `exampleSite/`

Happy blogging! 🚀
