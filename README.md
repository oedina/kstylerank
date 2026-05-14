# KStyleRank

**The data-driven ranking platform for Korean drama and K-pop fashion styles.**

Live at → [yourusername.github.io/kstylerank](https://yourusername.github.io/kstylerank)

---

## What it is

KStyleRank tracks and ranks the most popular clothing styles from Korean dramas, K-pop idols, and Seoul street fashion — updated weekly, scored by a hybrid algorithm combining real data signals with community votes.

### Ranking algorithm

| Signal | Weight | Source |
|--------|--------|--------|
| Google Trends (KR) | 21% | Search volume, normalised 0–100 |
| Hashtag frequency | 21% | Naver, Instagram, TikTok |
| Drama viewership | 18% | Nielsen Korea, Netflix charts |
| Community votes | 40% | Net likes, 30-day recency decay |

> Data signals = 60% of final score. Community votes = 40%, recency-weighted so older votes decay over 30 days.

---

## Features

- **Weekly rankings** — 9 styles ranked by hybrid score, updated every Monday
- **Detail drawer** — click any card to see outfit breakdown, drama sources, and raw signal scores
- **Wear it / Skip it voting** — community votes shift rankings in real time
- **Filter by category** — K-Drama, K-Pop, Seoul Street, Rising
- **Fendi-inspired luxury editorial design** — Cormorant Garamond serif, cream and ink palette
- **AI-illustrated or editorial photos** — no licensed celebrity images, zero copyright risk

---

## Tech stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML, CSS, JavaScript — no framework |
| Fonts | Google Fonts (Cormorant Garamond + Montserrat) |
| Photos | Unsplash (free editorial license) |
| Hosting | GitHub Pages (free) |
| Backend | — *(coming soon: Supabase + Vercel)* |

---

## Roadmap

### Phase 1 — GitHub Pages (now)
- [x] Weekly style rankings
- [x] Hybrid scoring algorithm (data + community)
- [x] Detail drawer with outfit breakdown
- [x] Wear it / Skip it voting
- [x] Filter by category

### Phase 2 — Backend (next)
- [ ] Persistent votes across sessions (Supabase)
- [ ] User accounts and vote history
- [ ] Weekly automated score recalculation
- [ ] Drama and idol detail pages

### Phase 3 — Real domain & monetisation
- [ ] Custom domain (kstylerank.com)
- [ ] Affiliate links inside detail drawer (Shop this look)
- [ ] Brand sponsorship placements
- [ ] Premium early-access trend reports
- [ ] B2B data API for fashion brands

---

## Data sources — April 2026

Scores are computed from:
- **Google Trends KR** — weekly search interest, normalised 0–100
- **Naver shopping + blog hashtag volume** — estimated from trending data
- **Drama viewership** — Nielsen Korea cable ratings + Netflix Top 10 KR charts
- **Community votes** — seeded from simulated baseline, updated by real user votes

---

## Running locally

No build step needed. Just open `index.html` in any browser:

```bash
git clone https://github.com/yourusername/kstylerank.git
cd kstylerank
open index.html        # macOS
start index.html       # Windows
xdg-open index.html   # Linux
```

---

## Deploying to GitHub Pages

1. Push this repo to GitHub (must be **public**)
2. Go to **Settings → Pages**
3. Source: `Deploy from a branch` → branch: `main`, folder: `/ (root)`
4. Save — live in ~2 minutes at `https://yourusername.github.io/kstylerank`

---

## Adding a custom domain later

1. Buy a domain from Namecheap or Google Domains
2. In repo **Settings → Pages**, enter your custom domain
3. In your domain registrar, add a CNAME record:
   - Name: `www`
   - Value: `yourusername.github.io`
4. HTTPS is automatic — no SSL certificate setup needed

---

## Contributing

Spotted a wrong drama source? Want to suggest a style for next week's ranking? Open an issue or pull request.

---

## License

MIT — free to use, fork, and build on.

---

*KStyleRank is an independent fan project. All drama and idol references are for editorial and informational purposes. Photos sourced from Unsplash under free editorial license. AI illustrations are original works.*
