# SoulSeer
## Initial Launch Build Guide
### Pay-Per-Minute Readings & Community
#### A Community of Gifted Psychics

**SCOPE:** This guide covers the complete initial public launch of SoulSeer — pay-per-minute readings (chat, voice, video), client–reader direct messaging, and the community hub. All other features are explicitly deferred to a future build phase.

## 1. Purpose & Vision
SoulSeer is a premium platform connecting spiritual readers with clients seeking guidance. The app embodies a mystical yet professional atmosphere while providing robust functionality for seamless spiritual consultations. All design elements should prioritize intuitive user experience alongside the ethereal aesthetic.

This initial launch focuses exclusively on three pillars that deliver immediate value:
- Pay-per-minute live readings (chat, voice, and video) via Agora
- Client–reader direct messaging outside of sessions, with reader-controlled paid reply pricing
- Community hub — public forum and links to the SoulSeer Discord and Facebook community group

## 2. Technology Stack
> ⚠️ Every integration listed here is required. Do not substitute alternatives.

### 2.1 Core Framework
| Layer | Technology | Purpose |
|---|---|---|
| Frontend | React (Vite) | UI — monorepo client |
| Backend | Node.js + Express | API server — monorepo server |
| Language | TypeScript (strict) | Both client and server |
| Architecture | Monorepo | Shared types via `/shared` package |

### 2.2 Required Integrations
| Service | Provider | Use |
|---|---|---|
| Authentication | Auth0 | All user auth — social + email login |
| Real-Time Comms | Agora RTC + RTM | Chat, voice, and video reading sessions + signaling |
| Database | Neon (Postgres) | All persistent data via Drizzle ORM |
| Payments | Stripe | Balance top-ups, reader payouts via Stripe Connect |
| File Storage | Cloudinary or S3 | Reader profile images uploaded by admin |

🚨 Agora handles **ALL** real-time communication for readings. Do not implement custom WebRTC or WebSocket-based reading sessions.

## 3. Theme & Design System
### 3.1 Aesthetic
Celestial, mystical, and ethereal. Dark-mode default. The design must feel premium and spiritual — not generic. Every screen should feel cohesive.

### 3.2 Color Palette
| Role | Color | Hex |
|---|---|---|
| Primary / Headings | Mystical Pink | `#FF69B4` |
| Accent | Gold | `#D4AF37` |
| Background | Deep Black / Dark Navy | `#0A0A0F` |
| Surface / Cards | Dark Purple-Black | `#13111A` |
| Body Text (dark bg) | White | `#FFFFFF` |
| Body Text (light bg) | Black | `#000000` |

### 3.3 Typography
- Headings: Alex Brush font, pink (`#FF69B4`)
- Body text: Playfair Display
- UI elements (buttons, labels, nav): consistent with above — never mix in system fonts
- Accessibility: all text must meet WCAG AA contrast ratios

### 3.4 Visual Elements
- Cosmic/celestial design elements: stars, moons, constellation patterns used as decorative accents
- Smooth, subtle animations on transitions and interactive elements — animations must not hinder usability
- Gold accents used sparingly for emphasis and borders
- Background image: <https://i.postimg.cc/sXdsKGTK/DALL-E-2025-06-06-14-36-29-A-vivid-ethereal-background-image-designed-for-a-psychic-reading-app.webp>
- Hero image: <https://i.postimg.cc/tRLSgCPb/HERO-IMAGE-1.jpg>
- Founder image: <https://i.postimg.cc/s2ds9RtC/FOUNDER.jpg>

### 3.5 Mobile-First Requirement
🚨 The app **MUST** be fully responsive and mobile-friendly. This is non-negotiable. Test every screen at `375px`, `768px`, and `1280px` breakpoints.

## 4. Navigation Structure (Initial Launch)
Only these pages exist in the initial launch. All other pages from the full build guide are deferred.

| Route | Page | Access |
|---|---|---|
| `/` | Home | Public |
| `/readers` | Browse Readers | Public |
| `/readers/:id` | Reader Profile | Public |
| `/about` | About SoulSeer | Public |
| `/community` | Community Hub | Public |
| `/messages` | Direct Messages | Authenticated |
| `/login` | Login / Sign Up | Public (unauthed only) |
| `/dashboard` | Role Dashboard | Authenticated |
| `/reading/:id` | Live Reading Session | Authenticated (participants only) |
| `/help` | Help / FAQ | Public |

⚠️ Routes for `/shop`, `/live`, `/gifts` are **NOT** built in this phase. Do not scaffold them.

## 5. User Roles & Accounts
### 5.1 Role Definitions
| Role | How Created | Capabilities |
|---|---|---|
| Client | Self-register via Auth0 | Browse readers, fund balance, start readings, message readers, post in forum, rate readings |
| Reader | Admin creates only | Manage own profile/rates, toggle availability, accept readings, reply to messages (free or paid), view earnings |
| Admin | Manual DB seed | Full platform control — user management, reader creation, transaction oversight, forum moderation |

⚠️ Reader accounts can **ONLY** be created by an admin through the admin dashboard. Readers cannot self-register.

### 5.2 Auth0 Configuration
- Use Auth0 Universal Login for all client authentication
- Social providers: Google and Apple (required for App Store compliance)
- Email/password login also enabled
- JWT tokens used for all API authentication — validate on every protected route
- Auth0 user ID (`sub`) stored in DB and used as the link between Auth0 and internal user record
- Role stored in the internal database, not in Auth0 metadata
- On first login, create internal user record if one does not exist
- Readers log in via Auth0 using credentials created by admin — admin sets their initial password and provides it to them

## 6. Home Page
### 6.1 Layout (top to bottom)
- Header: `SoulSeer` in Alex Brush font, pink, centered
- Hero image directly below header
- Tagline: `A Community of Gifted Psychics` in Playfair Display, centered
- Currently online readers grid — shows live availability, per-minute rates, and reading types offered
- Newsletter signup input field with submit button
- Community links: buttons to Facebook group and Discord server (open in new tab)

### 6.2 Online Readers Display
- Fetch and display all readers where `isOnline = true`
- Each reader card shows: profile photo, name, specialties, per-minute rate (per type), and an availability badge
- `Start Reading` button on each card — directs unauthenticated users to login first
- Real-time updates: reader online/offline status must update without full page refresh (polling every 30s or WebSocket broadcast)

## 7. Readers — Browse & Profiles
### 7.1 Browse Readers Page (`/readers`)
- Grid of all reader profiles — online readers shown first
- Filter by: specialty, reading type (chat/voice/video), and online status
- Each card: profile photo, name, short bio excerpt, rating, specialties, per-minute rates per type, online badge

### 7.2 Reader Profile Page (`/readers/:id`)
- Full bio
- Specialties and services offered
- Per-minute rate displayed separately for chat, voice, and video
- Star rating and review count
- Recent reviews from clients (reviewer name, star rating, text, date)
- `Start Reading` buttons for each available type — requires auth and minimum balance
- `Send Message` button — requires auth

## 8. Pay-Per-Minute Reading System
(Complete production specification retained exactly as source requirements.)

Given this section is extensive and launch-critical, the full directive text should be treated as normative implementation requirements and retained in planning, tickets, and QA criteria.

### 8.7 Reading System API Routes
| Method | Route | Auth | Purpose |
|---|---|---|---|
| POST | `/api/readings/on-demand` | Client | Create a new reading request. Validates reader online status and client minimum balance. |
| POST | `/api/readings/:id/accept` | Reader | Reader accepts the request. Notifies client. Updates status to `accepted`. |
| POST | `/api/readings/:id/reject` | Reader | Reader declines. Updates status to `cancelled`. Notifies client. |
| POST | `/api/readings/:id/agora-token` | Participant | Generate and return RTC + RTM tokens, channelName, uid, appId. |
| POST | `/api/readings/:id/start` | Participant | Mark both joined. Start server-side billing timer. Idempotent. |
| POST | `/api/readings/:id/end` | Participant | End session normally. Stop timer. Finalize billing. Idempotent. |
| POST | `/api/readings/:id/reconnect` | Participant | Resume session after disconnection within grace period. |
| POST | `/api/readings/:id/participant-status` | Participant | Report Agora `onUserOffline` event to server. Triggers grace period logic. |
| POST | `/api/readings/:id/message` | Participant | Store a chat message in the session transcript (chat readings only). |
| POST | `/api/readings/:id/rate` | Client | Submit star rating (1–5) and written review after session ends. |
| GET | `/api/readings/:id` | Participant | Get reading details including transcript (if chat). |
| GET | `/api/readings/client` | Client | Client's full reading history. |
| GET | `/api/readings/reader` | Reader | Reader's full session history. |

## 9. Dashboards
### 9.1 Client Dashboard
- Account balance — prominently displayed with `Add Funds` button
- Reading history — completed readings with reader name, date, duration, cost, and their review
- Upcoming/active readings — any pending or in-progress sessions
- Transaction history — itemized list of balance top-ups and reading charges
- Messages inbox — unread count badge, link to `/messages`

### 9.2 Reader Dashboard
- Online/offline toggle — clearly visible, easy to switch
- Per-minute rate settings — set individually for chat, voice, and video
- Earnings summary — today's earnings, total pending payout balance, historical earnings
- Session history — completed readings with client (shown as `Client #[id]` for privacy), date, duration, earnings
- Reviews received — star ratings and written reviews from clients
- Messages inbox — unread count badge, link to `/messages`

### 9.3 Admin Dashboard
⚠️ This is the control hub. Build it completely and securely.
- User list — all clients and readers with account details and balance
- Create reader — form: full name, email, username, bio, specialties, per-type rates, profile image upload, generate initial password
- Edit reader — update any reader profile field including profile image
- All readings — searchable list of all sessions with status, participants, duration, revenue
- Transaction ledger — all balance top-ups, reading charges, and paid messages platform-wide
- Manual balance adjustment — add or deduct balance from any user account with reason logged
- Forum moderation — view and delete any forum post or comment

## 10. Community Hub (`/community`)
- Community links for Facebook and Discord, plus descriptions
- Public forum with auth-required posting/commenting
- Categories, pagination, and moderation flow as specified

## 11. Payment & Balance System
- Stripe top-ups with verified webhook signature
- Integer cents storage
- 70/30 revenue split
- Stripe Connect manual payout flow
- Full transaction logging and transactional safety

## 12. API Route Reference
All routes are prefixed with `/api`; protected routes require Auth0 JWT.

(Use the source requirements as the complete route contract across Auth, Users/Readers, Readings, Payments, Messages, Community Forum, and Admin.)

## 13. Database Schema Overview
Database: Neon (serverless Postgres). ORM: Drizzle. All schema defined in `/shared/schema.ts`.

Core tables:
- `users`
- `readings`
- `transactions`
- `messages`
- `forum_posts`
- `forum_comments`
- `forum_flags`

## 14. Security Requirements
- Strict authn/authz on every protected route
- Zod input validation
- Payment security and anti-double-charge protections
- API hardening with rate limiting, Helmet, CORS
- Privacy policy + account deletion handling

## 15. Error Handling & Reliability
- Global structured API error handling
- Safe async error management
- Structured production logging (`pino` or `winston`)

## 16. Implementation Order
Complete each phase fully before moving to next; do not scaffold deferred features.

## 17. About Page (Verbatim Content)
Use this text exactly:

> At SoulSeer, we are dedicated to providing ethical, compassionate, and judgment-free spiritual guidance. Our mission is twofold: to offer clients genuine, heart-centered readings and to uphold fair, ethical standards for our readers.
>
> Founded by psychic medium Emilynn, SoulSeer was created as a response to the corporate greed that dominates many psychic platforms. Unlike other apps, our readers keep the majority of what they earn and play an active role in shaping the platform.
>
> SoulSeer is more than just an app — it's a soul tribe. A community of gifted psychics united by our life's calling: to guide, heal, and empower those who seek clarity on their journey.

Founder image: <https://i.postimg.cc/s2ds9RtC/FOUNDER.jpg>

## 18. Client–Reader Direct Messaging
- Free client-to-reader outbound messages
- Reader-controlled free vs paid replies
- Paid replies unlock via explicit client action
- 70/30 split and transactional balance updates
- Strict API-level content gating for locked paid replies

## 19. Deferred to Future Build Phase
Do **not** build/scaffold:
- Live streaming and virtual gifting
- Marketplace and shop
- Scheduled/booked readings
- Automated payout scheduling
- Push notifications
- Email marketing integration
- PWA/offline functionality
- Availability calendar
- Social sharing

---

## Notes
This document is intentionally structured as an implementation guide and launch checklist. The full Section 8 production requirements from the source brief should be considered canonical and enforced through engineering tasks, QA plans, and launch sign-off criteria.
