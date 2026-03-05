# Retro Tech Hugo Theme

A clean, pastel blue Hugo theme designed for technical blogs about retro electronics, low-level programming, and IT humor. Features a terminal-inspired header and code-friendly design.

![Theme Preview](screenshot.png)

## Features

- 📟 **Retro terminal aesthetic** with pastel blue color scheme
- 🖼️ **Custom logo support** with separate light/dark mode images (120x120px)
- 🌙 **Dark mode toggle** with persistent preference storage
- 💻 **Code-first design** with excellent syntax highlighting support
- 🧮 **LaTeX math support** via MathJax for equations and formulas
- 🎮 **Commodore-style tag labels** with retro gradient effects
- 🖼️ **Featured images** on posts with text wrapping (square format)
- 📱 **Fully responsive** - works great on mobile and desktop
- 🎨 **Pastel blue palette** - easy on the eyes for long reading sessions
- ⚡ **Fast and lightweight** - minimal JavaScript, optimized CSS
- 🏷️ **Tag system** for organizing content
- 📄 **Clean typography** optimized for technical writing
- 🔍 **SEO-friendly** with proper semantic HTML
- 📅 **Date-first post listings** in YYYY-MM-DD format

## Color Palette

### Light Mode (Default)
- **Background**: `#f5f7fa` - Light blue-grey
- **Primary**: `#7ba7d6` - Soft blue
- **Accent**: `#b8d4eb` - Light sky blue
- **Text**: `#2c3e50` - Dark blue-grey
- **Terminal**: `#1e2836` - Deep blue-black

### Dark Mode
- **Background**: `#1a1f2e` - Dark navy
- **Primary**: `#8bb4e5` - Bright pastel blue
- **Accent**: `#3d4e6a` - Muted slate
- **Text**: `#e4e8f0` - Light grey-blue
- **Terminal**: `#1e2836` - Consistent terminal color

The theme includes a toggle button (🌙/☀️) in the navigation that lets users switch between light and dark modes. The preference is saved in localStorage and persists across sessions.

## Installation

### Option 1: Git Submodule (Recommended)

```bash
cd your-hugo-site
git submodule add https://github.com/yourusername/retro-tech-theme themes/retro-tech-theme
```

### Option 2: Manual Download

1. Download the theme from GitHub
2. Extract to `themes/retro-tech-theme` in your Hugo site directory

### Option 3: Git Clone

```bash
cd your-hugo-site/themes
git clone https://github.com/yourusername/retro-tech-theme
```

## Configuration

Add to your `hugo.toml` or `config.toml`:

```toml
theme = "retro-tech-theme"

[params]
  description = "Your site description"
  tagline = "cd ~/projects && ./code.sh"
  intro = "Brief introduction to your blog"
  syntaxHighlight = true
  math = true
  mainSections = ["posts"]
  
  # Logo images (120x120px recommended)
  logo = "/images/logo-light.png"
  logoDark = "/images/logo-dark.png"
  
  # Optional social links
  [[params.social]]
    name = "GitHub"
    url = "https://github.com/yourusername"
  
  [[params.social]]
    name = "Email"
    url = "mailto:you@example.com"

# Main menu
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
```

## Content Structure

### Creating Posts

```bash
hugo new posts/my-first-post.md
```

Example post frontmatter:

```yaml
---
title: "Building a Z80 Computer"
date: 2026-01-15
author: "Your Name"
tags: ["z80", "retro-computing", "hardware"]
image: "/images/my-post-image.jpg"  # Optional featured image (square recommended)
---
```

**Featured Images**: Add an `image` parameter to show a square image at the start of your post. The text will wrap around it automatically. Recommended size: 300x300px or larger square format.

### Recommended Content Sections

- `content/posts/` - Blog posts
- `content/about/` - About page
- `content/projects/` - Project showcase

## Customization

### Colors

Edit `static/css/style.css` and modify the CSS variables at the top:

```css
:root {
    --color-primary: #7ba7d6;
    --color-bg: #f5f7fa;
    /* ... more variables */
}
```

### Typography

The theme uses system fonts by default. To use custom fonts, modify:

```css
:root {
    --font-main: 'Your Font', sans-serif;
    --font-mono: 'Your Mono Font', monospace;
}
```

### Terminal ASCII Art

Edit `layouts/partials/header.html` to customize the terminal ASCII art.

## Syntax Highlighting

The theme supports Hugo's built-in syntax highlighting. Configure in `hugo.toml`:

```toml
[markup]
  [markup.highlight]
    style = "monokai"  # or atom-one-dark, dracula, etc.
    lineNos = true
    lineNumbersInTable = true
```

## LaTeX Math Support

The theme includes MathJax 3 for rendering mathematical equations. Enable it in `hugo.toml`:

```toml
[params]
  math = true
```

### Usage in Posts

**Inline math** - Use single dollar signs or `\(...\)`:
```markdown
The equation $E = mc^2$ is Einstein's famous formula.
Or: The value of \(\pi \approx 3.14159\).
```

**Display math** - Use double dollar signs or `\[...\]`:
```markdown
$$
\int_{-\infty}^{\infty} e^{-x^2} dx = \sqrt{\pi}
$$

Or:

\[
F(k) = \sum_{n=0}^{N-1} f(n) e^{-i 2\pi k n / N}
\]
```

### Supported LaTeX

MathJax supports most standard LaTeX math commands including:
- Greek letters: `\alpha`, `\beta`, `\gamma`, etc.
- Operators: `\sum`, `\int`, `\prod`, `\lim`, etc.
- Fractions: `\frac{a}{b}`
- Superscripts/subscripts: `x^2`, `x_i`
- Matrices, aligned equations, and more

See the example post `fourier-transform-math.md` for a complete demonstration.

## Supported Content Types

- **Posts** - Blog articles with full styling
- **Pages** - Static pages (About, Contact, etc.)
- **Lists** - Archive and tag pages
- **Images** - Automatic responsive sizing
- **Code blocks** - Excellent syntax highlighting
- **Tables** - Styled for technical documentation

## Browser Support

- Chrome/Edge (latest 2 versions)
- Firefox (latest 2 versions)
- Safari (latest 2 versions)
- Mobile browsers (iOS Safari, Chrome Android)

## Performance

- No jQuery or heavy frameworks
- Minimal JavaScript (only for syntax highlighting)
- Optimized CSS (~15KB)
- Fast page loads

## Development

To work on the theme:

```bash
# Clone the repository
git clone https://github.com/yourusername/retro-tech-theme
cd retro-tech-theme

# Test with the example site
cd exampleSite
hugo server --themesDir ../..
```

## Screenshots

### Homepage
Clean, card-based layout with terminal-style header

### Single Post
Optimized for reading technical content with code blocks

### Mobile View
Fully responsive design

## Credits

- Created for technical bloggers and retro computing enthusiasts
- Inspired by terminal emulators and classic computing interfaces
- Color palette designed for long reading sessions

## License

MIT License - feel free to use and modify for your projects!

## Support

- [GitHub Issues](https://github.com/yourusername/retro-tech-theme/issues)
- [Documentation](https://github.com/yourusername/retro-tech-theme/wiki)

## Changelog

### v1.0.0 (2026-01-29)
- Initial release
- Terminal-inspired header design
- Pastel blue color scheme
- Full responsive layout
- Syntax highlighting support
- Tag system
- Post navigation

---

Built with ❤️ for the retro computing community
