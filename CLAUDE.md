# Lark Documentation Project

## What is Lark?

Lark is a **Firebase Realtime Database alternative** — a real-time database with sub-50ms p99 latency, designed for multiplayer games and collaborative apps. It provides a JavaScript/TypeScript client SDK (`@lark-sh/client`) with a Firebase v8-style API, and is also compatible with existing Firebase Realtime Database SDKs.

**Target audience**: Developers building real-time apps who want to use Lark as their database. These docs should cover everything they need to get started and be productive.

## Source repositories

- **Server (Rust)**: `~/lark-rust` — architecture, security rules, wire protocol
- **Client SDK (JS/TS)**: `~/lark-js` — the Lark-native SDK
- **Proxy (Go)**: `~/lark-proxy` — REST API, SSE streaming, Firebase wire protocol proxy
- **Dashboard**: `~/lark-admin` — admin panel (React + Cloudflare Workers)

## Documentation structure

### Section 1: What is Lark

Overview of the service, getting oriented, and dashboard usage.

#### Group: Getting started
- **Introduction** (`index.mdx`) — What is Lark, key features (real-time sync, Firebase-compatible, sub-50ms latency, WebSocket + WebTransport, ~20KB native client). Positioning: drop-in Firebase RTDB replacement built for games.
- **Quickstart** (`quickstart.mdx`) — Fastest path to a working app. Create a project in the dashboard, install SDK, connect, write/read/subscribe. Minimal working example.

#### Group: Dashboard
- **Dashboard overview** (`dashboard/overview.mdx`) — Signing in (Google OAuth), projects list, creating a project
- **Project settings** (`dashboard/settings.mdx`) — Project name, project ID, ephemeral mode, auto-create databases, Firebase compatibility toggle (use first path as database, Firebase Auth project ID), secret key management (view/copy/regenerate)
- **Database management** (`dashboard/databases.mdx`) — Listing databases (search, pagination, status/active/inactive), creating databases, deleting databases, the real-time JSON editor (tree view, shallow view for large datasets), import/export JSON
- **Security rules editor** (`dashboard/rules-editor.mdx`) — The JSON5 rules editor in project settings, saving and validating rules. (Links to the full security rules reference in Section 2.)
- **Monitoring** (`dashboard/monitoring.mdx`) — Metrics cards (peak CCU, bandwidth, operations, latency), charts (1h/24h/7d), billing usage (CCU and bandwidth limits, warning thresholds), events log (high latency, approaching limits, connection rejected, storage warnings)
- **Pricing** (`dashboard/pricing.mdx`) — "Coming soon" placeholder page

### Section 2: Platform

Concepts that apply regardless of which SDK you use.

#### Group: Data
- **Data structure** (`platform/data-structure.mdx`) — JSON tree model, projects and databases, paths and keys, references. How data is organized.
- **Reading and writing** (`platform/read-write.mdx`) — Core operations: set (overwrite), update (shallow merge), remove, push (auto-generated keys), multi-path atomic updates, server values (`ServerValue.TIMESTAMP`). Optimistic local-first writes.
- **Sorting and filtering** (`platform/sorting.mdx`) — Firebase value ordering (null < false < true < numbers < strings < objects), 32-bit integer key sorting, orderByChild/orderByKey/orderByValue/orderByPriority, limits (limitToFirst/limitToLast), ranges (startAt/endAt/equalTo)
- **Validation and limits** (`platform/limits.mdx`) — Key rules (max 768 bytes, forbidden chars), string value max (10 MB), write size max (16 MB), volatile write max (2 KB), path depth, no control characters

#### Group: Security rules
- **Security rules overview** (`platform/security-rules.mdx`) — What rules are, how they work, the rules JSON structure, rule types (`.read`, `.write`, `.validate`)
- **Rules reference** (`platform/rules-reference.mdx`) — Available variables (`auth`, `data`, `newData`, `root`, `now`, `$wildcards`), snapshot methods (`val()`, `exists()`, `hasChild()`, `hasChildren()`, `parent()`, `child()`), string methods (`contains()`, `beginsWith()`, `endsWith()`, `matches()`, `length`), operators
- **Rules examples** (`platform/rules-examples.mdx`) — Common patterns: user-owned data, public read/private write, validation (required fields, types), role-based access, game-specific patterns

#### Group: Real-time features
- **Subscriptions** (`platform/subscriptions.mdx`) — Event types (`value`, `child_added`, `child_changed`, `child_removed`, `child_moved`), how delta events work (server sends only changes), shared views
- **OnDisconnect and presence** (`platform/presence.mdx`) — OnDisconnect operations (set/update/remove on disconnect), building presence systems, `.info/connected`
- **Volatile paths** (`platform/volatile.mdx`) — High-frequency data (game state, cursors), fire-and-forget writes, pattern matching, WebTransport datagrams (UDP), batching rates (20Hz WebTransport, 7Hz WebSocket), size limit (2 KB), how to configure volatile paths
- **Transactions** (`platform/transactions.mdx`) — Optimistic callback-style transactions (auto-retry, max 25 retries), multi-path transactions (object and array syntax), condition/CAS checks (value comparison, SHA-256 hash for complex objects)

#### Group: Authentication
- **Authentication overview** (`platform/authentication.mdx`) — How auth works in Lark, token types: anonymous, Lark customer tokens (HS256 JWT signed with secret key), Firebase Auth tokens (RS256, validated by proxy). `auth` object in security rules.

#### Group: Transports
- **Transports** (`platform/transports.mdx`) — WebSocket (universal), WebTransport (Chrome 97+, Edge 97+, Firefox 114+), auto-detection with fallback, connection URLs (`wss://{projectId}.larkdb.net/ws`, `https://{projectId}.larkdb.net:{7778-7809}/wt`), when to prefer WebTransport (games, volatile data, no head-of-line blocking)

### Section 3: Using Lark with Firebase SDKs

For developers with existing Firebase projects or who prefer the Firebase SDK.

#### Group: Getting started
- **Overview** (`firebase/overview.mdx`) — How Lark works as a Firebase RTDB backend, what's supported, what's different. Enable "Allow Legacy Firebase" in project settings.
- **Setup** (`firebase/setup.mdx`) — Configuring a Firebase app to point at Lark instead of Firebase RTDB. Connection URL format, config changes needed.

#### Group: Guides
- **Using Firebase Auth with Lark** (`firebase/auth.mdx`) — Setting the Firebase Auth Project ID in dashboard, how RS256 token validation works (proxy layer validates), using Firebase Auth tokens with Lark
- **Migrating an existing project** (`firebase/migration.mdx`) — Step-by-step: create Lark project, enable Firebase compatibility, configure "use first path as database" if needed, update Firebase config, migrate security rules, export/import data, testing
- **Compatibility notes** (`firebase/compatibility.mdx`) — What Firebase RTDB features are fully supported, any differences or limitations, wire protocol compatibility

### Section 4: REST API

Read and write data over HTTP — no SDK required. 100% compatible with the Firebase Realtime Database REST API.

#### Group: REST API
- **Overview** (`rest-api/overview.mdx`) — What the REST API is, URL format (`https://{projectId}.larkdb.net/{database}/{path}.json`), database-in-subdomain format, authentication via `?auth=` query parameter, operations at a glance (GET/PUT/POST/PATCH/DELETE), when to use the REST API
- **Reading data** (`rest-api/reading.mdx`) — GET requests, shallow reads (`?shallow=true`), querying (`orderBy`, `limitToFirst`, `limitToLast`, `startAt`, `endAt`, `equalTo`), formatting (`print=pretty`, `print=silent`, JSONP, download), timeouts
- **Writing data** (`rest-api/writing.mdx`) — PUT (set), POST (push), PATCH (update), DELETE (remove), multi-path updates, server values (`{".sv": "timestamp"}`), silent writes, error responses
- **Streaming** (`rest-api/streaming.mdx`) — Server-Sent Events (SSE), `Accept: text/event-stream`, event types (`put`, `patch`, `keep-alive`, `cancel`, `auth_revoked`), path semantics, browser EventSource and Node.js examples
- **Conditional requests** (`rest-api/conditional.mdx`) — ETags (`X-Firebase-ETag: true`), conditional writes (`If-Match`), `412 Precondition Failed`, compare-and-swap retry pattern

### Section 5: Using Lark with Lark SDKs

The native Lark JavaScript/TypeScript SDK.

#### Group: Getting started
- **Overview** (`lark-sdk/overview.mdx`) — Why use the Lark SDK (smaller, modern API, WebTransport support, volatile paths, TypeScript-native), installation (`npm install @lark-sh/client`), browser + Node.js support
- **Quickstart** (`lark-sdk/quickstart.mdx`) — Connect, write, read, subscribe, disconnect. Full working example.

#### Group: Guides
- **Connecting** (`lark-sdk/connecting.mdx`) — `LarkDatabase` constructor, `connect()` options (anonymous, token, domain, transport, webtransportTimeout), connection lifecycle, state management (`connected`, `state`, `transportType`), event callbacks (`onConnect`, `onDisconnect`, `onReconnecting`, `onError`)
- **References and paths** (`lark-sdk/references.mdx`) — `db.ref(path)`, `.child()`, `.parent`, `.root`, `.key`, `.path`, path validation
- **Writing data** (`lark-sdk/writing.mdx`) — `set()`, `update()`, `remove()`, `push()`, `setWithPriority()`, `setPriority()`, multi-path updates via `db.ref().update()` with `/` keys
- **Reading data** (`lark-sdk/reading.mdx`) — `once('value')`, `ref.get()`
- **Subscriptions** (`lark-sdk/subscriptions.mdx`) — `on()` returns unsubscribe function, event types, practical examples (game state, leaderboards, chat)
- **Queries** (`lark-sdk/queries.mdx`) — Chainable query methods, `orderByChild()`, `orderByKey()`, `orderByValue()`, `limitToFirst()`, `limitToLast()`, `startAt()`/`startAfter()`, `endAt()`/`endBefore()`, `equalTo()`, `queryIdentifier`
- **Transactions** (`lark-sdk/transactions.mdx`) — Callback-style `ref.transaction()`, `db.transaction()` with object/array syntax, conditions, `result.committed`, `result.snapshot`
- **Authentication** (`lark-sdk/auth.mdx`) — Anonymous connect, `signIn(token)`, `signOut()`, `onAuthStateChanged()`, `db.auth` property
- **Offline support** (`lark-sdk/offline.mdx`) — Auto-reconnection (exponential backoff, 1s→30s), `goOffline()`/`goOnline()` (preserves cache vs `disconnect()` which clears), pending writes (`hasPendingWrites()`, `getPendingWriteCount()`, `clearPendingWrites()`), `.info/connected`
- **OnDisconnect** (`lark-sdk/ondisconnect.mdx`) — `ref.onDisconnect()` → `set()`, `update()`, `remove()`, `setWithPriority()`, `cancel()`
- **Error handling** (`lark-sdk/errors.mdx`) — `LarkError` class, error codes: `permission_denied`, `invalid_data`, `not_found`, `invalid_path`, `timeout`, `not_connected`, `condition_failed`, `max_retries_exceeded`, `write_tainted`, `view_recovering`, `auth_required`
- **Firebase v8 compatibility layer** (`lark-sdk/fb-v8.mdx`) — `@lark-sh/client/fb-v8` sub-package, `on()` returns callback (not unsubscribe), `off()` for cleanup, context parameter support, migration path from Firebase v8 code

#### Group: API reference
- **LarkDatabase** (`lark-sdk/api/lark-database.mdx`) — Full constructor, properties, and methods reference
- **DatabaseReference** (`lark-sdk/api/database-reference.mdx`) — Full properties, navigation, write, read, query, subscription methods
- **DataSnapshot** (`lark-sdk/api/data-snapshot.mdx`) — `val()`, `exists()`, `key`, `ref`, `forEach()`, `hasChildren()`, `hasChild()`, `numChildren()`, `child()`, `getPriority()`, `exportVal()`, `toJSON()`, `isVolatile()`, `getServerTimestamp()`
- **OnDisconnect** (`lark-sdk/api/ondisconnect.mdx`) — All methods
- **LarkError** (`lark-sdk/api/lark-error.mdx`) — Error codes table with descriptions
- **Utilities** (`lark-sdk/api/utilities.mdx`) — `generatePushId()`, `ServerValue.TIMESTAMP`, `normalizePath()`, `joinPath()`, `getParentPath()`, `getKey()`, `isVolatilePath()`

## Key reference data

### Connection URL patterns
- WebSocket: `wss://{projectId}.larkdb.net/ws`
- WebTransport: `https://{projectId}.larkdb.net:{7778-7809}/wt`

### Wire protocol (shortened keys, for internal reference)
Client ops: `j` (join), `au` (auth), `s` (set), `u` (update), `d` (delete), `sb` (subscribe), `us` (unsubscribe), `o` (once), `od` (onDisconnect), `tx` (transaction)
Server events: `a` (ack), `n` (nack), `ev` (event — put/patch), `pi` (ping), `jc` (join complete), `ac` (auth complete)

### SDK package info
- Package: `@lark-sh/client` (npm)
- Version: 0.1.24 (Early Alpha)
- Size: ~20KB gzipped
- Dependencies: `ws` (Node.js WebSocket), `canonicalize` (JCS for transaction hashing)
- Supports: browsers + Node.js, CJS + ESM

## Voice and tone

Lark's voice is **conversational, technically credible, and confidently direct** — a founder speaking to developers, not a corporation speaking to customers. Think PostHog's irreverence, Heroku's "there is no step 2" magic, and PlanetScale's familiar-but-innovative positioning.

### Core principles

- **Confident but honest** — Make bold claims, but back them with specifics. "Sub-50ms p99 latency" not "blazing fast". Never overhype.
- **Direct and personal** — Second person ("you"), active voice, short sentences. Write like you're explaining to a smart friend, not writing a spec.
- **Technical credibility first** — Specificity over abstraction. "30Hz cursor updates" not "low-latency streaming". Developers trust numbers and concrete details.
- **Enablement over constraint** — Frame features as unlocking what developers want to build, not as a checklist. "Stop doing math and start building" energy.
- **Playful without being cutesy** — Parenthetical personality is good: "(Doing this on Firebase would bankrupt you by Tuesday.)" But humor serves the story, never the other way around.

### Lark is its own thing

Lark is inspired by Firebase, but it's its own product. Document Lark features as **Lark features** — they're "Lark security rules", "Lark transactions", "Lark subscriptions." Don't annotate every concept with its Firebase equivalent. A developer reading the Platform or Lark SDK sections should learn Lark on its own terms, not through a lens of "it's like Firebase but..."

The Firebase section (Section 3) is where we explicitly discuss Firebase compatibility, migration, and how Lark maps to Firebase concepts. That's the right place for it. Everywhere else, Lark stands alone.

### What to avoid

- Corporate speak, marketing fluff, or vague superlatives ("industry-leading", "best-in-class")
- Hedging language ("might", "could potentially") — be direct
- Talking down to the reader — assume they're experienced developers
- Constant Firebase references outside of Section 3 — don't say "hasChildren() (like Firebase)" or "Lark's security rules work just like Firebase's". Lark is Lark.
- Being snarky about competitors outside of where it's natural and warranted

### Formatting conventions

- Sentence case for headings
- Bold for UI elements: Click **Settings**
- Code formatting for file names, commands, paths, and code references
- All code examples in TypeScript
- Show practical, real-world examples (game state, presence, leaderboards, chat)
- The SDK is `@lark-sh/client` — always use the full package name in install commands
- Status: Early Alpha — mention once in the intro, don't repeat everywhere
- When referencing dashboard UI, describe the location (e.g., "in **Project Settings** under **Security Rules**")
- Platform section should be SDK-agnostic — show concepts, not specific SDK calls
- SDK-specific sections should show concrete code with imports
