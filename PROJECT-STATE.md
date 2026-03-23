# Mining Forum Event Analytics — Project State

**Last updated:** March 17, 2026
**Status:** Full design rebuild — interactive filters, 6 views, header theming

---

## Architecture

Static HTML/JS/CSS site deployed on Vercel. No build step — just serves files from `public/`.

- **Supabase** for all data (no hardcoded data)
- **Chart.js 4.4.1** via CDN for charts
- **@supabase/supabase-js v2** via CDN (`https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2`)
- **Inter** font via Google Fonts
- **vercel.json** with `outputDirectory: "public"`

## Supabase

- **Project ID:** ljyogcspkvqgjbiyzfbn
- **URL:** https://ljyogcspkvqgjbiyzfbn.supabase.co
- **Anon key:** eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImxqeW9nY3Nwa3ZxZ2piaXl6ZmJuIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzMyODE5OTAsImV4cCI6MjA4ODg1Nzk5MH0.oBKfsngoLIUjZl-w-iZm3W8Q4kMuFAAICidEdszNyGE
- **Org ID:** utzvdrbrvnfskfibimvv
- **Tables:**
  - `companies` — 205 rows (DGG unduplicated membership)
  - `event_participations` — 266 rows (77 MFE26 + 189 MFA26)
- **RLS:** Public SELECT enabled on both tables

### Schema

```sql
companies: id (PK int), company_name, company_status, primary_mineral, primary_country,
  primary_region, primary_subregion, primary_stock_exchange, stock_symbol, ticker,
  currency, stock_price_usd, fifty_two_week_range, one_year_return, market_cap_display,
  market_cap_usd, production_low, production_high, reserves, resources, profile_url

event_participations: id (serial PK), company_id (FK→companies.id), event_code,
  participation_status, company_name, company_status, primary_mineral, primary_country,
  primary_region, primary_subregion, primary_stock_exchange, stock_symbol, ticker,
  currency, stock_price_usd, fifty_two_week_range, one_year_return, market_cap_display,
  market_cap_usd, production_low, production_high, reserves, resources,
  presentation_type, presentation_date, presentation_time, presentation_location,
  payment_status, attendance, profile_url, webcast_url
  UNIQUE(company_id, event_code)
```

## File Structure

```
analytics/
  vercel.json              — Vercel config (outputDirectory: "public")
  package.json             — Minimal package.json for project identity
  public/
    index.html             — SPA entry point (tabs: DGG / MFE / MFA, 6 views: Overview / By Mineral / By Status / By Country / Production & Reserves / Flat Rank)
    css/styles.css         — Full design system (970 lines): filter pills, sticky legend, tooltips, cards, tables, responsive
    js/
      supabase-client.js   — Supabase init + fetch functions + helpers (formatMcap, groupBy, mineral groups, status colors, flags)
      overview.js          — Overview dashboard: summary boxes, mineral/status pills, doughnut charts, bar chart, top-15 table
      solitaire.js         — Solitaire card renderers + interactive mineral/status/country filter bars + flat rank view
      production-reserves.js — 4-tab view: Combined, Production Guidance, Reserves & Resources, MCap/Ounce valuation
      app.js               — Main controller: section/view routing, header theming, interactive filter system with toggle/reset
    logos/
      dgg-logo.webp        — Denver Gold Group logo (white on transparent)
      mfe26-logo.png       — Mining Forum Europe 2026 shield logo
      mfa26-logo.png       — Mining Forum Americas 2026 shield logo
```

## Section Key Colors

| Section | Key Color | Hex |
|---------|-----------|-----|
| DGG Membership | Gold/Amber | #8B6914 |
| MFE 2026 | Dark Green | #1B5E20 |
| MFA 2026 | Dark Blue | #1A237E |

## Design System (from SKILL.md)

- Header gradient: #1B2631 → #2C3E50
- Gold accent: #D4A017
- Body bg: #F0F2F5
- Font: Inter 400-800
- Status colors: Producer=#27AE60, Royalty=#2980B9, Dev(c/f)=#E67E22, Dev(PEA)=#F39C12, Exp(adv)=#9B59B6, Exp(early)=#1ABC9C, Bullion=#95A5A6
- Mineral groups: Gold=#D4A017, Silver=#7F8C8D, PGMs=#8E44AD, Copper=#CA6F1E, Other=#27AE60

## Known Cleanup Items

These are things to address in the next session:

1. **Visual polish** — Review actual deployed appearance, adjust spacing/sizing
2. **Section header branding** — Each section tab should change the header gradient to use the section's key color, not just swap the logo
3. **Production/reserves badges** — Only showing production; should show reserves and resources badges too
4. **Gold-equivalent conversion** — Not yet implemented in the client-side code; needed for proper ranking
5. **MCap/Ounce valuation** — The blended valuation metric from SKILL.md (65% prod, 30% res, 5% rsrc) is not implemented
6. **Country flags** — May be missing some countries; check against actual data
7. **Responsive testing** — Verify mobile layout works
8. **Disclaimer text** — Should use the official Denver Gold Group legal disclaimer
9. **GitHub repo** — git@github.com:DenverGold/analytics.git — not yet connected (sandbox has no SSH). User may want to push manually and connect Vercel via git integration
10. **Event-specific fields** — MFE/MFA views have extra fields (presentation_type, presentation_date, etc.) that aren't displayed yet
11. **CSV import** — Future data imports via CSV matching current seed structure — no import UI yet

## Skill Reference

The mining-forum-viz skill is at:
`/sessions/festive-admiring-mayer/mnt/Microsite Analytics Project/mining-forum-viz.skill`

Extract with: `unzip mining-forum-viz.skill -d skill-extract/mining-forum-viz/`

Key file: `SKILL.md` — Contains gold-equivalent conversion ratios, visual design system, solitaire layout patterns, common pitfalls, and LAMP integration details.

## Vercel

- **Team:** tim-9574's projects (team_gAj4p4fTeuDgX67ryVh6uX9V)
- **Deploy command:** `cd analytics && vercel deploy`
- npm is blocked in the Cowork sandbox; Vercel CLI must be run from user's machine
