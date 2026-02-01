# ComBank CMS

A full-featured content management system for building modern websites, blogs, and portfolios. Combines [Payload CMS](https://payloadcms.com/) backend with [Next.js](https://nextjs.org/) frontend.

## Tech Stack

- **Backend**: Payload CMS with PostgreSQL
- **Frontend**: Next.js 15, React 19
- **Styling**: Tailwind CSS with PostCSS
- **Rich Text**: Lexical editor with custom blocks
- **Forms**: React Hook Form + form builder plugin
- **SEO**: Built-in SEO optimization
- **Search**: Full-text search functionality
- **UI Components**: Radix UI with custom components

## Key Features

- **Authentication & Authorization** - User auth with role-based access control
- **Content Collections** - Posts, Pages, Categories, Media, Users
- **Layout Builder** - Drag-and-drop block system (Banner, CTA, Code, Media, etc.)
- **Draft & Publish Workflow** - Schedule and preview content before publishing
- **Live Preview** - Real-time preview while editing
- **SEO Management** - Meta titles, descriptions, sitemap generation
- **Redirects** - URL redirect management
- **Responsive Design** - Mobile, tablet, and desktop previews

## Project Structure

```
src/
├── collections/          # Data schemas (Posts, Pages, Users, Media, Categories)
├── blocks/              # Reusable content blocks
├── components/          # React components (Layout, Forms, Cards, etc.)
├── app/                 # Next.js app routes (frontend + Payload admin)
├── fields/              # Custom field configurations
├── heros/               # Hero section variants
├── hooks/               # Custom React hooks
├── utilities/           # Helper functions
└── access/              # Access control rules
```

## Quick Start

### Prerequisites

- Node.js 18+
- pnpm

### Installation

```bash
pnpm install
```

### Development

```bash
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000) to start. The admin panel is at `/admin`.

## Available Scripts

```bash
pnpm dev              # Start development server
pnpm build            # Build for production
pnpm start            # Start production server
pnpm lint             # Run ESLint
pnpm lint:fix         # Fix linting issues
pnpm payload          # Payload CLI commands
```

## Database

Uses PostgreSQL with Payload's database adapter. Migrations are automatically handled in development mode. For production:

```bash
pnpm payload migrate:create  # Create migration
pnpm payload migrate         # Run migrations
```

## Collections

- **Posts** - Blog articles with Lexical editor, SEO, and layout builder
- **Pages** - Landing pages with customizable layout blocks
- **Categories** - Taxonomy for organizing posts (supports nesting)
- **Media** - Asset management with image optimization
- **Users** - Authentication and user management

## Layout Blocks

Reusable content blocks for pages and posts:

- Banner
- Content
- Call To Action
- Media
- Code (with syntax highlighting)
- Archive

## Global Data

- **Header** - Navigation and branding
- **Footer** - Links and metadata

## Plugins & Features

- **SEO** - Meta management, structured data, sitemaps
- **Search** - Full-text search with Lexical integration
- **Redirects** - URL redirect management
- **Form Builder** - Dynamic form creation
- **Live Preview** - Real-time editing with draft preview

## Deployment

### Build for Production

```bash
pnpm build && pnpm start
```

### Deploy to Payload Cloud

```bash
# One-click deployment via https://payloadcms.com
```

### Deploy to Vercel

Uses Next.js with PostgreSQL adapter. Set environment variables:

- `POSTGRES_URL` - Database connection string
- `NEXT_PUBLIC_SERVER_URL` - Frontend URL

## Environment Variables

Create `.env` file with:

```env
DATABASE_URI=postgresql://user:password@localhost:5432/combank
PAYLOAD_SECRET=your-secret-key
NEXT_PUBLIC_SERVER_URL=http://localhost:3000
```

## License

MIT

You can also deploy your app manually, check out the [deployment documentation](https://payloadcms.com/docs/production/deployment) for full details.

## Questions

If you have any issues or questions, reach out to us on [Discord](https://discord.com/invite/payload) or start a [GitHub discussion](https://github.com/payloadcms/payload/discussions).
