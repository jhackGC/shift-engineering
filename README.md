# Shift Engineering Blog

A minimal GitHub Pages engineering blog using Jekyll and Markdown.

## Live Site

Once GitHub Pages is enabled, your blog will be available at:
`https://jhackGC.github.io/shift-engineering/`

## Enabling GitHub Pages

To enable GitHub Pages for this repository:

1. Go to your repository on GitHub
2. Click on **Settings** (top navigation)
3. Scroll down to the **Pages** section (left sidebar)
4. Under **Source**, select the branch you want to deploy (typically `main` or `master`)
5. Click **Save**
6. GitHub will build and deploy your site automatically (this may take 1-2 minutes)
7. Your site URL will appear at the top of the Pages section

## Project Structure

```
shift-engineering/
├── _config.yml          # Jekyll configuration
├── index.md             # Blog home page
├── _posts/              # Blog posts directory
│   └── YYYY-MM-DD-title.md
└── README.md            # This file
```

## Adding New Posts

To create a new blog post:

1. Create a new Markdown file in the `_posts/` directory
2. Name it using the format: `YYYY-MM-DD-title.md` (e.g., `2026-02-10-my-post.md`)
3. Add YAML front matter at the top of the file:

```markdown
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD
categories: category1 category2
---

# Your Post Title

Your content here...
```

4. Write your content using standard Markdown
5. Commit and push to GitHub
6. GitHub Pages will automatically build and deploy your changes

## Markdown Features

This blog supports:

- **Headers**: `# H1`, `## H2`, `### H3`, etc.
- **Lists**: Ordered and unordered
- **Links**: `[text](url)`
- **Images**: `![alt](image-url)`
- **Code blocks**: Fenced code blocks with syntax highlighting
- **Tables**: Standard Markdown tables

### Code Blocks Example

Use fenced code blocks with language specification for syntax highlighting:

````markdown
```python
def hello_world():
    print("Hello, World!")
```

```javascript
const greeting = () => {
  console.log("Hello, World!");
};
```
````

## Local Development (Optional)

To preview your site locally:

1. Install Ruby and Bundler
2. Create a `Gemfile`:
   ```ruby
   source 'https://rubygems.org'
   gem 'github-pages', group: :jekyll_plugins
   ```
3. Run: `bundle install`
4. Run: `bundle exec jekyll serve`
5. Visit `http://localhost:4000`

**Note**: Local development is optional. GitHub Pages will build your site automatically when you push changes.

## Theme and Styling

This blog uses the default [Minima](https://github.com/jekyll/minima) theme, which provides:

- Clean, responsive layout
- Syntax highlighting for code blocks
- Mobile-friendly design
- RSS feed generation

All styling is handled by the theme—no custom CSS required.

## Configuration

Edit `_config.yml` to customize:

- Site title and description
- Author information
- Build settings
- Theme selection

Changes to `_config.yml` require restarting Jekyll (local dev) or pushing to GitHub (GitHub Pages).

## Resources

- [Jekyll Documentation](https://jekyllrb.com/docs/)
- [GitHub Pages Documentation](https://docs.github.com/pages)
- [Markdown Guide](https://www.markdownguide.org/)
- [Minima Theme](https://github.com/jekyll/minima)