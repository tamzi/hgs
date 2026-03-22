# HGS — Game Marketplace Product Spec

## Vision

HGS is a browser-based game marketplace where users discover, purchase, and play HTML5 games — think Steam meets Poki for casual/indie browser games. The platform houses a growing catalog (currently 26 titles) spanning casino, sports, puzzle, and action genres.

---

## Game Catalog (Current — 26 Titles)

| Category | Games |
|----------|-------|
| Casino/Card | Baccarat, Red Dog, Roulette Royale, 3D Soccer Slot |
| Sports | Real Tennis, American Football Kicks, Arcade Golf, 8 Ball Pool, Ski Rush |
| Action/Arcade | Snake Attack, Virus Attack, Zap Aliens, Swing Jetpack, Cube Ninja, Fat Shark, The Bandit Hunter |
| Puzzle/Casual | Word Finder, Christmas Memory, Tap the Tile, Halloween Breaker, Going Nuts, Soap Ball Craze |
| Platformer | Burger Time, Gingerman Rescue, My Zombie Classmates |

---

## Feature Breakdown

### Phase 1 — MVP (Landing Page + Core Experience)

#### 1.1 Landing Page / Storefront

- Hero section with featured/promoted game
- Game grid with thumbnail cards (screenshot, title, category badge, price)
- Category filters: Casino, Sports, Action, Puzzle, Platformer, All
- Search bar with instant filtering by game title
- "Free to Try" vs "Premium" visual distinction on cards
- Responsive layout — works on desktop, tablet, and mobile
- Footer with about, terms of service, privacy policy, contact

#### 1.2 Game Detail Page

- Game title, description, screenshots/preview GIF
- Play button (launches game in embedded iframe or fullscreen)
- Category tags and difficulty indicator
- "Similar Games" suggestions
- Star rating (aggregate)
- Device compatibility info (desktop, mobile, touch support)

#### 1.3 User Accounts

- Sign up / log in (email + password)
- OAuth options: Google, GitHub (low friction)
- User profile page: username, avatar, games owned, play history
- Email verification
- Password reset flow

#### 1.4 Monetization — Pay to Play

- **Credit system**: Users buy credits (HGS Coins) with real money
  - Credit packs: e.g., 100 coins ($0.99), 500 coins ($3.99), 1200 coins ($7.99)
  - Games cost credits to unlock (e.g., 50–200 coins depending on game)
  - Once unlocked, a game is playable forever (not per-session)
- **Payment integration**: Stripe for credit card / Apple Pay / Google Pay
- **Alternative model to consider**: Monthly subscription ($4.99/mo) for unlimited access to all games
- **Free tier**: Allow 1-2 free games or time-limited trials (e.g., 3 minutes of play before paywall)
- Purchase confirmation and receipt via email
- Transaction history on user profile

#### 1.5 Game Player Experience

- Games launch in an iframe container within the HGS shell (keeps navbar/branding visible)
- Fullscreen toggle
- Quick-exit button to return to catalog
- Loading spinner/progress bar while game assets load
- Orientation lock prompt on mobile (already exists per-game)

#### 1.6 Basic Admin Panel

- View all games, toggle visibility (published/draft)
- Set price (in credits) per game
- View total users, total revenue, active players
- Add/edit game metadata (title, description, category, thumbnail)

---

### Phase 2 — Engagement & Retention

#### 2.1 Leaderboards

- Per-game leaderboards (top scores)
- Global leaderboard (total score across all games)
- Weekly/monthly/all-time filters
- Friend leaderboards (see Phase 2.4)
- Server-side score validation to prevent client-side cheating (critical for casino games)

#### 2.2 Achievements & Progress

- Per-game achievements (e.g., "Score 1000 in Snake Attack", "Win 5 hands in Baccarat")
- Achievement badges displayed on user profile
- Progress tracking: games played, time played, win streaks
- Cloud save: persist game state (credits/money in casino games, level progress) server-side

#### 2.3 Reviews & Ratings

- 1–5 star rating per game (one rating per user)
- Optional text review
- Helpful/not helpful voting on reviews
- Sort games by rating on catalog page

#### 2.4 Social Features

- Friends list (add by username or invite link)
- Activity feed: "X just scored 5000 in Ski Rush"
- Share scores to Twitter/Facebook/WhatsApp
- Challenge a friend: send a link to beat your score

#### 2.5 Notifications

- Email notifications: new game added, friend activity, promotional credits
- In-app notification bell: achievement unlocked, friend request, weekly digest

---

### Phase 3 — Growth & Scale

#### 3.1 Developer/Publisher Portal

- Allow third-party developers to submit HTML5 games
- Game submission form: upload zip, set metadata, set price
- Revenue share model (e.g., 70/30 developer/platform split)
- Developer dashboard: views, plays, revenue, ratings
- Automated game review process (file size limits, security scan, playtest)

#### 3.2 Advanced Discovery

- "Recommended for you" based on play history
- "Trending this week" based on play count
- Curated collections: "Best Casino Games", "Quick 5-Minute Games", "New This Month"
- Tags system beyond categories (e.g., "multiplayer", "retro", "relaxing")
- Sort by: newest, most popular, highest rated, price

#### 3.3 Tournaments & Events

- Time-limited tournaments on specific games
- Entry fee (in credits) with prize pool
- Live leaderboard during tournament
- Seasonal events: holiday themes, special challenges

#### 3.4 Analytics Dashboard (Admin)

- DAU/MAU, retention curves, session length
- Revenue per game, conversion rate (free → paid)
- Funnel: visit → signup → purchase → play
- Game performance: load time, error rate, bounce rate
- Cohort analysis: do users who play casino games also play action games?

#### 3.5 Referral & Loyalty Program

- Referral links: give 50 free credits to referrer and referee
- Daily login bonus (small credit reward)
- "Play 5 games this week" challenges for bonus credits

---

## Technical Architecture

### Frontend

- **Framework**: React or Next.js (SSR for SEO, fast page loads)
- **Game container**: Sandboxed iframe with `postMessage` API for score reporting
  - Strict `targetOrigin` on all `postMessage` calls (never use `"*"`)
  - Origin allowlist validation on incoming messages via `event.origin`
  - Typed message schema (e.g., `{ type: "score", payload: { value: number }, nonce: string, timestamp: number }`)
  - Nonce and timestamp validation to prevent replay attacks
- **Styling**: Tailwind CSS for rapid, responsive UI
- **State management**: User auth state, game library, credits balance

### Backend

- **API**: Node.js (Express or Fastify) or Python (FastAPI)
- **Database**: PostgreSQL — users, games, transactions, scores, achievements
- **Auth**: JWT tokens + refresh tokens, OAuth2 for social login
- **Payments**: Stripe API (checkout sessions, webhooks for confirmation)
- **File storage**: S3-compatible (game assets, thumbnails, user avatars)

### Infrastructure

- **Hosting**: Vercel/Netlify (frontend) + Railway/Render/AWS (backend)
- **CDN**: CloudFront or Cloudflare for game assets (games are 1–17 MB each)
- **Security**:
  - HTTPS everywhere
  - Content Security Policy (CSP) — distinct policies for host app and game iframes:
    - **Host app**: `default-src 'self'; script-src 'self' 'nonce-<random>'; style-src 'self' 'unsafe-inline'; img-src 'self' https://cdn.hgs.com; frame-src 'self' https://games.hgs.com` (nonces required for Next.js hydration/inline bootstrap scripts)
    - **Game iframes**: `default-src 'self'; script-src 'self' 'unsafe-inline'; frame-ancestors https://hgs.com; connect-src 'self'` — initially allows `'unsafe-inline'` because current games rely on inline `<script>` blocks for initialization; tighten to nonce-based or remove `'unsafe-inline'` as inline scripts are refactored to external files
    - `frame-ancestors` directive to restrict which domains can embed games
  - Server-side score validation
  - Rate limiting on API endpoints
  - Input sanitization
  - Stripe webhook signature verification

### Database Schema (Core Tables)

```sql
users          — id, email, username, password_hash, avatar_url, credits_balance, created_at
games          — id, title, slug, description, category, price_credits, thumbnail_url, game_path, is_published, created_at
purchases      — id, user_id, game_id, credits_spent, purchased_at
transactions   — id, user_id, type (purchase/credit_buy/refund), amount, stripe_payment_id, created_at
scores         — id, user_id, game_id, score, validated, created_at
reviews        — id, user_id, game_id, rating, review_text, created_at
achievements   — id, game_id, title, description, criteria_json
user_achievements — id, user_id, achievement_id, unlocked_at
```

---

## Monetization Model — Revenue Projections

| Revenue Stream | Model | Notes |
|----------------|-------|-------|
| Credit packs | One-time purchases | Primary revenue. Stripe takes ~2.9% + $0.30 per transaction |
| Subscription | $4.99/mo unlimited | Alternative/complementary. Higher LTV, predictable revenue |
| Tournaments | Entry fees | Credit sink, keeps users engaged and spending |
| Developer revenue share | 30% platform cut | Phase 3 — scales catalog without building games yourself |

---

## MVP Priorities (Ordered)

1. **Landing page with game catalog** — users can browse and see all 26 games
2. **User authentication** — sign up, log in, profile
3. **Credit system + Stripe integration** — buy credits, unlock games
4. **Game player page** — play games in an iframe container with HGS shell
5. **Basic admin panel** — manage games, view metrics
6. **Leaderboards** — per-game score tracking (server-validated)

---

## Competitive Landscape

| Platform | Model | Strength | HGS Differentiator |
|----------|-------|----------|---------------------|
| Steam | Buy games | Massive catalog, community | Browser-based, no install, instant play |
| itch.io | Pay what you want | Indie friendly, open | Curated quality, built-in casino/card niche |
| Poki | Free + ads | Huge traffic, SEO | No ads, premium feel, pay-to-play |
| CrazyGames | Free + ads + dev revenue share | Large catalog | No ads cluttering experience |
| Kongregate (sunset) | Free + microtransactions | Was the standard | Modern tech, mobile-first |

**HGS positioning**: Premium, ad-free HTML5 game arcade with a casino/card game niche. Instant play in browser, no downloads. Clean UX, curated catalog.

---

## Open Questions for Review

1. **Pricing**: Should casino games cost more credits than casual games?
2. **Free tier**: How many free games (if any) to hook users?
3. **Subscription vs credits**: Offer both? Start with one?
4. **Age verification (Phase 1 blocker)**: Casino-themed games require age verification before launch, even if no real money is wagered in-game. This must include:
   - Geo-gating to restrict access in jurisdictions where simulated gambling is regulated
   - Age-gate flow (date-of-birth entry or equivalent) before accessing casino category games
   - Legal review sign-off on compliance requirements per target market
5. **Multiplayer**: Any plans for real-time multiplayer (e.g., 8 Ball Pool vs a friend)?
6. **Game submission**: Open the platform to third-party developers in Phase 3, or keep it first-party only?
7. **Mobile apps**: Wrap the web app in a PWA or native shell (Capacitor/TWA) for app store presence?
8. **Branding**: "HGS" — does it need a full brand name? Logo? Color scheme?
