# Car Shopping App — Project Context

## What This Is
A personal-use car shopping app — essentially a Cars.com knockoff where I can customize the UX, filters, and data presentation to fix things that bother me about existing car shopping sites. I am the only user.

## Stack Decisions
- **Framework:** Next.js (App Router) deployed on Vercel free tier
- **Database:** Postgres — likely self-hosted on my Mac Mini via Homebrew, accessed over Tailscale. May also use Neon (Launch tier) as a backup or alternative.
- **Language:** TypeScript
- **Styling:** Tailwind CSS

I have not used Next.js before. I'm comfortable with Claude Code handling the heavy lifting, but explain non-obvious Next.js patterns (app router file conventions, server vs client components, data fetching approaches) briefly when introducing them rather than assuming I know.

## Infrastructure
- **Mac Mini (Apple Silicon):** Runs 24/7, no sleep. This is the primary dev machine and potential DB host. I access it remotely via Tailscale + SSH from my phone using Claude Code CLI in tmux.
- **Tailscale:** Used for secure remote access — no ports exposed to the public internet.
- **Vercel:** Deployment target. Free tier is fine for personal use.
- **GitHub:** Repos managed via Claude Code. Merge to main when things are working. Keep branches tidy.

## Development Workflow
- I work primarily through Claude Code CLI, sometimes via SSH from my phone.
- I use `claude --continue` and `claude --resume` to pick up sessions across devices.
- Prefer direct commits to main for straightforward changes. Use branches for larger features.
- When creating new files or restructuring, explain what you're doing and why briefly — don't just silently rearrange things.

## Data Sources
- TBD — potential options include Marketcheck API, CarGurus unofficial endpoints, or scraping. Start with a clean data model that can accept listings from any source.
- The data model should support: make, model, year, trim, mileage, price, location, exterior/interior color, drivetrain, transmission, fuel type, body style, dealer vs private, listing URL, photos, and a flexible metadata/tags field for anything else.

## Design Preferences
- Clean, fast, minimal UI. No visual clutter.
- Filter-heavy interface — the whole point is better filtering than existing sites.
- Mobile-friendly since I'll often browse from my phone.
- Don't over-engineer. This is a personal tool, not a startup. Prefer simple and working over architecturally elegant.

## Decision Context & Open Questions

This section captures the reasoning behind choices and things still being figured out. It's here so future sessions have the full picture, not just the conclusions.

### Why Next.js + Claude Code over vibe-coding platforms
We evaluated Lovable, Bolt, v0, Replit, and similar "vibe coding" platforms. They're good for getting a generic CRUD scaffold up fast, but this project's entire value prop is customizing the stuff that bothers me about existing car sites — filter logic, data model, UX details. Those platforms make the easy parts easy but fight you on customization, which is the whole point here. Claude Code + Next.js gives full control where it matters most, and I'm already comfortable with Claude Code as a workflow.

### Database: self-hosted Postgres vs Neon
Currently paying for Neon Launch tier (~$5/mo minimum, usage-based). The Mac Mini already runs 24/7 drawing ~6W idle ($8-9/yr electricity), so self-hosting Postgres on it is nearly free. Tradeoffs considered:
- **Pro self-hosting:** Zero cold-start latency (Neon scale-to-zero has ~500ms-1s wake), full extension control (TimescaleDB etc.), no egress costs for local access, saves ~$60-90/yr.
- **Pro Neon:** Automatic backups, point-in-time restore, accessible from anywhere without Tailscale, zero maintenance on Postgres upgrades, no disruption if Mini dies.
- **Current lean:** Self-host on Mini with Tailscale for access, set up cron backups to cloud storage. This also serves a separate fitness/time-series project that benefits from local TimescaleDB.
- **Not decided yet.** Could go either way or use both (Neon for the Vercel-deployed web app, local Postgres for development).

### Vercel + self-hosted DB connectivity
If the app deploys to Vercel but the DB is on the Mini, Vercel's serverless functions need to reach the Mini over Tailscale or a tunnel. This may add complexity. Alternatives: use Neon for production and local Postgres for dev, or run the app on the Mini too (skip Vercel). This needs to be figured out early.

### Data sourcing is the hard part
The app is only as good as its data. Options under consideration:
- **Marketcheck API:** Legitimate paid API with dealer inventory data. Cleanest option but has cost.
- **CarGurus unofficial endpoints:** Free but fragile, could break anytime.
- **Scraping Cars.com / Autotrader / etc.:** Legally gray, technically brittle, but comprehensive.
- **Manual entry / CSV import:** Lowest-tech option — just paste listings I find interesting. Might be the right v1 approach: build the filter/browse UI first, worry about automated ingestion later.
- No decision made yet. The data model is designed to be source-agnostic so we can start with any approach and swap later.

### Mobile access pattern
I'll frequently browse listings from my phone. This means the UI needs to be genuinely mobile-first, not just responsive-as-afterthought. Filters especially need to work well on small screens — this is one of the main UX complaints with existing car sites.

### Scope philosophy
This is a personal tool. It doesn't need to handle millions of listings or concurrent users. It's fine to make choices that wouldn't scale (e.g., client-side filtering of a few thousand listings, SQLite instead of Postgres if that's simpler, polling instead of webhooks). Optimize for "I can build and change this quickly" over architectural correctness.

## Things To Avoid
- No auth system needed — it's just me.
- No analytics or tracking.
- No complex deployment pipelines. `git push` → Vercel auto-deploys is the workflow.
- Don't install heavy dependencies for simple problems. Prefer built-in Next.js/React capabilities.
