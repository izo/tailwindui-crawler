# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is a Node.js crawler tool that downloads TailwindUI components from tailwindcss.com/plus. It scrapes authenticated pages and saves components in various formats (HTML, React JSX, Vue, Alpine.js) to a local output folder for offline use and version tracking.

## Development Commands

- `npm start` - Run the crawler with current .env configuration
- `npm install` - Install dependencies
- `node index.mjs` - Direct execution (same as npm start)

## Configuration

The tool requires a `.env` file with authentication and configuration:

**Required:**
- `EMAIL` - TailwindUI account email
- `PASSWORD` - TailwindUI account password

**Optional:**
- `OUTPUT` - Output directory (defaults to ./output)
- `LANGUAGES` - Comma-separated list: html,react,vue,alpine (defaults to html)
- `COMPONENTS` - Comma-separated list or 'all': marketing,application-ui,ecommerce
- `BUILDINDEX` - Set to 1 to generate preview index page
- `TEMPLATES` - Set to 1 to download template files
- `FORCE_UPDATE` - Set to 1 to overwrite existing components
- `DEBUG` - Set to 1 to enable verbose logging

## Architecture

### Core Files

- **index.mjs** - Main crawler script and entry point (ESM module)
- **utils.mjs** - Shared utilities (string manipulation, file operations)
- **transformers/** - Component transformation modules for different output formats

### Key Components

**Authentication & Networking:**
- HTTPS-based fetching with cookie session management
- Login flow handling with CSRF token management
- Retry logic with exponential backoff

**Data Processing Pipeline:**
1. Authenticate with tailwindcss.com/plus
2. Crawl UI blocks index page to discover components
3. Parse component data from embedded JSON in data-page attributes
4. Extract component code snippets for each requested language
5. Apply format-specific transformations
6. Save to organized directory structure

**Component Processing:**
- Cheerio for HTML parsing and manipulation
- Language-specific transformations (React JSX conversion, Alpine.js extraction)
- Intelligent React component naming based on path structure
- Component skipping to avoid re-downloading existing files

### Output Structure

```
output/
├── html/           # Raw HTML components
├── react/          # JSX components with proper naming
├── vue/            # Vue components
├── alpine/         # Alpine.js enhanced HTML
├── templates/      # Template ZIP files
├── preview/        # Local preview site
└── assets.json     # Asset tracking for incremental updates
```

### Transformer System

Each transformer in `transformers/` handles specific output format conversion:

- **convertReact.js** - HTML to JSX conversion with proper React attributes
- **convertVue.js** - HTML to Vue SFC conversion
- **stripAlpine.js** - Remove Alpine.js directives
- **addTailwindCss.js** - Inject Tailwind CSS imports
- And others for color changes, logo replacements, etc.

### Component Naming

React components use intelligent naming based on the component path:
- Extracts meaningful path segments (skips 'ui-blocks', 'index')
- Converts to PascalCase
- Replaces generic 'Example' function names with descriptive names

### Error Handling

- Comprehensive retry logic for network requests
- ETag-based caching to avoid unnecessary downloads
- Graceful handling of missing components or authentication failures
- Debug mode with detailed logging for troubleshooting

## GitHub Actions Integration

The tool supports automated GitHub Actions workflows for keeping private repositories synchronized with TailwindUI updates. See README for workflow configuration.

## Development Notes

- Uses ES modules (type: "module" in package.json)
- Node.js 16+ required
- Cheerio for server-side DOM manipulation
- Cookie-based session management for authentication
- Environment variable expansion support via dotenv-expand