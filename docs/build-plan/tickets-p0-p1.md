# Toki — P0 and P1 tickets

Owner roles: FS1–FS3 full-stack engineers, ME realtime/media engineer, DS designer, OPS part-time DevOps. Estimates are ideal engineering days; plan about 40% on top for review, bugs and meetings.

- P0 Foundations (weeks 1–2): 24 tickets, 42 days
- P1 Messaging core (weeks 3–8): 41 tickets, 66.5 days

## P0 · Foundations (weeks 1–2)

### Repo & delivery

- **T-001 Monorepo scaffold** · FS1 · 2d · wk 1
  - pnpm workspaces + Turborepo with apps/web, apps/api, apps/gateway, apps/mobile (placeholder), packages/contracts, packages/ui, packages/config. TypeScript strict, ESLint, Prettier. `pnpm build` and `pnpm test` pass from the root and each app boots locally.
- **T-002 Local dev stack** · OPS · 1d · wk 1 · after T-001
  - Docker Compose for Postgres 16, Redis 7, MinIO, Mailpit and a LiveKit dev server. `pnpm dev` starts everything; README steps work on a fresh machine.
- **T-003 CI pipeline** · OPS · 1.5d · wk 1 · after T-001
  - GitHub Actions: lint, typecheck, unit tests, build, Playwright smoke, dependency cache. Required check on main.
- **T-004 Staging and PR previews in Singapore** · OPS · 3d · wk 2 · after T-003
  - Cloud account in the Singapore region, Kubernetes staging namespace, managed Postgres, Redis and S3, secrets manager. Every PR gets a preview URL for the web app.
- **T-005 Migrations and seed data** · FS2 · 1.5d · wk 1 · after T-002
  - Drizzle migrations run in CI and on deploy. Seed matches the prototype: Acme workspace; Alice, Bob, Chi, Dũng, Hà and the guest Khách Mời; #general, #design, #du-an-alpha (private), #releases.
- **T-006 Observability baseline** · ME · 1.5d · wk 2 · after T-004
  - Structured JSON logs, OpenTelemetry traces across api and gateway, Sentry for web and api, /healthz and /readyz endpoints, a staging dashboard.
- **T-007 Apple and Google developer accounts** · OPS · 0.5d · wk 1
  - Apple Developer (organisation) and Google Play accounts, bundle IDs, APNs key, Firebase project for FCM. Started in week 1 because approval can take two weeks.
- **T-008 Expo app skeleton and build lanes** · FS3 · 3d · wk 2 · after T-001, T-007 · board: PhoneList
  - Expo app in apps/mobile importing packages/contracts and the API client; EAS internal builds for iOS and Android from CI. Placeholder sign-in screen. Prepared before the mobile hire joins.
### Identity

- **T-010 Identity schema with tenant isolation** · FS2 · 2d · wk 1 · after T-005
  - Tables workspace, user, membership (owner/admin/member/guest), session, device. Postgres row-level security on workspace_id; a test proves one workspace cannot read another's rows.
- **T-011 Company sign-in (OIDC, then SAML)** · FS2 · 3d · wk 2 · after T-010 · board: SignIn
  - OIDC for Google Workspace and Microsoft Entra, SAML via a maintained library. Users from verified company domains are provisioned on first sign-in. Audit event on every sign-in.
- **T-012 Email sign-in** · FS2 · 1.5d · wk 2 · after T-010 · board: SignIn
  - Magic link valid 15 minutes, single use; display name asked on first sign-in. Emails go to Mailpit locally.
- **T-013 Test personas (dev only)** · FS2 · 0.5d · wk 2 · after T-005 · board: SignIn
  - “Sign in as a test person” for Owner, Member and Admin seeds behind DEV_PERSONAS. A test fails the build if the route exists in a production bundle.
- **T-014 Sessions and revocation** · FS2 · 1.5d · wk 2 · after T-010, T-040
  - httpOnly, SameSite=Lax cookie sessions with rotation; device record; API to list and revoke all sessions (used later by “Sign out everywhere”). Revocation closes open WebSockets within 5 s.
- **T-015 Sign-in screen** · FS3 · 1.5d · wk 2 · after T-011, T-012, T-021 · board: SignIn
  - Built to the SignIn board: company account, test personas, email + display name, language picker. Error states for expired link and unknown domain.
- **T-016 Hash-chained audit writer** · FS3 · 1.5d · wk 2 · after T-010 · board: Admin
  - Append-only audit_event with prev_hash and SHA-256 hash, written in the same transaction as the change. Covers sign-in, session revoke, role change. Verifier ships later; the chain starts now.
### Design system

- **T-020 Design tokens and themes** · DS · 1.5d · wk 1 · after T-001 · board: Main
  - Port the prototype's dark and light tokens, Inter and JetBrains Mono, comfortable and compact density. Theme setting: match system, light, dark; toggle in the rail.
- **T-021 Core components** · FS1 · 4d · wk 1 · after T-020
  - Button, IconButton, Avatar (hue initials + presence dot), Badge, Tabs, Menu, Dialog, Toggle, Select, Input, Tooltip in packages/ui with Storybook. axe accessibility checks in CI.
- **T-022 App shell** · FS1 · 2d · wk 2 · after T-021 · board: Rail
  - Icon rail + side panel + main pane, collapsible side panel, routes for Home, DMs, Activity, Files, Later; active view styling and badges slots.
- **T-023 Visual regression harness** · FS1 · 1d · wk 2 · after T-022, T-003
  - Playwright screenshots of the shell and components in both themes, compared on every PR.
### Language

- **T-030 i18n framework** · FS3 · 1.5d · wk 1 · after T-001
  - ICU message catalogs, English strings extracted from the boards, Vietnamese catalog scaffolded. Locale-aware dates with 24-hour default and per-user time zone.
- **T-031 Vietnamese text utilities** · FS3 · 1d · wk 1 · after T-001 · board: Search
  - NFC normalisation on write; accent folding for search and autocomplete (“phat hanh” matches “phát hành”, đ → d). Shared in packages/contracts with unit tests.
### Realtime

- **T-040 WebSocket gateway** · ME · 3d · wk 1 · after T-002
  - Session-authenticated WebSocket, heartbeat, client reconnect with backoff, Redis pub/sub fan-out to user and conversation channels. Horizontal scale proven with two gateway pods.
- **T-041 Event envelope and sequence numbers** · ME · 2d · wk 2 · after T-040
  - Typed event contracts; per-conversation seq; client resumes from last seq and fetches gaps over REST, so no message is lost across reconnects.
- **T-042 Connection indicator** · FS1 · 0.5d · wk 2 · after T-040, T-022 · board: SidePanel
  - “Connected” state in the side panel; offline banner and reconnecting state.

## P1 · Messaging core (weeks 3–8)

### Conversations

- **T-100 Conversation and membership API** · FS2 · 2d · wk 3 · after T-010 · board: Channels
  - Kinds public, private, dm, group_dm; create, update, join, leave; member role manager. Permission checks for private channels.
- **T-101 Create a channel** · FS3 · 1d · wk 3 · after T-100 · board: Channels
  - Dialog with name (lowercase, hyphens, Vietnamese folded to a slug), private toggle, description. Audit event channel.create.
- **T-102 Channel directory** · FS3 · 2d · wk 4 · after T-100, T-031 · board: Channels
  - Search by name or description, Any/Only joined, A to Z, member counts, Joined/Private/Archived badges, Join, Leave, Open.
- **T-103 DMs and group DMs** · FS2 · 2.5d · wk 4 · after T-100 · board: DMs
  - Open-or-create 1:1 DM; group DMs up to 8 people; New message picker; DM inbox with previews and unread counts; intro block with View profile.
- **T-104 Make private and archive** · FS2 · 1d · wk 7 · after T-100, T-016 · board: ChannelDetails
  - One-way private conversion with in-page confirmation; archive makes a channel read-only and hides it from the sidebar. Both audited.
### Messages

- **T-110 Message model and send API** · FS2 · 2d · wk 3 · after T-100, T-041 · board: Channel
  - Rich-text JSON (ProseMirror schema shared in contracts), idempotency key, seq assignment, per-user rate limit, message.created event.
- **T-111 Message list** · FS1 · 3d · wk 3 · after T-110 · board: Channel
  - Virtualised list, day dividers, grouping of consecutive messages, load older, jump to latest. 50-message channel opens under 500 ms.
- **T-112 Optimistic send and failures** · FS1 · 1d · wk 4 · after T-111 · board: Channel
  - Pending and failed states, retry and discard, ordering preserved when sends resolve out of order.
- **T-113 Edit and delete** · FS2 · 1.5d · wk 5 · after T-110 · board: Channel
  - Edit window from workspace setting (default no limit), “edited” label, delete leaves a tombstone when the message has replies.
- **T-114 Reactions** · FS3 · 1.5d · wk 5 · after T-110, T-132 · board: Channel
  - Add and remove, counts, who-reacted tooltip, quick reactions; reaction events.
- **T-115 Pins** · FS3 · 1d · wk 6 · after T-110 · board: Channel
  - Pin and unpin, Pinned label on the message, Pins tab with count.
- **T-116 Mentions and channel links** · FS3 · 1.5d · wk 5 · after T-110 · board: Channel
  - @person, @channel, @here, @everyone (permission hook), #channel links; own mentions highlighted.
- **T-117 Code formatting** · FS1 · 0.5d · wk 5 · after T-111 · board: Channel
  - Inline code and code blocks in the monospace face with a copy button.
- **T-118 Link unfurls** · FS2 · 2.5d · wk 6 · after T-110 · board: Channel
  - Worker fetches Open Graph data with SSRF protection and a 3 s timeout, caches results, skips blocked domains; preview card with title, description and domain.
- **T-119 Typing indicator** · ME · 1d · wk 4 · after T-041 · board: Channel
  - Throttled typing events through Redis, 6 s expiry, “X is typing…” line; respects the “Show when people are typing” preference.
- **T-120 Presence** · ME · 1.5d · wk 4 · after T-040 · board: SidePanel
  - Online and away from gateway heartbeats and idle detection; presence dots everywhere avatars appear; online-first DM ordering.
### Composer

- **T-130 Composer editor** · FS1 · 3d · wk 3 · after T-110 · board: Composer
  - Tiptap editor on the shared schema: bold, italic, strikethrough, link, bulleted and numbered lists, quote, code, code block, with keyboard shortcuts; Aa toggle hides the toolbar.
- **T-131 Mention and channel autocomplete** · FS1 · 1.5d · wk 4 · after T-130, T-031 · board: Composer
  - @ and # pickers, accent-insensitive, members ranked first, keyboard navigation.
- **T-132 Emoji picker** · FS3 · 1d · wk 4 · after T-021 · board: Composer
  - Search, recently used, skin tone; shared by composer and reactions.
- **T-133 Send key setting** · FS1 · 0.5d · wk 6 · after T-130, T-180 · board: Preferences
  - Enter sends by default; “Ctrl+Enter sends; Enter starts a new line” option.
- **T-134 Drafts** · FS3 · 1d · wk 6 · after T-130 · board: SidePanel
  - Unsent text saved per conversation on the server, restored on any device, listed under Drafts.
### Files

- **T-140 Upload pipeline** · FS2 · 3d · wk 4 · after T-004 · board: Composer
  - Presigned S3 uploads with progress, size limit, MIME sniffing, ClamAV scan job, image thumbnails and video posters.
- **T-141 Attachments in messages** · FS3 · 1.5d · wk 5 · after T-140, T-111 · board: Channel
  - Inline images and video per preference, file cards with size and type, Files & links tab per conversation.
### Threads

- **T-150 Thread model and summary** · FS2 · 1.5d · wk 5 · after T-110 · board: Channel
  - thread_root_id, reply count, last reply time and participant avatars on the root; thread.replied event.
- **T-151 Thread pane** · FS1 · 2d · wk 6 · after T-150, T-130 · board: Thread
  - Built to the Thread board: root and replies, reply composer, “Also send to channel”.
- **T-152 Follow and unfollow** · FS2 · 0.5d · wk 6 · after T-150 · board: Thread
  - Auto-follow when you post or are mentioned; manual Follow/Unfollow in the pane.
### Read state & navigation

- **T-160 Read state and unread counts** · ME · 2d · wk 5 · after T-041, T-111 · board: Channel
  - last_read_seq per member, unread and mention counts, mark read on view, “New messages” divider, counts synced across devices.
- **T-161 Side panel** · FS3 · 2.5d · wk 5 · after T-022, T-160 · board: SidePanel
  - Starred, Channels and Direct messages sections; unread bold + badge; star and unstar; Find a conversation; filter and sort menu.
- **T-162 Side panel shortcuts** · FS3 · 1d · wk 7 · after T-161 · board: SidePanel
  - All unreads, Threads (followed threads list), Drafts and Directories entries.
- **T-163 Jump to (Ctrl K)** · FS1 · 1.5d · wk 6 · after T-031, T-100 · board: JumpTo
  - Quick switcher over channels and people, fuzzy and accent-insensitive, recent first, full keyboard support.
- **T-164 Channel header** · FS1 · 1.5d · wk 4 · after T-100 · board: Channel
  - Name, topic, member stack and count, star, notification-setting stub, Messages / Files & links / Pins tabs, Meet button placeholder.
### People & profile

- **T-170 Profile fields** · FS3 · 1.5d · wk 6 · after T-010, T-140 · board: Preferences
  - Display name, full name, title, pronouns, phone, time zone, photo upload with crop.
- **T-171 Profile card** · FS3 · 1d · wk 7 · after T-170 · board: Profile
  - Popover from any avatar: names, title, local time, email, phone, Message action.
- **T-172 Avatar and workspace menu** · FS1 · 0.5d · wk 7 · after T-022 · board: Profile
  - Workspace info, Jump to, Browse channels, Preferences, Sign out wired; Admin console item for admins only.
- **T-173 People directory** · FS2 · 1.5d · wk 7 · after T-170, T-031 · board: People
  - Search by name, title or email; Title and Location filters; presence; Most relevant sort.
- **T-174 Channel details** · FS2 · 2.5d · wk 7 · after T-100, T-104 · board: ChannelDetails
  - About (name, topic, description), member list, Make manager, Remove, Add people, “Only channel managers can post” enforced by the API, Leave.
### Preferences

- **T-180 Preferences store and Settings dialog** · FS3 · 2d · wk 5 · after T-021 · board: Preferences
  - Typed preference schema in contracts, one row per user, synced live. Settings dialog with all 12 tabs; Profile & status and Messages & media wired, the rest labelled as coming later.
### Meet spike

- **T-190 LiveKit on staging** · ME · 4d · wk 7 · after T-004 · board: Meet
  - LiveKit with TURN on staging, token endpoint, one room per conversation, two-person call on a test page from two networks.
### Quality

- **T-195 End-to-end flows** · FS1 · 2d · wk 8 · after T-151, T-141
  - Playwright: sign in, create channel, post, react, reply in thread, DM, upload a file. Runs on every PR in under 8 minutes.
- **T-196 Load smoke test** · ME · 1.5d · wk 8 · after T-160
  - 200 simulated clients across 10 channels; send-to-render p95 under 300 ms on staging; report filed.
- **T-197 Week 8 pilot in #du-an-alpha** · DS · 0.5d · wk 8 · after T-195
  - Move one project channel onto Toki for real work; collect issues in a feedback log; triage into P2/P3.
