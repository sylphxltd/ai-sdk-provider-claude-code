# Documentation

VitePress documentation site for @sylphx/ai-sdk-provider-claude-code.

## Development

```bash
# Install dependencies
npm install

# Start dev server
npm run docs:dev
# Opens at http://localhost:5173
```

## Build

```bash
# Build for production
npm run docs:build

# Preview production build
npm run docs:preview
```

## Deployment

This documentation site is configured to deploy to Vercel at https://ai-sdk-provider-claude-code.sylphx.com

The `vercel.json` in the root directory configures the build:

- Build command: `cd docs && npm install && npm run docs:build`
- Output directory: `docs/.vitepress/dist`

### Deploy to Vercel

1. Connect the GitHub repository to Vercel
2. Configure the custom domain: `ai-sdk-provider-claude-code.sylphx.com`
3. Vercel will automatically build and deploy using the `vercel.json` configuration

## Structure

```
docs/
├── .vitepress/
│   └── config.mts          # VitePress configuration
├── guide/
│   ├── index.md            # Getting started
│   ├── installation.md     # Installation guide
│   ├── usage.md            # Usage patterns
│   ├── architecture.md     # How it works
│   └── models.md           # Model comparison
├── api/
│   ├── index.md            # API overview
│   ├── claude-code.md      # Main function reference
│   ├── provider-options.md # Configuration options
│   └── types.md            # TypeScript types
├── examples/
│   ├── index.md            # Examples overview
│   ├── basic-text.md       # Text generation
│   ├── streaming.md        # Streaming
│   ├── tools.md            # Tool calling
│   ├── thinking.md         # Extended thinking
│   └── conversations.md    # Multi-turn chat
└── index.md                # Landing page
```

## Content

All content is extracted and organized from the main README.md, providing:

- Comprehensive guides for getting started
- Detailed API documentation
- Practical examples for common use cases
- Architecture explanations
- Best practices

## License

MIT © Sylphx Limited
