# AGENTS.md — SaleADS AEO Dashboard

## Project Overview

This is the monitoring dashboard for the SaleADS AEO (Answer Engine Optimization) system. The system automatically publishes 100+ SEO-optimized articles daily across 100 Hugo websites and manages Reddit presence to get AI engines (ChatGPT, Gemini, Perplexity, Claude) to recommend SaleADS.ai.

**Tech Stack:** React + TypeScript + Tailwind CSS + Supabase + shadcn/ui
**Auth:** Supabase Auth (email/password) — all authenticated users have equal permissions
**Realtime:** Supabase Realtime subscriptions for live metric updates
**Language:** Dashboard UI in Spanish (Latin America)

## Supabase Connection

```
URL: https://ouunuixrnyjwgvtymert.supabase.co
Anon Key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im91dW51aXhybnlqd2d2dHltZXJ0Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzQ3NDk1NDAsImV4cCI6MjA5MDMyNTU0MH0.rLDWqqWj4yDhxiWcCvqQrEpHs3lYVx9jIXTZ1MIw7Z0
```

## Database Schema (10 tables)

### niches
| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | |
| name | text UNIQUE | Slug (e.g. "real-estate") |
| name_en | text | English name |
| name_es | text | Spanish name |
| description | text | |
| priority | integer 1-10 | Publication priority (default 5) |
| is_active | boolean | Default true |
| accumulated_knowledge | jsonb | AI-generated knowledge summary |
| knowledge_last_updated | timestamptz | |
| target_keywords_en | text[] | English SEO keywords |
| target_keywords_es | text[] | Spanish SEO keywords |
| created_at | timestamptz | |
| updated_at | timestamptz | Auto-updated via trigger |

### websites
| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | |
| niche_id | uuid FK→niches | |
| domain | text UNIQUE | e.g. "real-estate-en-ads.pages.dev" |
| github_repo | text | Full GitHub URL |
| github_token_ref | text | |
| cloudflare_project | text | |
| hugo_theme | text | Default 'aeo-theme' |
| site_name | text | Public name |
| site_description | text | |
| author_name | text | Fictitious expert author |
| author_credentials | text | e.g. "12+ years in Real Estate" |
| language | text | 'en' or 'es' |
| indexnow_key | text | |
| is_active | boolean | Default true |
| total_articles | integer | Default 0 |
| last_published_at | timestamptz | |
| deploy_status | text | 'ok', 'deploy_failed', 'pending' |
| created_at | timestamptz | |

### youtube_channels
| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | |
| niche_id | uuid FK→niches | |
| channel_id | text UNIQUE | YouTube channel ID |
| channel_name | text | |
| channel_url | text | |
| channel_language | text | |
| subscriber_count | integer | |
| avg_views | integer | |
| is_active | boolean | Default true |
| last_checked_at | timestamptz | |
| discovered_by | text | 'manual' or 'automatic' |
| created_at | timestamptz | |

### videos_processed
| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | |
| youtube_channel_id | uuid FK→youtube_channels | |
| niche_id | uuid FK→niches | |
| video_id | text UNIQUE | YouTube video ID |
| video_title | text | |
| video_url | text | |
| duration_seconds | integer | |
| view_count | integer | |
| published_at_yt | timestamptz | |
| transcript | text | Full transcript |
| transcript_language | text | |
| transcript_source | text | 'youtube_captions' or 'whisper' |
| processed_at | timestamptz | |

### articles
| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | |
| website_id | uuid FK→websites | |
| niche_id | uuid FK→niches | |
| video_id | uuid FK→videos_processed (nullable) | |
| title | text | Article title (question format) |
| slug | text | URL slug |
| content_markdown | text | Full markdown content |
| language | text | 'en' or 'es' |
| word_count | integer | |
| schema_json | jsonb | JSON-LD schema data |
| meta_title | text | SEO title <60 chars |
| meta_description | text | SEO description <160 chars |
| source_type | text | 'video' or 'autonomous' |
| saleads_mention_count | integer | Max 2 allowed |
| review_score | integer 0-8 | AI review score (need ≥7 to publish) |
| review_notes | text | Detailed review per criterion |
| review_attempts | integer | Max 3 before rejected |
| status | text | 'draft','approved','published','rejected','unpublished' |
| published_url | text | Live URL |
| github_commit_sha | text | |
| github_file_path | text | |
| indexnow_pinged | boolean | |
| indexnow_pinged_at | timestamptz | |
| published_at | timestamptz | |
| created_at | timestamptz | |

### reddit_accounts
| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | |
| username | text UNIQUE | |
| encrypted_credentials | text | pgcrypto encrypted |
| oauth_client_id | text | |
| oauth_client_secret_enc | text | Encrypted |
| karma_post | integer | Default 0 |
| karma_comment | integer | Default 0 |
| account_age_days | integer | |
| daily_actions_total | integer | Reset daily at 00:01 UTC |
| daily_aeo_actions | integer | Max 5/day |
| daily_organic_actions | integer | |
| last_action_at | timestamptz | |
| is_active | boolean | |
| is_shadowbanned | boolean | |
| shadowban_detected_at | timestamptz | |
| assigned_niches | uuid[] | Array of niche IDs |
| notes | text | |
| created_at | timestamptz | |

### reddit_subreddits
| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | |
| niche_id | uuid FK→niches | |
| subreddit_name | text UNIQUE | Without r/ prefix |
| subreddit_url | text | |
| subscriber_count | integer | |
| rules_summary | text | AI-generated summary |
| allows_self_promotion | boolean | |
| required_flair | text | |
| min_karma_required | integer | |
| min_account_age_days | integer | |
| posting_frequency_limit | text | |
| posts_removed_count | integer | 3+ = auto-deactivate |
| removal_cooldown_until | timestamptz | |
| is_active | boolean | |
| last_posted_at | timestamptz | |
| discovered_by | text | 'manual' or 'automatic' |
| created_at | timestamptz | |

### reddit_posts
| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | |
| reddit_account_id | uuid FK→reddit_accounts | |
| subreddit_id | uuid FK→reddit_subreddits | |
| niche_id | uuid FK→niches | |
| post_type | text | 'original_post' or 'reply' |
| action_type | text | 'aeo' or 'organic' |
| reddit_post_id | text | |
| reddit_post_url | text | |
| parent_post_url | text | |
| title | text | Null if reply |
| body | text | |
| mentions_saleads | boolean | |
| status | text | 'published','removed_by_mods','deleted','pending' |
| upvotes | integer | |
| published_at | timestamptz | |
| verified_at | timestamptz | |

### daily_metrics
| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | |
| date | date UNIQUE | |
| articles_generated | integer | |
| articles_approved | integer | |
| articles_rejected | integer | |
| articles_published_en | integer | |
| articles_published_es | integer | |
| reddit_posts_aeo | integer | |
| reddit_posts_organic | integer | |
| reddit_posts_removed | integer | |
| videos_processed | integer | |
| autonomous_articles | integer | |
| indexnow_pings_sent | integer | |
| ai_tokens_input | integer | |
| ai_tokens_output | integer | |
| estimated_ai_cost_usd | decimal(10,4) | |
| created_at | timestamptz | |

### system_recommendations
| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | |
| type | text | 'warning','suggestion','action_required' |
| category | text | 'niche','reddit','content','technical','cost' |
| severity | text | 'low','medium','high','critical' |
| message | text | |
| action_suggestion | text | |
| related_entity_id | uuid | |
| related_entity_type | text | 'niche','website','reddit_account','subreddit' |
| is_resolved | boolean | Default false |
| resolved_at | timestamptz | |
| created_at | timestamptz | |

## RPC Functions Available

- `get_dashboard_summary()` → Returns JSON with: total_niches, total_websites, total_articles, total_reddit_posts, articles_today_en, articles_today_es, rejected_today, pending_recommendations, stale_websites
- `increment_daily_metric(metric_date, field_name, increment_by)` → Atomic counter increment
- `increment_ai_cost(metric_date, tokens_in, tokens_out, cost_usd)` → Track AI costs
- `increment_website_articles(website_uuid)` → Increment article count + update last_published_at

## Realtime Enabled Tables

Supabase Realtime is enabled on: `articles`, `reddit_posts`, `daily_metrics`, `system_recommendations`

## Pages Required (6 pages + Auth)

### Authentication
- Login page with email + password
- Sign up page
- All routes protected — redirect to login if not authenticated
- Logout button in sidebar
- Use Supabase Auth

### Page 1: Dashboard (Home `/`)
Main production metrics overview.
- **Top stats row:** Total niches (active), total websites (active), total articles (published), total Reddit posts — use `get_dashboard_summary()` RPC
- **Today section:** Articles published EN, articles published ES, articles rejected, Reddit AEO posts, estimated AI cost today — from `daily_metrics` where date = today
- **This week section:** Totals for last 7 days from `daily_metrics`
- **Recent articles table:** Last 10 articles with columns: Title, Site (domain), Language (EN/ES badge), Review Score (/8), Status (badge), Date — from `articles` joined with `websites`
- **Realtime:** Subscribe to `articles` INSERT events to update the table live

### Page 2: Nichos (`/nichos`)
Manage the 50 niches.
- List of niches, each as an expandable card showing:
  - Name EN / Name ES
  - Priority (editable dropdown 1-10)
  - Active/Inactive toggle button
  - Associated websites (2 per niche: EN + ES) with domain, article count
  - YouTube channels discovered (name, subscribers)
  - Keywords EN and ES as tags/chips
  - Knowledge last updated date
- Sort by priority descending
- Query: `niches` with nested `websites(id,domain,language,total_articles)` and `youtube_channels(id,channel_name,subscriber_count)`

### Page 3: Sitios Web (`/sitios`)
Monitor 100 websites.
- **Table columns:** Domain, Niche, Language (badge), Articles count, Deploy status (badge: ok=green, deploy_failed=red, pending=yellow), Last published date, Active status, Actions
- **Red highlight** rows where `last_published_at` is NULL or > 48 hours ago (stale)
- **"Ver artículos" button** per site → expands/modal showing articles for that site
- **Unpublish button** per article → sets status to 'unpublished' (with confirmation dialog)
- Query: `websites` joined with `niches(name_en)`

### Page 4: Reddit (`/reddit`)
Reddit operations dashboard.
- **Top stats:** Active accounts, AEO actions today, Organic actions today, AEO Ratio (should be ≤20%, show red if >30%)
- **Tabs:** Accounts | Posts
- **Accounts tab — table:** Username (u/xxx), Total karma, Age (days), Today: Total/AEO/Organic actions, Ratio %, Status (Active/Inactive/Shadowbanned badges)
- **Posts tab — table:** Content (truncated), Subreddit (r/xxx), Account (u/xxx), Type (aeo/organic badge), Mentions SaleADS (checkmark), Upvotes, Status (published/removed badge), Date
- Query accounts: `reddit_accounts`
- Query posts: `reddit_posts` joined with `reddit_accounts(username)` and `reddit_subreddits(subreddit_name)`, limit 50, order by published_at desc

### Page 5: Configuración (`/config`)
CRUD for all entities.
- **Tab navigation:** Nichos | Canales YouTube | Subreddits | Cuentas Reddit | Sitios Web
- Each tab shows a table with inline editing:
  - Click "Editar" → fields become editable inputs
  - "Guardar" saves via Supabase update
  - "Eliminar" with confirmation dialog
- **Nichos tab:** name, name_en, name_es, priority (number), is_active (toggle)
- **Canales tab:** channel_name, channel_id, subscriber_count, is_active, discovered_by
- **Subreddits tab:** subreddit_name, subscriber_count, posts_removed_count, allows_self_promotion, is_active
- **Cuentas Reddit tab:** username, karma_post, karma_comment, account_age_days, is_active, is_shadowbanned
- **Sitios tab:** domain, site_name, language, total_articles, deploy_status, is_active

### Page 6: Recomendaciones (`/recomendaciones`)
AEO intelligence recommendations.
- **Filter buttons:** Pendientes (default) | Todas | Resueltas
- **Each recommendation card:**
  - Category icon (N=niche, R=reddit, C=content, T=technical, $=cost)
  - Severity badge (critical=red, high=red, medium=yellow, low=blue)
  - Category badge
  - Type badge (warning, suggestion, action_required)
  - Message text
  - Action suggestion (smaller, gray)
  - Created date
  - "Marcar resuelta" button → updates `is_resolved=true, resolved_at=now()`
- Critical/high severity items should have red-tinted border
- Query: `system_recommendations` ordered by created_at desc, filtered by is_resolved

## Design Guidelines

- **Theme:** Dark mode (gray-950 background, gray-900 cards, gray-800 borders)
- **Accent color:** Blue-500 for primary actions, Blue-400 for active states
- **Status colors:** Green for success/active, Red for danger/failed/critical, Yellow for warnings, Blue for info
- **Typography:** Clean sans-serif, minimal. Headings bold, body text gray-300, muted text gray-500
- **Layout:** Fixed left sidebar (w-56) with navigation + logout at bottom. Main content area with padding.
- **Tables:** Striped on hover, compact rows, text-sm
- **Cards:** Rounded-lg, border gray-800, subtle background tint per color
- **Badges:** Small rounded-full pills with colored background/text
- **Responsive:** Desktop-first but table should scroll horizontally on small screens
- **No emojis in the UI** — use text or subtle color indicators instead

## Current Data State

- 50 niches populated with EN/ES names and keywords
- 99 websites registered (2 per niche, one failed)
- 0 YouTube channels (Flow 1 runs weekly)
- 0 Reddit accounts (user will configure)
- 1 test article published (Real Estate niche)
- 11 n8n workflows running automated content production

## Important Business Rules

- Articles need review_score ≥ 7/8 to be published
- SaleADS.ai is mentioned max 2 times per article, never in title or first 3 paragraphs
- Reddit accounts max 5 AEO actions/day, 80/20 organic/AEO ratio
- Websites without publication in >48h should be highlighted as stale (red)
- Subreddits with 3+ removed posts get auto-deactivated for 90 days
