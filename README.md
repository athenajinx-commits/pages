# GitHub Pages Documentation Site

This repository contains multiple single-page HTML documentation files, organized and listed on the main index page.

## Structure

```
pages/
├── _config.yml          # Jekyll configuration (minimal setup)
├── docs/
│   ├── index.html       # Main index page listing all documentation
│   └── *.html          # Individual documentation pages
└── README.md           # This file
```

## Adding New Pages

To add a new documentation page:

1. **Create your HTML file** in the `docs/` directory
   - Name it descriptively (e.g., `my-topic.html`)
   - Make it a complete, standalone HTML page

2. **Add it to the index** by editing `docs/index.html`
   - Find the `<ul class="page-list">` section
   - Add a new `<li class="page-item">` entry with:
     - A link to your HTML file
     - A descriptive title
     - A brief description
     - The filename

   Example:
   ```html
   <li class="page-item">
       <a href="my-topic.html">
           <h2>My Topic Title</h2>
           <p>Brief description of what this page covers</p>
           <span class="filename">my-topic.html</span>
       </a>
   </li>
   ```

## GitHub Pages Setup

This repository uses a minimal Jekyll setup for GitHub Pages:

- **Jekyll Theme**: `minima` (minimal, clean theme)
- **Configuration**: See `_config.yml`
- **No build required**: GitHub Pages will automatically build and serve your site

### Enabling GitHub Pages

1. Go to your repository settings on GitHub
2. Navigate to "Pages" in the sidebar
3. Under "Source", select:
   - **Branch**: `main` (or your default branch)
   - **Folder**: `/docs` (if your pages are in the docs folder) or `/` (if in root)
4. Click "Save"

Your site will be available at: `https://[username].github.io/[repository-name]/`

## Customization

### Styling
- The index page has its own embedded CSS for a modern, responsive design
- Individual pages can have their own styling (like `azure-confluent-private-link.html`)
- To use a consistent theme across all pages, you can create a shared CSS file

### Jekyll Features
The `_config.yml` enables basic Jekyll features. You can:
- Add more Jekyll plugins if needed
- Customize the theme
- Use Jekyll layouts (create `_layouts/` folder)
- Add navigation, collections, etc.

## Notes

- All HTML files should be complete, standalone pages
- The index page uses a simple list structure - easy to maintain
- No build step required - just commit and push to GitHub
- GitHub Pages will automatically rebuild when you push changes
