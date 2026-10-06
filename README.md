# Hugo Static Site Generator (SSG)

A static site built with **Hugo**, the Go-based static site generator, featuring a custom theme built from scratch.

The project explores building a lightweight, content-focused website with Hugo's templating system rather than relying on a pre-built theme or JavaScript frontend framework.

## Features

- Custom Hugo theme — `zenshin-impact`
- Responsive layouts
- Light and dark mode
- Markdown content rendering
- Syntax highlighting
- Custom inline code and table styling
- Taxonomies and tags
- Post navigation
- RSS feed
- Social sharing
- Static deployment with Vercel

## Built With

- **Hugo** — Go-based static site generator
- **Go Templates** — layouts and reusable partials
- **HTML**
- **CSS**
- **Markdown**
- **Vercel** — deployment

## Project Structure

```text
.
├── content/                 # Markdown content
├── static/                  # Images and static assets
├── themes/
│   └── zenshin-impact/      # Custom Hugo theme
├── hugo.toml                # Hugo configuration
└── vercel.json              # Deployment configuration
```

The custom theme contains the site's layouts, partials, navigation, post rendering, responsive styling, and light/dark appearance.

## Local Development

Make sure Hugo is installed, then run:

```bash
hugo server -D
```

The development server will be available at:

```text
http://localhost:1313
```

## Build

Generate the production static site with:

```bash
hugo
```

Hugo generates the final static files into the `public/` directory.

## Live Site

**https://zenshin.blog**
