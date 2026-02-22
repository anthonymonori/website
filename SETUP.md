# Local Development Setup

## Prerequisites

This project uses [jekyll-mermaid-prebuild](https://github.com/Texarkanine/jekyll-mermaid-prebuild) to render Mermaid diagrams at build time. This requires additional dependencies.

### Required Dependencies

1. **Ruby 3.4+** (for Jekyll, required by jekyll-mermaid-prebuild)
2. **Node.js 24+** (for mermaid-cli)
3. **mermaid-cli** (for diagram rendering)
4. **Puppeteer dependencies** (for headless Chrome)

### Installation

#### macOS

```bash
# Install Node.js (if using Homebrew)
brew install node

# Install mermaid-cli globally
npm install -g @mermaid-js/mermaid-cli

# Verify installation
mmdc --version
```

#### Linux (Debian/Ubuntu/WSL)

```bash
# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install mermaid-cli globally
npm install -g @mermaid-js/mermaid-cli

# Install Puppeteer dependencies
sudo apt-get update
sudo apt-get install -y libgbm1 libasound2t64 libatk1.0-0 \
  libatk-bridge2.0-0 libcups2 libdrm2 libxcomposite1 \
  libxdamage1 libxfixes3 libxrandr2 libxkbcommon0 \
  libpango-1.0-0 libcairo2 libnss3 libnspr4

# Verify installation
mmdc --version
```

#### Windows

```powershell
# Install Node.js from https://nodejs.org/

# Install mermaid-cli globally
npm install -g @mermaid-js/mermaid-cli

# Verify installation
mmdc --version
```

## Building the Site

```bash
# Install Ruby dependencies
gem install bundler
bundle install

# Build the site
bundle exec jekyll build

# Or serve locally with live reload
bundle exec jekyll serve --livereload
```

## Verifying Mermaid Diagrams

After building, check that SVG files are generated in `assets/svg/`:

```bash
ls -la assets/svg/
```

## Troubleshooting

### "mmdc not found"

Install mermaid-cli:
```bash
npm install -g @mermaid-js/mermaid-cli
```

### "Puppeteer cannot launch headless Chrome" (Linux)

Install the required system libraries:
```bash
sudo apt-get update
# Ubuntu 24.04+ uses libasound2t64, older versions use libasound2
sudo apt-get install -y libgbm1 libasound2t64 libatk1.0-0 \
  libatk-bridge2.0-0 libcups2 libdrm2 libxcomposite1 \
  libxdamage1 libxfixes3 libxrandr2 libxkbcommon0 \
  libpango-1.0-0 libcairo2 libnss3 libnspr4
```

### Diagrams not rendering

1. Check build output for `MermaidPrebuild:` messages
2. Verify mmdc works: `mmdc -i test.mmd -o test.svg`
3. Clear cache: `rm -rf .jekyll-cache/jekyll-mermaid-prebuild/`
