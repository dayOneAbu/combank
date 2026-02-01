# ComBank CMS - Architecture & Implementation Details

## System Architecture

### Frontend/Backend Split

- **(frontend)** - Next.js 15 static site generation & streaming
- **(payload)** - Payload admin panel at `/admin` route

Both coexist in single Next.js application with app router.

## Collections & Data Models

### Users

- Authentication enabled
- Admin panel access control
- Roles: Authenticated users can create/edit/delete content
- Timestamps: Automatically tracked

### Posts

- **Features**: Draft/publish workflow, SEO optimization, live preview, Lexical editor
- **Blocks**: Banner, Code, MediaBlock (layout builder)
- **Fields**:
  - `title` (required)
  - `heroImage` (upload)
  - `content` (rich text with Lexical)
  - `categories` (relationship)
  - `authors` (relationship)
  - `meta` (SEO plugin: title, description, image)
- **Access**: Authenticated can create/edit, anyone can read published
- **Hooks**: Auto-revalidation on publish/delete

### Pages

- **Features**: Draft workflow, custom layouts, hero section
- **Blocks**: CallToAction, Content, MediaBlock, Archive, FormBlock
- **Fields**:
  - `title` (required)
  - `hero` section (configurable type)
  - `layout` (blocks array - required)
  - `meta` (SEO)
- **Access**: Authenticated create/edit, anyone read published
- **Hooks**: Auto-revalidation

### Categories

- Nested taxonomy (via Nested Docs plugin)
- Supports hierarchies: "News > Technology"
- Slug generation with custom logic
- Publicly readable

### Media

- Upload destination: `/public/media`
- Features:
  - Focal point selection
  - Automatic image resizing
  - Thumbnail generation (300x300)
  - Alt text & caption (Lexical)
  - Publicly accessible

## Access Control Patterns

### Access Rules

```
authenticated     → User exists (req.user is truthy)
authenticatedOrPublished → User exists OR _status === 'published'
anyone           → All access allowed
```

### Default Access Model

| Collection | Create        | Read            | Update        | Delete        |
| ---------- | ------------- | --------------- | ------------- | ------------- |
| Posts      | Authenticated | AuthOrPublished | Authenticated | Authenticated |
| Pages      | Authenticated | AuthOrPublished | Authenticated | Authenticated |
| Categories | Authenticated | Anyone          | Authenticated | Authenticated |
| Media      | Authenticated | Anyone          | Authenticated | Authenticated |
| Users      | Authenticated | Authenticated   | Authenticated | Authenticated |

## Plugins Architecture

### Active Plugins

1. **Redirects Plugin** (`@payloadcms/plugin-redirects`)
   - Collections: pages, posts
   - Manages URL redirects
   - Custom field override for rebuild warning

2. **Nested Docs Plugin** (`@payloadcms/plugin-nested-docs`)
   - Collections: categories
   - Supports hierarchical taxonomies
   - URL generation: builds full path from parent chain

3. **SEO Plugin** (`@payloadcms/plugin-seo`)
   - Auto-generates: `{title} | Payload Website Template`
   - Auto-generates: `{serverURL}/{slug}`
   - Fields: title, description, image (with og size)

4. **Form Builder Plugin** (`@payloadcms/plugin-form-builder`)
   - Disabled: Payment fields
   - Rich text editor for confirmation messages
   - Used in FormBlock on pages

5. **Search Plugin** (`@payloadcms/plugin-search`)
   - Collections indexed: posts
   - Custom field overrides for search
   - Before sync hooks for data transformation

6. **Payload Cloud Plugin** (`@payloadcms/payload-cloud`)
   - Cloud deployment support

## Blocks System

### Block Architecture

Dynamic rendering system using `blockType` discriminator.

**Implemented Blocks:**

- `archive` - ArchiveBlock (collection listing)
- `content` - ContentBlock (rich text)
- `cta` - CallToActionBlock (calls-to-action)
- `formBlock` - FormBlock (dynamic forms)
- `mediaBlock` - MediaBlock (images/video)

**Block Config Pattern:**
Each block exports:

- `config.ts` - Payload field configuration
- `Component.tsx` - React render component
- `Component.client.tsx` - Optional client-side logic

**Rendering:**

```typescript
// RenderBlocks.tsx maps blockType to components
// Wraps each block in margin container
// Handles unknown block types gracefully
```

## Revalidation & Caching Strategy

### Hooks System

Automatic ISR (Incremental Static Revalidation) via Next.js.

**Hook Locations:**

- `src/collections/Posts/hooks/revalidatePost.ts` - Post changes
- `src/collections/Pages/hooks/revalidatePage.ts` - Page changes
- `src/Header/hooks/revalidateHeader.ts` - Header updates
- `src/Footer/hooks/revalidateFooter.ts` - Footer updates
- `src/hooks/revalidateRedirects.ts` - Redirect changes

**Pattern:**

```typescript
afterChange: [hook] // Runs after document publish/update
afterDelete: [hook] // Runs on deletion
// Calls revalidatePath() on affected routes
```

## Content Fields & Configuration

### Hero Types

Multiple hero variants configured in `src/heros/config.ts`:

- HighImpact
- MediumImpact
- LowImpact
- PostHero

Each with distinct styling and layout options.

### Lexical Editor Setup

Rich text editor with features:

- **Heading**: h1, h2, h3, h4
- **Blocks**: Custom block insertion (Banner, Code, MediaBlock)
- **Toolbar**: Fixed & inline options
- **Rules**: Horizontal rules

Default configuration in `src/fields/defaultLexical.ts`

### Slug Fields

Custom slug field with:

- Auto-generation from title
- Manual override capability
- Uniqueness validation
- Kebab-case formatting

## Frontend Features

### Metadata Generation

`generateMeta.ts` - Next.js metadata API integration:

- Extracts SEO fields from documents
- Generates Open Graph images
- Handles fallback OG image: `/website-template-OG.webp`
- Constructs page titles with branding suffix

### Component Library

**Layout Components:**

- AdminBar - Edit button for admin panel
- Header - Navigation with theme context
- Footer - Configurable links
- PageRange - Pagination controls

**Content Components:**

- RichText - Lexical renderer
- Card - Content card wrapper
- CollectionArchive - Collection listing with filters
- Media - Image with optimization
- Link - Navigation with Payload redirection support

**UI Components:** (shadcn/ui based)
Located in `src/components/ui/`

### Providers

Theme management via React Context:

- `ThemeProvider` - Global theme state
- `HeaderThemeProvider` - Header-specific theming

## TypeScript Configuration

### Path Aliases

```json
{
  "@payload-config": "./src/payload.config.ts",
  "react": "./node_modules/@types/react",
  "@/*": "./src/**"
}
```

### Strict Mode Enabled

- `strict: true`
- `noUncheckedIndexedAccess: true`
- `noImplicitOverride: true`
- `strictNullChecks: true`

## Environment Setup

### Required Variables

```
DATABASE_URI=postgres://user:pass@host:5432/combanketh
PAYLOAD_SECRET=<random-secret>
NEXT_PUBLIC_SERVER_URL=http://localhost:3000
CRON_SECRET=<optional-for-scheduled-jobs>
```

### Development Database

- Postgres with `push: true` for auto-migrations
- Production: `push: false` (requires explicit migrations)

## Styling & Design System

### Stack

- **CSS**: Tailwind CSS with custom theme variables
- **Variables**: CSS custom properties in `src/cssVariables.js`
- **UI Framework**: Radix UI primitives
- **Icons**: Lucide React
- **Typography**: Geist font family

### Admin Styling

- Custom SCSS: `src/app/(payload)/custom.scss`

## Search Implementation

### Configuration

- Indexed collections: Posts
- Field overrides in `src/search/fieldOverrides.ts`
- Pre-sync transformation: `src/search/beforeSync.ts`
- Frontend: `src/search/Component.tsx`

## File Upload Strategy

### Media Storage

- **Static Location**: `/public/media/`
- **Publicly Accessible**: Yes
- **Image Sizes**:
  - thumbnail: 300px
  - Additional sizes configurable
- **Focal Point**: Manual selection for cropping

## API Routes

### Payload Admin API

- Default route: `/api/payload`
- GraphQL support: `/api/graphql`
- REST endpoints: `/api/{collection}`

### Seed Endpoint

- Location: `src/endpoints/seed/`
- Manual trigger via admin panel
- Destructive: Drops and recreates database

## Build & Deployment

### Build Process

1. TypeScript compilation
2. Payload config validation
3. Next.js static/SSR export
4. Sitemap generation via next-sitemap

### Post-Build

- Sitemap auto-generated to `public/sitemap*.xml`

### Docker Support

- `Dockerfile` + `docker-compose.yml`
- Local PostgreSQL container
- Environment file support

## Performance Optimizations

### Image Optimization

- Sharp image processing
- Multiple size variants
- Focal point support
- Lazy loading via Next Image component

### Caching

- On-demand revalidation on content changes
- Static generation by default
- ISR for dynamic content

### Search Indexes

- Payload search plugin for full-text
- Configurable field indexing

## Development Workflow

### Key Scripts

```bash
pnpm dev              # Dev server with hot reload
pnpm build            # Production build
pnpm lint             # ESLint validation
pnpm generate:types   # Payload type generation
pnpm payload migrate:create  # Create migration
pnpm payload migrate  # Run migrations
```

### Type Safety

- Auto-generated types: `src/payload-types.ts`
- Full TypeScript support
- Type-safe API responses

## Admin Panel Customization

### Custom Components

- **BeforeLogin**: Welcome message
- **BeforeDashboard**: Dashboard greeting
- Custom row labels for collections
- Live preview with device breakpoints:
  - Mobile: 375x667
  - Tablet: 768x1024
  - Desktop (full width)
