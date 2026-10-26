# Sanity CMS Interview Questions and Answers

A beginner-friendly interview guide with explanations, practical code examples, GROQ queries, and scenario-based answers.

> **Scope:** This document consolidates the 41 questions visible in the supplied screenshots. The original numbering is preserved. Explanations and code examples have been expanded where useful. Code is illustrative; replace project IDs, dataset names, field names, and secrets with your own values.

## Table of Contents

- [Sanity CMS basics — Questions 1–7](#sanity-cms-basics--questions-1-7)
- [Schema and content modeling — Questions 8–13](#schema-and-content-modeling--questions-8-13)
- [GROQ query language — Questions 14–20](#groq-query-language--questions-14-20)
- [Portable Text and media — Questions 21–23](#portable-text-and-media--questions-21-23)
- [Studio customization — Questions 24–26](#studio-customization--questions-24-26)
- [Next.js integration — Questions 27–31](#nextjs-integration--questions-27-31)
- [Security and deployment — Questions 32–36](#security-and-deployment--questions-32-36)
- [Scenario questions — Questions 37–41](#scenario-questions--questions-37-41)
- [Quick revision checklist](#quick-revision-checklist)

---

## Sanity CMS basics — Questions 1–7

### 1. What is Sanity CMS?

**Answer:** Sanity is a headless Content Management System (CMS). Content is stored in Sanity’s hosted Content Lake, and a frontend such as React or Next.js fetches it through APIs. Its two main parts are **Sanity Studio**, where editors create and manage content, and the **Content Lake**, where content is stored.

**Example:** A marketing editor creates a blog post in Studio. A Next.js website fetches the post using GROQ and displays it to visitors. The same content could also be used by a mobile app.

**Typical flow:**

```text
Editor → Sanity Studio → Content Lake → API/GROQ → Next.js website
```

### 2. What is a headless CMS? How is it different from a traditional CMS?

**Answer:**

- A traditional CMS, such as a typical WordPress setup, often combines content management with website presentation.
- A headless CMS manages content separately from the frontend. The frontend gets content through APIs.
- The same content can be reused on a website, mobile app, kiosk, or other channel.

**Example:** If a company has a website and a mobile app, both can fetch the same product description from Sanity while displaying it using different designs.

| Traditional CMS | Headless CMS |
|---|---|
| Content and presentation are often coupled | Content and presentation are separated |
| Theme usually controls website rendering | Developer builds the frontend separately |
| Content reuse across channels can require extra work | APIs make multi-channel reuse natural |

### 3. Why choose Sanity over other headless CMSs?

**Answer:** Sanity provides flexible content modeling written in JavaScript or TypeScript, GROQ for querying content, real-time collaboration, and a Studio interface developers can customize. It works well when content structures and editorial workflows need flexibility.

**Trade-off:** It usually requires more developer setup than a simpler, more opinionated CMS. The best choice depends on the team, content model, integrations, and budget.

### 4. What is Sanity Studio?

**Answer:** Sanity Studio is an open-source, React-based content editing application. Developers configure it using JavaScript or TypeScript schemas. It can be deployed using Sanity hosting or embedded in an application, for example at `/studio`.

**Example schema:**

```ts
// schemaTypes/post.ts
import { defineType, defineField } from 'sanity'

export const post = defineType({
  name: 'post',
  title: 'Blog Post',
  type: 'document',
  fields: [
    defineField({ name: 'title', title: 'Title', type: 'string' }),
    defineField({
      name: 'slug',
      title: 'Slug',
      type: 'slug',
      options: { source: 'title' },
    }),
    defineField({
      name: 'body',
      title: 'Body',
      type: 'array',
      of: [{ type: 'block' }],
    }),
  ],
})
```

### 5. What is the Content Lake?

**Answer:** The Content Lake is Sanity’s hosted content store. Content is represented as structured documents, commonly JSON-like records. Applications read and write data through Sanity APIs, subject to dataset access and token permissions.

**Example document:**

```json
{
  "_id": "post-hello-world",
  "_type": "post",
  "title": "Hello World",
  "slug": {
    "_type": "slug",
    "current": "hello-world"
  }
}
```

The `_type` identifies the document type; `_id` identifies the document.

### 6. What are a project, a dataset, and an API version?

**Answer:**

- **Project:** Your Sanity workspace, identified by a `projectId`.
- **Dataset:** A collection of content inside a project, often named `production`, `development`, or `staging`.
- **API version:** A date string, such as `2024-01-01`, that pins API behavior so future API changes are less likely to affect the app unexpectedly.

**Client example:**

```ts
import { createClient } from 'next-sanity'

export const client = createClient({
  projectId: process.env.NEXT_PUBLIC_SANITY_PROJECT_ID!,
  dataset: process.env.NEXT_PUBLIC_SANITY_DATASET!,
  apiVersion: '2024-01-01',
  useCdn: true,
})
```

Use your actual project ID and dataset. Pin an explicit API version rather than using a moving version.

### 7. What is the difference between draft and published documents?

**Answer:** When an editor changes a document, Sanity can save an unpublished draft. The draft allows editors to work without immediately changing the public website. When the editor publishes, the published version becomes available to normal published-content queries.

**Key points:**

- Drafts support editing and review before publication.
- A public frontend generally queries published content.
- Preview mode must be deliberately configured to access drafts and should be protected.

---

## Schema and content modeling — Questions 8–13

### 8. What is a schema?

**Answer:** A schema is the blueprint for your content. It defines the fields each content type has, their types, validation rules, and how editors work with them. A clear schema keeps data consistent and makes it easier to query and render.

**Example:**

```ts
import { defineType, defineField } from 'sanity'

export const post = defineType({
  name: 'post',
  title: 'Blog Post',
  type: 'document',
  fields: [
    defineField({
      name: 'title',
      title: 'Title',
      type: 'string',
      validation: Rule => Rule.required().max(80),
    }),
    defineField({
      name: 'slug',
      title: 'Slug',
      type: 'slug',
      options: { source: 'title' },
    }),
    defineField({
      name: 'body',
      title: 'Body',
      type: 'array',
      of: [{ type: 'block' }],
    }),
  ],
})
```

### 9. What are the common schema types?

**Answer:**

| Type | Typical use |
|---|---|
| `document` | Main content item, such as a post or product |
| `string` / `text` | Short or long text |
| `number`, `boolean`, `datetime` | Basic values |
| `slug` | URL-friendly ID, such as `my-first-post` |
| `image` / `file` | Media |
| `reference` | Link to another document, such as a post’s author |
| `object` | Group of fields inside a document |
| `array` | List of items |
| `block` | Rich-text block used by Portable Text |

**Example — reference and array fields:**

```ts
defineField({
  name: 'author',
  title: 'Author',
  type: 'reference',
  to: [{ type: 'author' }],
}),
defineField({
  name: 'tags',
  title: 'Tags',
  type: 'array',
  of: [{ type: 'string' }],
})
```

### 10. What is the difference between a reference and an object?

**Answer:**

- A **reference** points to a separate document that can be reused, such as an author shared by many posts.
- An **object** is embedded inside a document and usually belongs only to that document, such as an address stored on a store record.

**Rule of thumb:** Use a reference when the item needs to be managed independently or reused. Use an object when the data belongs to the parent document.

**Reference example:**

```ts
defineField({
  name: 'author',
  title: 'Author',
  type: 'reference',
  to: [{ type: 'author' }],
})
```

**Embedded object example:**

```ts
defineField({
  name: 'address',
  title: 'Address',
  type: 'object',
  fields: [
    { name: 'street', type: 'string', title: 'Street' },
    { name: 'city', type: 'string', title: 'City' },
  ],
})
```

### 11. How do you add validation?

**Answer:** Add a `validation` rule to a field. Sanity validates the value in Studio and shows an error when the rule is not satisfied.

**Example:**

```ts
defineField({
  name: 'title',
  title: 'Title',
  type: 'string',
  validation: Rule => Rule.required().max(80),
})
```

More examples:

```ts
// Required string with a minimum length
validation: Rule => Rule.required().min(3)

// Number within a range
validation: Rule => Rule.required().min(1).max(100)

// Required slug
validation: Rule => Rule.required()
```

For custom rules, return an error message when the value is invalid and `true` when it is valid.

### 12. How do you handle multilingual content?

**Answer:** Two common approaches are:

1. **Field-level translation:** Keep translations in one document, such as `title.en` and `title.hi`, or in a localized object.
2. **Document-level translation:** Create a separate document for each language and link the translations together, often using a translation plugin or shared translation identifier.

**How to choose:** Field-level translation can be convenient for small, tightly related records. Separate documents can be easier when each language has its own publishing workflow, slug, or localized references. Choose a consistent locale naming scheme and define fallback behavior.

**Simplified field-level example:**

```ts
defineField({
  name: 'title',
  title: 'Title',
  type: 'object',
  fields: [
    { name: 'en', title: 'English', type: 'string' },
    { name: 'hi', title: 'Hindi', type: 'string' },
  ],
})
```

### 13. How do you change a schema after the project is live?

**Answer:** Changing a schema does not automatically transform existing content. If you rename a field, change its type, or restructure data, write a migration script to update existing documents. Test the migration on a development dataset first, verify the result, then run it against production with a recovery plan.

**Illustrative patch example:**

```ts
import { createClient } from '@sanity/client'

const client = createClient({
  projectId: process.env.SANITY_PROJECT_ID!,
  dataset: 'development',
  apiVersion: '2024-01-01',
  token: process.env.SANITY_WRITE_TOKEN,
  useCdn: false,
})

await client
  .patch('post-hello-world')
  .set({ subtitle: 'Updated subtitle' })
  .commit()
```

Do not put a write token in browser code. For bulk migrations, use a reviewed script and test it on a non-production dataset before running it on production.

---

## GROQ query language — Questions 14–20

### 14. What is GROQ?

**Answer:** GROQ stands for **Graph-Relational Object Queries**. It is Sanity’s query language. You use it to fetch, filter, sort, paginate, and shape content.

### 15. Write a basic GROQ query.

**Answer:** This query returns the title and slug of every document whose type is `post`.

```groq
*[_type == "post"] {
  title,
  slug
}
```

**What each part means:**

- `*` means search across documents.
- `[_type == "post"]` filters the results to post documents.
- `{ title, slug }` returns only those fields.

**A more frontend-friendly projection:**

```groq
*[_type == "post"] {
  _id,
  title,
  "slug": slug.current
}
```

Here, `"slug": slug.current` creates a simple `slug` property from the nested slug field.

### 16. How do you get data from a reference?

**Answer:** Use the `->` dereference operator. It follows a reference to the referenced document.

```groq
*[_type == "post"] {
  title,
  "authorName": author->name
}
```

This returns each post title and the referenced author’s name. The `author` field must be a reference to an author document.

**Fetch several author fields:**

```groq
*[_type == "post"] {
  title,
  author->{
    name,
    bio
  }
}
```

### 17. How do you sort and paginate results?

**Answer:** Use `order()` to sort and a range slice to limit the returned records.

```groq
*[_type == "post"]
  | order(publishedAt desc)[0...10] {
    title,
    publishedAt,
    "slug": slug.current
  }
```

This returns the latest ten posts that have a `publishedAt` value. In GROQ, `[0...10]` includes index 0 and excludes index 10, so it returns up to ten records.

**Next page:**

```groq
*[_type == "post"]
  | order(publishedAt desc)[10...20] {
    title,
    publishedAt
  }
```

For frequently changing or very large datasets, consider cursor-based pagination with a stable sort key rather than relying only on offsets.

### 18. What is a projection?

**Answer:** A projection is the `{ ... }` part of a GROQ query that decides which fields are returned. It can rename fields, follow references, and calculate values.

```groq
*[_type == "post"] {
  title,
  "authorName": author->name,
  "year": string(publishedAt)[0...4]
}
```

The aliases `"authorName"` and `"year"` become property names in the returned result. Make sure the source fields exist and that computed expressions match your data shape.

### 19. What is the difference between GROQ and GraphQL?

**Answer:** GROQ is designed for querying Sanity content and allows flexible filtering and reshaping inside a query. GraphQL uses a defined schema and query structure familiar to many teams, and requires the relevant GraphQL API/schema setup. Sanity supports both approaches; GROQ is a common default for Sanity projects.

### 20. How do you make GROQ queries faster?

**Answer:**

- Fetch only the fields the page needs.
- Filter early, commonly with `_type`.
- Paginate instead of fetching every document.
- Avoid unnecessary deep reference lookups.
- Use appropriate caching/CDN behavior for public published content.
- Inspect query patterns and measure performance rather than guessing.

**Less efficient:**

```groq
*[_type == "post"]
```

This returns full documents, potentially including large bodies and fields the page does not need.

**More focused:**

```groq
*[_type == "post" && defined(publishedAt)]
  | order(publishedAt desc)[0...10] {
    _id,
    title,
    "slug": slug.current,
    publishedAt
  }
```

---

## Portable Text and media — Questions 21–23

### 21. What is Portable Text?

**Answer:** Portable Text is Sanity’s structured rich-text format. Instead of storing the body only as an HTML string, it stores content as structured data. This can preserve paragraphs, formatting marks, links, and custom blocks so different frontends can render the same content.

**Schema example:**

```ts
defineField({
  name: 'body',
  title: 'Body',
  type: 'array',
  of: [{ type: 'block' }],
})
```

You can extend `of` with custom objects, such as an embedded image or callout block, when the content model requires them.

### 22. How do you display Portable Text in React or Next.js?

**Answer:** Install `@portabletext/react`, fetch the body from Sanity, and render it with the `PortableText` component.

```bash
npm install @portabletext/react
```

**Basic example:**

```tsx
import { PortableText } from '@portabletext/react'

export function PostBody({ value }: { value: any[] }) {
  return <PortableText value={value} />
}
```

**With a custom component:**

```tsx
import { PortableText } from '@portabletext/react'

const components = {
  marks: {
    link: ({ children, value }: any) => (
      <a href={value?.href} target="_blank" rel="noreferrer">
        {children}
      </a>
    ),
  },
}

export function PostBody({ value }: { value: any[] }) {
  return <PortableText value={value} components={components} />
}
```

In production, define proper TypeScript types for the Portable Text value and validate external link URLs as appropriate.

### 23. How does Sanity handle images?

**Answer:** Images are uploaded to Sanity’s asset storage and served through its CDN. The `@sanity/image-url` package can generate image URLs with specific widths, heights, crops, and formats. `hotspot: true` lets editors select the important part of an image so cropping can preserve it.

**Schema example:**

```ts
defineField({
  name: 'mainImage',
  title: 'Main image',
  type: 'image',
  options: { hotspot: true },
})
```

**Image URL builder:**

```ts
import imageUrlBuilder from '@sanity/image-url'
import { createClient } from 'next-sanity'

const client = createClient({
  projectId: process.env.NEXT_PUBLIC_SANITY_PROJECT_ID!,
  dataset: process.env.NEXT_PUBLIC_SANITY_DATASET!,
  apiVersion: '2024-01-01',
  useCdn: true,
})

const builder = imageUrlBuilder(client)

export const urlFor = (source: any) => builder.image(source)

// Example:
// urlFor(post.mainImage).width(900).height(600).fit('crop').auto('format').url()
```

Use appropriately sized images instead of downloading original-size assets for every page. If using `next/image`, configure the Sanity image host in the Next.js image settings.

---

## Studio customization — Questions 24–26

### 24. How can you customize Sanity Studio?

**Answer:** Because Studio is a React app, you can:

- Build custom input components for fields.
- Use the Structure tool to organize the sidebar.
- Add document actions and badges.
- Install or build plugins.
- Customize document previews and editor workflows.

**Example — custom Studio structure:**

```ts
// sanity.config.ts (simplified)
import { defineConfig } from 'sanity'
import { structureTool } from 'sanity/structure'
import { schemaTypes } from './schemaTypes'

export default defineConfig({
  name: 'default',
  title: 'My Content Studio',
  projectId: process.env.SANITY_STUDIO_PROJECT_ID!,
  dataset: process.env.SANITY_STUDIO_DATASET!,
  plugins: [structureTool()],
  schema: { types: schemaTypes },
})
```

The exact configuration can differ depending on the project setup and Studio version.

### 25. What are plugins? Name some examples.

**Answer:** Plugins add features to Studio. Examples include:

- **Vision:** lets developers test GROQ queries inside Studio.
- **Presentation tool:** supports visual editing and preview workflows when configured.
- **Custom input plugins:** for color pickers, code editors, or specialized fields.

Before adding a plugin, check that it is maintained, compatible with your Sanity version, and appropriate for the project’s security needs.

### 26. How do you set up how a document appears in lists?

**Answer:** Configure the schema’s `preview` property. The `select` object chooses which fields should be used as the title, subtitle, or media.

```ts
export const post = {
  name: 'post',
  title: 'Blog Post',
  type: 'document',
  fields: [
    { name: 'title', title: 'Title', type: 'string' },
    { name: 'mainImage', title: 'Main image', type: 'image' },
  ],
  preview: {
    select: {
      title: 'title',
      media: 'mainImage',
    },
  },
}
```

This helps editors identify documents quickly in Studio lists.

---

## Next.js integration — Questions 27–31

### 27. How do you connect Sanity to Next.js?

**Answer:** Install `next-sanity`, create a client with the project ID and dataset, and fetch content with GROQ. In the Next.js App Router, a Server Component can fetch content on the server and render it.

**Install:**

```bash
npm install next-sanity
```

**Create a client:**

```ts
// sanity/lib/client.ts
import { createClient } from 'next-sanity'

export const client = createClient({
  projectId: process.env.NEXT_PUBLIC_SANITY_PROJECT_ID!,
  dataset: process.env.NEXT_PUBLIC_SANITY_DATASET!,
  apiVersion: '2024-01-01',
  useCdn: true,
})
```

**Fetch and render posts in a Server Component:**

```tsx
// app/blog/page.tsx
import { client } from '@/sanity/lib/client'

const query = `*[_type == "post"] | order(publishedAt desc)[0...10] {
  _id,
  title,
  "slug": slug.current
}`

type Post = {
  _id: string
  title: string
  slug: string
}

export default async function BlogPage() {
  const posts: Post[] = await client.fetch(query)

  return (
    <main>
      <h1>Blog posts</h1>
      {posts.map(post => (
        <article key={post._id}>
          <h2>{post.title}</h2>
          <p>Slug: {post.slug}</p>
        </article>
      ))}
    </main>
  )
}
```

This example assumes the project has `post` documents with the selected fields. For a real site, link each post to a route and handle missing slugs.

### 28. What is the difference between `useCdn: true` and `useCdn: false`?

**Answer:**

- `useCdn: true` uses Sanity’s CDN for cached reads. It is usually a good choice for public, published content because it is fast.
- `useCdn: false` queries the API without the CDN cache and is useful when a workflow needs fresher data, such as certain preview or post-publish checks.

**Example:**

```ts
// Public published content
const publicClient = createClient({
  projectId: 'yourProjectId',
  dataset: 'production',
  apiVersion: '2024-01-01',
  useCdn: true,
})

// Fresh reads or server-side preview workflows
const freshClient = createClient({
  projectId: 'yourProjectId',
  dataset: 'production',
  apiVersion: '2024-01-01',
  useCdn: false,
})
```

`useCdn: false` alone does **not** grant draft access. Draft queries need the appropriate authenticated client configuration and permissions.

### 29. How does live preview work?

**Answer:** Next.js Draft Mode, together with Sanity’s Presentation tool or another preview interface, lets authorized editors view unpublished changes on the website before publishing. The preview frontend must use a server-side client configured to read draft content and must protect the preview route.

**High-level flow:**

1. An editor opens preview from Studio.
2. A protected route authenticates the request and enables Draft Mode.
3. The server fetches draft content using authorized credentials.
4. The preview page renders the draft.
5. The editor publishes or exits preview.

**Security reminders:** Never expose a privileged token in browser JavaScript. Do not cache private draft responses publicly. Use the preview tooling and client configuration appropriate to the Sanity and Next.js versions in the project.

### 30. What are SSG, ISR, and SSR when using Sanity?

**Answer:**

- **SSG (Static Site Generation):** Pages are built ahead of time. It is fast, but content can become stale until the site is rebuilt or revalidated.
- **ISR (Incremental Static Regeneration):** Static pages can be regenerated after a time interval or on demand.
- **SSR (Server-Side Rendering):** Content is fetched while handling each request. This can provide fresh data but may add request latency and server work.

**When to use them:**

| Strategy | Good fit |
|---|---|
| SSG | Stable marketing pages, documentation, infrequently changing posts |
| ISR | Blogs, product pages, and sites that need a balance of speed and freshness |
| SSR | Content that must be fetched per request or depends on request-specific information |

The best choice depends on content freshness, traffic, personalization, and deployment architecture.

### 31. How do you update the website when content is published?

**Answer:** Configure a Sanity webhook to call a protected Next.js route when relevant content is published. The route validates the webhook, identifies the affected page, and calls `revalidatePath()` or `revalidateTag()` to refresh cached content.

**Illustrative route handler:**

```ts
// app/api/revalidate/route.ts
import { revalidatePath } from 'next/cache'
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  const secret = request.headers.get('x-webhook-secret')

  if (!process.env.SANITY_WEBHOOK_SECRET ||
      secret !== process.env.SANITY_WEBHOOK_SECRET) {
    return NextResponse.json({ message: 'Unauthorized' }, { status: 401 })
  }

  const body = await request.json()

  if (body._type === 'post' && typeof body.slug === 'string') {
    revalidatePath(`/blog/${body.slug}`)
  }

  return NextResponse.json({ revalidated: true })
}
```

Configure the webhook to send the fields this handler expects. For production, use a robust signature or secret-validation strategy, validate the payload shape, and ensure the endpoint can safely handle retries.

---

## Security and deployment — Questions 32–36

### 32. How do API tokens work?

**Answer:** Tokens grant specific permissions to read or write content. Store privileged tokens in environment variables or a secrets manager. Never put write-capable tokens in frontend code or commit them to source control. Give each token only the permissions it needs.

**Environment example:**

```bash
SANITY_API_READ_TOKEN=your_read_token
SANITY_API_WRITE_TOKEN=your_write_token
```

Use server-side environment variables without a `NEXT_PUBLIC_` prefix for secrets. The `NEXT_PUBLIC_` prefix means a value may be exposed to the browser in Next.js.

### 33. How do roles work?

**Answer:** Project members receive roles such as Administrator, Editor, or Viewer, which determine what they can do. The exact built-in and custom role options depend on the Sanity plan and project configuration. Apply least privilege: give people only the access needed for their work.

**Example:** An editor may need to update and publish content, while a developer running a read-only content check may only need read access.

### 34. How do you handle login for your website’s users?

**Answer:** Sanity’s login is for managing Studio access; it is not automatically an authentication system for visitors to your website. For website users, integrate a separate authentication solution such as Auth.js (formerly NextAuth.js), Auth0, or Keycloak.

**Important distinction:**

- Sanity manages content and editor access.
- Your application’s authentication system manages visitor accounts and sessions.
- Your backend must enforce authorization for protected data and actions; hiding UI controls is not enough.

### 35. How do you deploy Studio?

**Answer:** You can deploy Studio to Sanity hosting or embed it inside a Next.js application at a route such as `/studio`.

**Common Sanity CLI workflow:**

```bash
# Run from the configured Sanity project
npx sanity deploy
```

Check the current CLI prompts and project configuration before deploying. For an embedded Studio, configure the Studio route and deploy it through the same build pipeline as the Next.js app. After deployment, verify login, dataset selection, and permissions.

### 36. How does version history work?

**Answer:** Sanity keeps document history so users can inspect previous changes and, depending on the workflow and permissions, restore an earlier version. History is useful for investigating accidental edits and understanding how content changed.

**Good practice:** Version history is not a substitute for backups or a tested migration/rollback strategy, especially for bulk content transformations.

---

## Scenario questions — Questions 37–41

### 37. Design the content model for a blog.

**Answer:** Create three main document types:

1. **Post:** title, slug, author reference, category references, main image, publication date, and Portable Text body.
2. **Author:** name, image, and biography.
3. **Category:** title and optionally a slug or description.

**Example post schema:**

```ts
import { defineType, defineField } from 'sanity'

export const post = defineType({
  name: 'post',
  title: 'Post',
  type: 'document',
  fields: [
    defineField({
      name: 'title',
      title: 'Title',
      type: 'string',
      validation: Rule => Rule.required().max(80),
    }),
    defineField({
      name: 'slug',
      title: 'Slug',
      type: 'slug',
      options: { source: 'title' },
      validation: Rule => Rule.required(),
    }),
    defineField({
      name: 'author',
      title: 'Author',
      type: 'reference',
      to: [{ type: 'author' }],
    }),
    defineField({
      name: 'categories',
      title: 'Categories',
      type: 'array',
      of: [{ type: 'reference', to: [{ type: 'category' }] }],
    }),
    defineField({
      name: 'mainImage',
      title: 'Main image',
      type: 'image',
      options: { hotspot: true },
    }),
    defineField({
      name: 'publishedAt',
      title: 'Published at',
      type: 'datetime',
    }),
    defineField({
      name: 'body',
      title: 'Body',
      type: 'array',
      of: [{ type: 'block' }],
    }),
  ],
})
```

**Example query:**

```groq
*[_type == "post" && slug.current == $slug][0] {
  title,
  publishedAt,
  "author": author->{name, bio},
  "categories": categories[]->{title},
  mainImage,
  body
}
```

This separates reusable author and category data from the post, avoiding repeated copies of the same author details.

### 38. Published changes aren’t showing on the site. How would you debug it?

**Answer:** Debug systematically rather than immediately changing code.

1. **Confirm publication:** Verify the document is published and not still only a draft.
2. **Check the query:** Run the GROQ query and confirm the document matches the filters and slug.
3. **Check the project and dataset:** Confirm the frontend points to the intended project and production dataset.
4. **Check caching:** Temporarily test with a client using `useCdn: false` to determine whether CDN caching is involved.
5. **Check ISR or revalidation:** Confirm the expected time-based or webhook-triggered revalidation ran successfully.
6. **Inspect the frontend:** Check server logs, API responses, and route generation to see whether the correct data reaches the page.

**Example diagnostic query:**

```groq
*[_type == "post" && slug.current == $slug] {
  _id,
  _type,
  title,
  "slug": slug.current,
  publishedAt
}
```

Run this with the same dataset and slug as the application. If the document is returned by the query but the website is stale, investigate caching and route revalidation.

### 39. How do you move content from WordPress to Sanity?

**Answer:** A typical migration process is:

1. Export WordPress content.
2. Map WordPress fields to Sanity schemas.
3. Transform HTML or rich text into Sanity’s Portable Text format.
4. Upload images and other media as Sanity assets.
5. Create/import documents and preserve author/category relationships.
6. Verify document counts, links, slugs, and rendered pages.
7. Run a final synchronization and switch the frontend over when ready.

**Illustrative migration checklist:**

```text
WordPress export
      ↓
Transform fields and HTML
      ↓
Upload media assets
      ↓
Create Sanity documents
      ↓
Verify counts, references, and URLs
      ↓
Test website → cut over
```

Sanity’s dataset import command expects Sanity-compatible NDJSON, so raw WordPress export data usually needs transformation first. Test with a small batch before migrating all content.

### 40. How do you set up separate environments?

**Answer:** Use separate datasets, such as `development`, `staging`, and `production`, and select the correct dataset through environment variables. Test schema and content migrations on development before applying them to production.

**Example environment configuration:**

```bash
# .env.development.local
NEXT_PUBLIC_SANITY_PROJECT_ID=yourProjectId
NEXT_PUBLIC_SANITY_DATASET=development
```

```bash
# Production deployment environment
NEXT_PUBLIC_SANITY_PROJECT_ID=yourProjectId
NEXT_PUBLIC_SANITY_DATASET=production
```

Keep environment-specific tokens separate. Avoid running destructive scripts against production by accident; make the target dataset explicit and verify it before a migration.

### 41. How would you test Sanity content in a CI/CD pipeline?

**Answer:** Test the schema, content queries, and pages against a staging or test dataset. Useful checks include:

- Run type checks and linting.
- Execute GROQ queries to check required content exists.
- Validate required fields and references where practical.
- Run end-to-end tests, for example with Playwright, against staging.
- Confirm pages render correctly after content changes.
- Keep tokens secure and avoid destructive tests against production.

**Example Playwright smoke test:**

```ts
import { test, expect } from '@playwright/test'

test('blog page displays its heading', async ({ page }) => {
  await page.goto('/blog')
  await expect(
    page.getByRole('heading', { name: 'Blog posts' })
  ).toBeVisible()
})
```

This test checks the page’s visible behavior. Add project-specific tests for a known published post, slug routes, missing content, and preview workflows as appropriate.

---

## Quick revision checklist

The screenshot recommends prioritizing **questions 1–8, 14–17, 21, and 27–31** for a quick beginner-level revision.

- [ ] Explain headless CMS, Studio, and Content Lake.
- [ ] Understand schemas, common field types, references, objects, and validation.
- [ ] Write a basic GROQ query, follow a reference with `->`, and sort/paginate results.
- [ ] Explain Portable Text and how to render it in React.
- [ ] Connect Sanity to Next.js using a client and GROQ.
- [ ] Compare `useCdn: true` with `useCdn: false`.
- [ ] Explain SSG, ISR, SSR, preview, and webhook-triggered revalidation.
- [ ] Explain token security, environment separation, and how to debug stale content.

## Interview tip

When answering a technical question, try this structure: **define the concept → explain why it matters → give a short example → mention one trade-off or security consideration.** This makes answers clearer and shows practical understanding.


---

## Additional Intermediate Questions — Questions 42–55

> These are supplemental interview questions added to extend the 41 questions visible in your screenshots. They are not presented as verbatim questions from the video.

### 42. How do you fetch a single post by its slug in GROQ?

**Answer:** Pass the slug as a query parameter and select the first matching document. Parameters are safer and easier to reuse than building query strings by concatenating user input.

```groq
*[_type == "post" && slug.current == $slug][0] {
  _id,
  title,
  "slug": slug.current,
  body,
  publishedAt
}
```

```ts
const post = await client.fetch(query, { slug: 'hello-world' })
```

### 43. How do you filter posts by category?

**Answer:** If categories are stored as references, compare the category document ID with the references stored in the post.

```groq
*[_type == "post" && $categoryId in categories[]._ref] {
  title,
  "slug": slug.current
}
```

Pass `categoryId` as a parameter. Confirm that the schema field is an array of references named `categories`; adjust the query if your schema differs.

### 44. How do you fetch a post and its author in one query?

**Answer:** Use the dereference operator `->` in the projection to fetch fields from the referenced author.

```groq
*[_type == "post" && slug.current == $slug][0] {
  title,
  body,
  "author": author->{
    name,
    bio,
    image
  }
}
```

This avoids making separate client requests for the post and author when one GROQ query can return the required shape.

### 45. How do you generate a unique slug?

**Answer:** Use Sanity’s `slug` field type and set `options.source` to the title. If uniqueness is required, add a uniqueness validation function that checks existing documents while accounting for the document being edited. Slug uniqueness should be scoped to the relevant content type or route structure.

```ts
defineField({
  name: 'slug',
  title: 'Slug',
  type: 'slug',
  options: {
    source: 'title',
    isUnique: async (value, context) => {
      const { document, getClient } = context
      const client = getClient({ apiVersion: '2024-01-01' })
      const id = document?._id.replace(/^drafts\./, '')
      const params = { draftId: `drafts.${id}`, publishedId: id, value }

      const query = `!defined(*[
        _type == "post" &&
        slug.current == $value &&
        !(_id in [$draftId, $publishedId])
      ][0]._id)`

      return client.fetch(query, { ...params, value })
    },
  },
})
```

Check the current Sanity schema API and type signatures for your installed version before using a custom `isUnique` implementation.

### 46. How do you use environment variables in Next.js safely?

**Answer:** Public configuration such as a project ID or dataset may use a `NEXT_PUBLIC_` prefix. Privileged tokens must not use that prefix because values with it can be bundled into client-side code.

```bash
NEXT_PUBLIC_SANITY_PROJECT_ID=yourProjectId
NEXT_PUBLIC_SANITY_DATASET=production
SANITY_API_READ_TOKEN=server_only_token
```

Read `SANITY_API_READ_TOKEN` only in server-side code. Do not commit real secrets to Git.

### 47. How do you implement a protected preview route?

**Answer:** A preview route should authenticate the request before enabling Draft Mode, and draft content should be fetched only on the server with the necessary permissions. Do not let an arbitrary public request enable preview or expose unpublished content.

**Conceptual flow:**

```text
Studio preview action
  → authenticated Next.js route
  → validate access / secret
  → enable Draft Mode
  → fetch draft content on server
  → render preview
```

The exact route APIs depend on the Next.js version. Use Sanity’s current Presentation/Visual Editing integration guidance for a production implementation.

### 48. How do you revalidate a specific post after publication?

**Answer:** Configure a Sanity webhook to call a protected endpoint and pass the document type and slug. After validating the request, revalidate the affected route.

```ts
import { revalidatePath } from 'next/cache'

revalidatePath('/blog/hello-world')
```

For dynamic routes, derive the path from validated webhook data rather than hardcoding it. Validate the webhook signature or secret and avoid accepting arbitrary paths from untrusted requests.

### 49. How do you render a Sanity image using `next/image`?

**Answer:** Generate an appropriately sized image URL with Sanity’s image URL builder, then pass it to Next.js Image. Configure the Sanity image host in `next.config` according to the installed Next.js version.

```tsx
import Image from 'next/image'
import { urlFor } from '@/sanity/lib/image'

export function PostImage({ image, title }: {
  image: unknown
  title: string
}) {
  const src = urlFor(image).width(1200).auto('format').url()

  return (
    <Image
      src={src}
      alt={title}
      width={1200}
      height={800}
      sizes="(max-width: 768px) 100vw, 800px"
    />
  )
}
```

Use dimensions and `sizes` appropriate for the actual layout. Configure remote image access as required by your Next.js version.

### 50. How do you type data returned from GROQ?

**Answer:** Define a TypeScript type matching the query projection, or use Sanity type-generation tools. The type should reflect the fields the query actually returns, including nullable or optional values.

```ts
type PostCard = {
  _id: string
  title: string
  slug: string | null
  publishedAt?: string
}

const posts: PostCard[] = await client.fetch(query)
```

Types improve development feedback but do not validate remote data at runtime. Handle missing fields in the UI as well.

### 51. How do you handle a missing post in Next.js?

**Answer:** If a slug query returns no document, show a 404 rather than trying to render missing fields.

```tsx
import { notFound } from 'next/navigation'
import { client } from '@/sanity/lib/client'

export default async function PostPage({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  const post = await client.fetch(
    `*[_type == "post" && slug.current == $slug][0]`,
    { slug }
  )

  if (!post) notFound()

  return <article><h1>{post.title}</h1></article>
}
```

This example uses the promise-based `params` shape used by current Next.js versions; adapt it for the version in your project.

### 52. How do you model reusable site settings?

**Answer:** Create a settings document for shared values such as the site title, logo, navigation, and social links. Customize Studio structure to present a predictable settings entry and prevent duplicate singleton documents where necessary.

```groq
*[_type == "siteSettings"][0] {
  title,
  logo,
  navigation[] {
    label,
    url
  }
}
```

The exact query depends on whether navigation items contain URLs, page references, or nested children.

### 53. How do you support content for multiple brands or websites?

**Answer:** If brands share schemas and editorial permissions, add a `site` or `brand` reference/identifier and filter queries by it. If the sites require strong content isolation, different permissions, or separate release processes, consider separate datasets or projects.

```groq
*[_type == "post" && site._ref == $siteId] {
  title,
  "slug": slug.current
}
```

Only use this form if `site` is a reference field. Match the filter to the actual schema.

### 54. How do you migrate content safely?

**Answer:** Map the old schema to the new one, write a migration script, test on a development dataset, and verify document counts, references, images, and slugs. Make the migration repeatable where practical and keep a recovery plan.

**Checklist:**

1. Back up/export the source data.
2. Test a small batch.
3. Transform fields and upload assets.
4. Verify relationships and rendered pages.
5. Run the production migration during a controlled window.
6. Monitor errors and keep a rollback/repair plan.

### 55. How do you troubleshoot a slow Sanity-powered page?

**Answer:** Measure before optimizing. Inspect the query response size, number of requests, rendering strategy, image sizes, and cache behavior. Reduce unnecessary fields and deep dereferencing, paginate lists, optimize images, and use static rendering or ISR for content that does not need per-request freshness.

---

## Additional Advanced and Scenario-Based Questions — Questions 56–65

### 56. How would you design a content model for a blog with related posts?

**Answer:** Use separate `post`, `author`, and `category` document types. Each post references an author and can contain an array of category references. Related posts can be manually selected with references or computed from shared categories.

```groq
*[_type == "post" && slug.current == $slug][0] {
  title,
  "author": author->{name, bio},
  "categories": categories[]->{title},
  "relatedPosts": relatedPosts[]->{title, "slug": slug.current}
}
```

Add `relatedPosts` as an array of references in the schema if editors should choose them manually.

### 57. What would you do if a published post is returned by GROQ but not shown on the website?

**Answer:** The content query is likely working, so investigate the frontend path. Check whether the route uses static generation, ISR, or a cache; verify the webhook ran; inspect the rendered response and deployment logs; and confirm the page is using the same slug and dataset as the test query.

### 58. How would you design a content migration that can be safely rerun?

**Answer:** Make the migration idempotent where possible: use stable document IDs, detect existing records, and update or skip records deliberately instead of blindly creating duplicates. Log failures, process manageable batches, and verify the output after each run.

### 59. How would you build global navigation in Sanity?

**Answer:** Store navigation in a settings document as an array of objects. Each item might contain a label, URL or page reference, and optional child items. Validate labels and destinations and define how references are resolved in GROQ.

```groq
*[_type == "siteSettings"][0] {
  navigation[] {
    label,
    url,
    "pageSlug": page->slug.current
  }
}
```

This assumes each navigation item has a `page` reference and a `url` field; adjust the projection to match your schema.

### 60. How would you support preview without exposing drafts publicly?

**Answer:** Require authentication to enable preview, use a server-side client with draft-reading permissions, and avoid putting privileged tokens in browser bundles. Do not publicly cache responses containing drafts. Ensure the preview session can be disabled and test the route as an unauthenticated visitor.

### 61. How would you design a high-traffic website powered by Sanity?

**Answer:** Use CDN-backed reads for public published content, static generation or ISR for cacheable pages, focused GROQ projections, and optimized responsive images. Revalidate only affected pages or tags and keep preview/authenticated content separate from public cacheable responses.

### 62. How can you show live content updates?

**Answer:** Use Sanity’s supported live content tooling or client subscriptions for the chosen framework. Subscribe only to the content needed by the current view, handle updates safely, and clean up subscriptions. For ordinary public pages, static rendering plus revalidation may be simpler and more cost-effective than live subscriptions.

### 63. How do you keep schema and frontend changes compatible during deployment?

**Answer:** Prefer backward-compatible changes such as adding optional fields. Deploy frontend code that can tolerate old and new content shapes, migrate documents in a controlled sequence, test in staging, and keep a rollback plan. Avoid changing a field’s meaning or type without planning the data migration.

### 64. How do you decide between Sanity and another CMS?

**Answer:** Compare the complexity of the content model, editor workflow, collaboration, localization, permissions, integrations, frontend flexibility, cost, and maintenance burden. Sanity is a strong option when structured content and a customizable editing experience are important. A simpler CMS may be preferable if the project needs only basic publishing and the team wants less setup.

### 65. What are the most important Sanity concepts to explain in an interview?

**Answer:** Be prepared to explain:

- Headless CMS, Studio, Content Lake, projects, and datasets.
- Schemas, field types, references, validation, and migrations.
- GROQ filters, projections, dereferencing, sorting, and pagination.
- Portable Text and image URL generation.
- Next.js data fetching, SSG/ISR/SSR, Draft Mode, and revalidation.
- API token security, roles, deployment, and environment separation.
- How to debug stale content and design a maintainable content model.

A strong answer defines the concept, gives a short code example, explains when to use it, and mentions one trade-off or security consideration.

---

## Notes on completeness

- **Questions 1–41** are based on the questions visible in the screenshots you supplied, with explanations and examples expanded.
- **Questions 42–65** are additional interview-preparation questions added to fill out the guide. They are supplemental questions, not verified as questions spoken in the original video.
- This file is one consolidated Markdown document. If the video contains further questions that are not shown in the screenshots or readable from the recording, those would need to be transcribed separately to claim a word-for-word complete video transcript.


## Advanced Add-on: Production-Focused Questions — Questions 66–80

> These 15 questions extend the guide with production-oriented interview topics. They supplement the earlier questions and are not claimed to be verbatim questions from the original video.

### 66. How does Sanity Live Content work with Next.js?

**Answer:** Live Content lets a frontend respond to content changes without relying only on periodic revalidation. Depending on the configured integration, Sanity can provide live query updates and support visual editing workflows.

**Illustrative example** (the `sanityFetch` helper comes from a configured Sanity live-content setup; it is not available automatically):

```tsx
import { defineQuery } from 'next-sanity'
import { sanityFetch } from '@/sanity/lib/live'

const POSTS_QUERY = defineQuery(`
  *[_type == "post"] | order(publishedAt desc) {
    _id,
    title,
    "slug": slug.current
  }
`)

export default async function PostsPage() {
  const posts = await sanityFetch({ query: POSTS_QUERY })

  return posts.data.map(post => (
    <article key={post._id}>{post.title}</article>
  ))
}
```

### 67. How would you implement secure Draft Mode?

**Answer:** Authenticate the preview request, enable Draft Mode only after authorization, and use a server-side client with draft-reading permissions. Never expose a privileged token in browser code.

```ts
// Conceptual example: app/api/draft/route.ts
import { draftMode } from 'next/headers'
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const secret = request.nextUrl.searchParams.get('secret')

  if (
    !process.env.PREVIEW_SECRET ||
    secret !== process.env.PREVIEW_SECRET
  ) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const draft = await draftMode()
  draft.enable()

  return NextResponse.redirect(new URL('/preview', request.url))
}
```

For production, validate the preview request and document access, and follow the current Next.js and Sanity preview integration guidance.

### 68. How do you implement webhook-based cache invalidation?

**Answer:** A Sanity webhook sends a request after a relevant publish event. A protected Next.js endpoint validates the request and invalidates only affected paths or tags.

```ts
import { revalidatePath } from 'next/cache'
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  const secret = request.headers.get('x-webhook-secret')

  if (!process.env.SANITY_WEBHOOK_SECRET ||
      secret !== process.env.SANITY_WEBHOOK_SECRET) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const body = await request.json()

  if (body._type === 'post' && typeof body.slug === 'string') {
    revalidatePath(`/blog/${body.slug}`)
  }

  return NextResponse.json({ revalidated: true })
}
```

Configure the webhook to send the fields this handler expects. In production, validate the signature or secret robustly and validate the payload shape.

### 69. How would you design a large content model with reusable references?

**Answer:** Separate independently managed entities into document types, then reference them from parent documents. For example, a post references an author and contains category references, while a settings document stores global configuration.

```groq
*[_type == "post" && slug.current == $slug][0] {
  title,
  body,
  "author": author->{name, bio},
  "categories": categories[]->{title}
}
```

This reduces duplicated data and allows author or category updates to be reused across pages.

### 70. How do you prevent duplicate slugs?

**Answer:** Configure slug generation from the title and add a uniqueness check. The check must exclude the current document during editing and account for draft and published IDs. The exact callback types and implementation should be checked against the installed Sanity version.

```ts
defineField({
  name: 'slug',
  type: 'slug',
  options: {
    source: 'title',
    isUnique: async (value, context) => {
      const { document, getClient } = context
      const client = getClient({ apiVersion: '2024-01-01' })
      const id = document?._id.replace(/^drafts\./, '')

      const query = `!defined(*[
        _type == "post" &&
        slug.current == $slug &&
        !(_id in [$id, $draftId])
      ][0]._id)`

      return client.fetch(query, {
        slug: value,
        id,
        draftId: `drafts.${id}`,
      })
    },
  },
})
```

### 71. How would you migrate a schema without breaking a live website?

**Answer:** Use a backward-compatible deployment strategy:

1. Add the new field without immediately removing the old one.
2. Deploy frontend code that can read both structures.
3. Migrate existing documents in batches.
4. Validate migrated data.
5. Switch the frontend to the new field.
6. Remove the old field only after all consumers are updated.

This reduces the chance of the frontend breaking while old and new content coexist.

### 72. How do you optimize GROQ queries for performance?

**Answer:** Filter early, project only required fields, paginate large results, avoid unnecessary nested references, and measure performance.

```groq
*[_type == "post" && defined(publishedAt)]
  | order(publishedAt desc)[0...20] {
    _id,
    title,
    "slug": slug.current,
    publishedAt,
    "authorName": author->name
  }
```

Avoid returning entire documents when a page needs only a few fields.

### 73. How do you handle localization at scale?

**Answer:** Choose either field-level translations or separate documents linked by a translation identifier. Define locale-specific slugs, publishing rules, and fallback behavior. Filter queries explicitly by locale.

```groq
*[
  _type == "post" &&
  language == $locale &&
  slug.current == $slug
][0] {
  title,
  body,
  language
}
```

This assumes each post has a `language` field.

### 74. How do you implement role-based content access?

**Answer:** Use Sanity roles and dataset permissions for editorial access. For website users, enforce authorization in the application backend. Never assume hiding content in React makes it secure. Sensitive content should be queried server-side with the correct credentials and permissions.

### 75. How would you support multiple brands in one Sanity project?

**Answer:** If brands share editorial workflows, model a brand or site identifier and scope queries to it.

```groq
*[
  _type == "product" &&
  brand._ref == $brandId
] {
  title,
  description
}
```

Use this form only if `brand` is a reference field. If brands require strict isolation or different permissions, separate datasets or projects may be more appropriate.

### 76. How do you handle large content migrations?

**Answer:** Build a repeatable migration pipeline that maps source fields, transforms rich text, uploads assets, and preserves references. Test on a development dataset, use stable document IDs, process batches, log errors, and verify counts and relationships before production cutover.

**Checklist:**

1. Back up or export the source data.
2. Test a small batch.
3. Transform fields and upload assets.
4. Verify relationships and rendered pages.
5. Run the production migration in a controlled window.
6. Monitor errors and retain a recovery plan.

### 77. How do you debug stale content when GROQ returns the latest data?

**Answer:** If the query returns new content but the page does not show it, inspect the rendering and caching layers: Sanity CDN caching, Next.js static rendering or ISR, route/tag revalidation, browser caching, deployment environment variables, and whether the page uses the same dataset and slug. Test the API response and rendered page separately.

### 78. How would you implement visual editing?

**Answer:** Configure Sanity’s Presentation tool and the appropriate Next.js preview/live-content integration. Editors should be able to open the frontend, select editable content, preview drafts, and navigate between documents. Protect draft access and configure the preview URL, content source maps, and live-editing setup as required by the chosen integration.

### 79. How do you make Sanity queries type-safe with TypeScript?

**Answer:** Define types based on the returned projection or use Sanity’s supported type-generation tooling. Reflect the actual fields returned by GROQ, including nullable values.

```ts
type PostSummary = {
  _id: string
  title: string
  slug: string | null
}

const query = `*[_type == "post"] {
  _id,
  title,
  "slug": slug.current
}`

const posts = await client.fetch<PostSummary[]>(query)
```

Type annotations improve developer feedback but do not validate remote data at runtime.

### 80. When would you choose Sanity over a traditional CMS?

**Answer:** Choose Sanity when the application needs structured reusable content, a customized editorial experience, multiple frontend channels, or flexible content modeling. A traditional CMS may be simpler when the project mainly needs standard website publishing with minimal custom development. Consider editor requirements, developer effort, integrations, access controls, and long-term maintenance.
