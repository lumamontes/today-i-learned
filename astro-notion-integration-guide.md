# Astro + Notion Integration Guide

## Overview
This guide shows how to integrate your Notion TIL database with your Astro portfolio website to automatically pull and display content.

## Prerequisites
- Astro project set up
- Notion database with TIL entries
- Notion integration token

## Step 1: Install Dependencies

```bash
pnpm add @notionhq/client
pnpm add -D @types/node
```

## Step 2: Environment Variables

Create/update `.env` file:
```env
NOTION_TOKEN=your_notion_token_here
NOTION_DATABASE_ID=your_database_id_here
```

Add to `.env.example`:
```env
NOTION_TOKEN=
NOTION_DATABASE_ID=
```

## Step 3: Notion API Utility

Create `src/lib/notion.ts`:

```typescript
import { Client } from '@notionhq/client';

const notion = new Client({
  auth: import.meta.env.NOTION_TOKEN,
});

export interface TILEntry {
  id: string;
  title: string;
  content: string;
  tags: string[];
  date: string;
  published: boolean;
  slug: string;
  sourceUrl?: string;
}

export async function getTILEntries(): Promise<TILEntry[]> {
  try {
    const response = await notion.databases.query({
      database_id: import.meta.env.NOTION_DATABASE_ID,
      filter: {
        property: 'Published',
        checkbox: {
          equals: true,
        },
      },
      sorts: [
        {
          property: 'Date',
          direction: 'descending',
        },
      ],
    });

    return await Promise.all(
      response.results.map(async (page: any) => {
        // Get page content
        const blocks = await notion.blocks.children.list({
          block_id: page.id,
        });

        // Extract text content from blocks
        const content = blocks.results
          .map((block: any) => {
            if (block.type === 'paragraph') {
              return block.paragraph.rich_text
                .map((text: any) => text.plain_text)
                .join('');
            }
            return '';
          })
          .filter(Boolean)
          .join('\n\n');

        return {
          id: page.id,
          title: page.properties.Title.title[0]?.plain_text || '',
          content,
          tags: page.properties.Tags.multi_select.map((tag: any) => tag.name),
          date: page.properties.Date.date?.start || '',
          published: page.properties.Published.checkbox,
          slug: page.properties.Slug.rich_text[0]?.plain_text || '',
          sourceUrl: page.properties.Source?.url || undefined,
        };
      })
    );
  } catch (error) {
    console.error('Error fetching TIL entries:', error);
    return [];
  }
}

export async function getTILEntry(slug: string): Promise<TILEntry | null> {
  const entries = await getTILEntries();
  return entries.find(entry => entry.slug === slug) || null;
}
```

## Step 4: TIL Index Page

Create `src/pages/til/index.astro`:

```astro
---
import Layout from '../../layouts/Layout.astro';
import { getTILEntries } from '../../lib/notion';

const tilEntries = await getTILEntries();
---

<Layout title="Today I Learned">
  <main class="max-w-4xl mx-auto px-4 py-8">
    <h1 class="text-4xl font-bold mb-8">Today I Learned</h1>
    
    <div class="grid gap-6">
      {tilEntries.map((entry) => (
        <article class="border rounded-lg p-6 hover:shadow-lg transition-shadow">
          <h2 class="text-2xl font-semibold mb-2">
            <a href={`/til/${entry.slug}`} class="hover:text-blue-600">
              {entry.title}
            </a>
          </h2>
          
          <div class="flex items-center gap-4 mb-3 text-sm text-gray-600">
            <time>{new Date(entry.date).toLocaleDateString()}</time>
            {entry.sourceUrl && (
              <a href={entry.sourceUrl} class="text-blue-500 hover:underline" target="_blank">
                Source
              </a>
            )}
          </div>
          
          <div class="flex flex-wrap gap-2 mb-4">
            {entry.tags.map((tag) => (
              <span class="bg-gray-200 px-2 py-1 rounded-md text-xs">
                {tag}
              </span>
            ))}
          </div>
          
          <p class="text-gray-700 line-clamp-3">
            {entry.content.substring(0, 200)}...
          </p>
        </article>
      ))}
    </div>
  </main>
</Layout>
```

## Step 5: Individual TIL Page

Create `src/pages/til/[slug].astro`:

```astro
---
import Layout from '../../layouts/Layout.astro';
import { getTILEntry, getTILEntries } from '../../lib/notion';

export async function getStaticPaths() {
  const entries = await getTILEntries();
  
  return entries.map((entry) => ({
    params: { slug: entry.slug },
    props: { entry },
  }));
}

const { entry } = Astro.props;

if (!entry) {
  return Astro.redirect('/404');
}
---

<Layout title={entry.title}>
  <article class="max-w-3xl mx-auto px-4 py-8">
    <header class="mb-8">
      <h1 class="text-4xl font-bold mb-4">{entry.title}</h1>
      
      <div class="flex items-center gap-4 mb-4 text-gray-600">
        <time>{new Date(entry.date).toLocaleDateString()}</time>
        {entry.sourceUrl && (
          <a href={entry.sourceUrl} class="text-blue-500 hover:underline" target="_blank">
            View Source
          </a>
        )}
      </div>
      
      <div class="flex flex-wrap gap-2">
        {entry.tags.map((tag) => (
          <span class="bg-gray-200 px-3 py-1 rounded-md text-sm">
            {tag}
          </span>
        ))}
      </div>
    </header>
    
    <div class="prose max-w-none">
      {entry.content.split('\n\n').map((paragraph) => (
        <p class="mb-4">{paragraph}</p>
      ))}
    </div>
    
    <footer class="mt-8 pt-4 border-t">
      <a href="/til" class="text-blue-500 hover:underline">
        ← Back to TIL
      </a>
    </footer>
  </article>
</Layout>
```

## Step 6: Add to Navigation

Add TIL link to your main navigation:

```astro
<!-- In your navigation component -->
<nav>
  <a href="/til">Today I Learned</a>
  <!-- other nav items -->
</nav>
```

## Step 7: RSS Feed (Optional)

Create `src/pages/til.xml.js`:

```javascript
import rss from '@astrojs/rss';
import { getTILEntries } from '../lib/notion';

export async function GET(context) {
  const entries = await getTILEntries();
  
  return rss({
    title: 'Today I Learned',
    description: 'My learning journey in tech',
    site: context.site,
    items: entries.map((entry) => ({
      title: entry.title,
      pubDate: new Date(entry.date),
      description: entry.content.substring(0, 200) + '...',
      link: `/til/${entry.slug}/`,
    })),
  });
}
```

## Step 8: Build Configuration

Update `astro.config.mjs` to include environment variables:

```javascript
import { defineConfig } from 'astro/config';

export default defineConfig({
  // ... other config
  vite: {
    define: {
      'import.meta.env.NOTION_TOKEN': JSON.stringify(process.env.NOTION_TOKEN),
      'import.meta.env.NOTION_DATABASE_ID': JSON.stringify(process.env.NOTION_DATABASE_ID),
    },
  },
});
```

## Step 9: Deployment

1. Add environment variables to your hosting platform (Vercel, Netlify, etc.)
2. Deploy your site

## Step 10: Automation (Optional)

To automatically rebuild when you add new TIL entries:

### Option A: Webhook + Build Hook
1. Set up a webhook in Notion (if available)
2. Configure it to trigger your deployment build hook

### Option B: Scheduled Builds
1. Set up scheduled builds on your hosting platform
2. Rebuild daily/weekly to pull new content

## Usage

1. Write new TIL entries in Notion
2. Mark them as "Published" when ready
3. Your website will pull them automatically on next build
4. For immediate updates, trigger a manual build

## Styling Notes

The examples use Tailwind CSS classes. Adjust the styling to match your existing design system.

## Troubleshooting

- **Build errors**: Ensure environment variables are set correctly
- **No content**: Check that entries are marked as "Published" in Notion
- **API errors**: Verify your Notion integration has access to the database

## Next Steps

- Add search functionality
- Implement tag filtering
- Add pagination for large numbers of entries
- Create a Books section following the same pattern