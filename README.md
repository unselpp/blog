# My Blog

A minimal blog built with Astro, Markdown, and Shiki.

## Features

- ✨ Cream background with beautiful typography
- 📝 Write posts in Markdown
- 💻 Syntax highlighting with Shiki
- 🚀 Fast static site generation
- 📱 Responsive design

## Getting Started

1. Install dependencies:
   ```bash
   npm install
   ```

2. Start the development server:
   ```bash
   npm run dev
   ```

3. Open [http://localhost:4321](http://localhost:4321) in your browser.

## Writing Blog Posts

Create new blog posts by adding Markdown files to `src/content/blog/`.

Each post needs frontmatter at the top:

```markdown
---
title: "My Post Title"
description: "Optional description"
pubDate: 2025-01-15
tags: ["tag1", "tag2"]
---

Your content here...
```

## Building

Build for production:

```bash
npm run build
```

Preview the production build:

```bash
npm run preview
```

## Deployment

Deploy to Vercel or Netlify:

- **Vercel**: Connect your GitHub repo or run `vercel`
- **Netlify**: Connect your GitHub repo or run `netlify deploy`

