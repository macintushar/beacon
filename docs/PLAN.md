## Beacon — MVP Plan

### Product scope

- Core users: teams, communities, organizations needing async chat.
- Platforms roadmap: Web (v1), Mobile (v1.5), Desktop via Tauri (v2).
- Features (v1): workspaces, public/private channels, DMs, single-level threads, mentions (@user, #channel) with typeahead, reactions (Unicode), basic channel-scoped search, image uploads, soft edit/delete, notifications via Web Push, roles (admin/member), invite-only onboarding, 3-user cap for cloud.
- Out of scope (v1): screen sharing, voice/video, presence/typing/read receipts, advanced search.

### Tenancy and routing

- Deployment model: Single-tenant SaaS (one isolated deployment per customer). Users can belong to multiple workspaces within a deployment.
- Workspace routing: `app.beacon.xyz/{workspaceSlug}`
- Workspace slug: unique per deployment; editable if the new slug remains unique.
- Default channel: create `#general` on workspace creation.

### Authentication and onboarding

- Auth provider: Better Auth with OAuth only (Google + Microsoft). No SSO in v1.
- Workspace creation: anyone can create a workspace.
- Invites: admins can invite via email only; acceptance required to join (no domain auto-join).
- Joining: OAuth sign-in; on invite acceptance, user is added to the target workspace.

### Roles and permissions

- Roles: admin, member. Multiple admins allowed.
- Invites: admins only.
- Channel creation: members can create public channels; admins can create public and private channels.
- Message deletion: admins can delete any message; members can delete only their own.

### Messaging model

- Message content: Markdown (basic formatting). WYSIWYG input.
- Mentions: `@user`, `#channel` with typeahead.
- Threads: single-level replies; no nested threads; no per-thread reply count display in channel list.
- Edits/deletes: unlimited window; soft-delete flags; edits marked with an "edited" flag (no history).

### Files (images only in v1)

- Allowed types: PNG, JPG, WEBP.
- Max size: 5 MB per image.
- Processing: strip EXIF; use the uploaded image as the thumbnail (no separate rendition pipeline in v1).
- Access pattern: images served via authenticated proxy endpoint.
- Storage: Bun's S3 API against S3-compatible storage (e.g., S3, R2, MinIO).

### Search

- Scope: channel-scoped search only.
- Language: English; Postgres FTS.

### Notifications

- Transport: Web Push.
- Triggers: DMs, mentions, replies.
- DND: per-user DND switch to suppress notifications.

### PWA

- Ship an installable Vite PWA for the web app in v1 (basic app shell caching; no offline compose).

### Hosting and infrastructure

- API: Hono + Bun; hosted on Render (region: Singapore).
- Web: TanStack Router app; hosted on Vercel (global edge by default).
- Email: Nodemailer via SMTP for invite emails.
- Database: Postgres (managed; backups policy to be set later). All timestamps in UTC.
- Cache/queues/rate limits: Redis for rate limiting.
- Observability: Sentry for error tracking.

### Monetization and licensing

- Cloud offering monetized via Polar.
- Cloud free tier: limit 3 users per workspace.
- Enforcement: on invite/join, check current workspace member count; block the 4th and surface error.
- Cloud vs self-host detection: environment flag with license key validated against external auth service.

### Realtime (WebSockets)

- Transport: WebSockets.
- Presence/typing: not included in v1.
- Event outline (subject to refinement):
  - `channel.created`, `channel.updated`, `channel.archived`
  - `channel.member.added` (private channels), `channel.member.removed`
  - `message.created`, `message.updated`, `message.deleted`
  - `thread.reply.created`, `thread.reply.updated`, `thread.reply.deleted`
  - `reaction.added`, `reaction.removed`

### Data model (Drizzle + Postgres) — high level

- `user`: id (UUID), display_name, email (unique), avatar_url, created_at
- `workspace`: id, name, slug (unique per deployment), created_at
- `workspace_member`: id, workspace_id, user_id, role (admin|member), created_at
- `channel`: id, workspace_id, name, slug, is_private (bool), created_at
- `channel_member`: id, channel_id, user_id (for private channels), created_at
- `dm_conversation`: id, workspace_id, created_at
- `dm_participant`: id, dm_conversation_id, user_id
- `message`: id, workspace_id, channel_id nullable, dm_conversation_id nullable, author_id, body_md, is_edited (bool), is_deleted (bool), created_at, updated_at
- `thread`: id, root_message_id, created_at
- `thread_reply`: id, thread_id, author_id, body_md, is_edited, is_deleted, created_at, updated_at
- `reaction`: id, message_id nullable, thread_reply_id nullable, user_id, emoji (unicode), created_at
- `file_asset`: id, workspace_id, uploader_id, message_id nullable, thread_reply_id nullable, mime_type, byte_size, storage_key, created_at
- `invite`: id, workspace_id, email, token, created_at, accepted_at nullable, expires_at nullable
- `oauth_account`: id, user_id, provider (google|microsoft), provider_account_id
- `notification_subscription`: id, user_id, endpoint, p256dh, auth, created_at
- Indices for FTS on `message.body_md` and `thread_reply.body_md` (English config), plus common foreign keys.

### API surface (Hono) — initial outline

- `POST /auth/oauth/:provider/callback` (Better Auth handler)
- `POST /workspaces` — create workspace (creates `#general`)
- `GET /workspaces/:workspaceId` — fetch details
- `PATCH /workspaces/:workspaceId` — update name/slug
- `POST /workspaces/:workspaceId/invites` — admins send invite (SMTP)
- `POST /invites/:token/accept` — accept invite (OAuth required)
- `GET /workspaces/:workspaceId/channels` / `POST /workspaces/:workspaceId/channels`
- `POST /channels/:channelId/members` — add member (private channels)
- `GET /channels/:channelId/messages` / `POST /channels/:channelId/messages`
- `PATCH /messages/:messageId` / `DELETE /messages/:messageId`
- `POST /threads/:messageId/replies` / `PATCH /replies/:replyId` / `DELETE /replies/:replyId`
- `POST /messages/:id/reactions` / `DELETE /reactions/:reactionId`
- `POST /uploads/images` — authenticated proxy upload; strips EXIF; stores via Bun S3
- `GET /assets/:fileId` — authenticated proxy fetch (checks workspace membership)
- `GET /channels/:channelId/search?q=` — channel-scoped search (FTS)
- `POST /notifications/subscribe` — save Web Push subscription

### Rate limiting (Redis) — examples

- Login/OAuth callback, invite creation, message create/edit/delete, upload endpoint.

### Security notes

- Enforce per-workspace membership checks for all channel, message, asset, and search routes.
- Soft-delete flags respected in queries and search.
- Authenticated image proxy only; no public buckets.

### Milestones

1. Foundation
   - Repo setup (Turborepo). API app (Hono+Bun). Web app (TanStack Router + Vite PWA).
   - Drizzle + Postgres schema and migrations. Better Auth (Google/Microsoft). Basic layout + routing.
2. Workspaces & Channels
   - Create workspace, slug edit, default `#general`. Admin/member roles. Public/private channels.
   - Invites (SMTP). License flag + 3-user cap enforcement (cloud only).
3. Messaging
   - Channel messages with Markdown WYSIWYG, mentions. Edits/deletes (soft). Unicode reactions.
   - WebSockets wiring and event broadcasting. DMs and single-level threads.
4. Files & Search
   - Image uploads via proxy (EXIF strip), S3 storage via Bun API, auth fetch endpoint.
   - Channel-scoped FTS search (English), result highlighting in UI.
5. Notifications & PWA polish
   - Web Push subscriptions, triggers for DMs/mentions/replies. User DND toggle.
   - PWA manifest/icons/basic caching; deploy to Vercel/Render (SG). Sentry + rate limits.

### Open questions

- Postgres backups cadence and retention policy.
- Any compliance/export requirements (deferred).
