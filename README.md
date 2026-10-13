# Abdelghani Elyagouti: Data Science & AI Portfolio

A static, English-only portfolio (HTML, CSS, vanilla JS) with a private admin dashboard. Content lives in Supabase, so the whole site can be edited without touching code. Hosted on GitHub Pages, no build step.

**Live site:** your GitHub Pages URL · **Admin:** `/admin.html`

## Features

- Sections in this order: Home, About, Education, Experience, Skills, Services, Projects, Certificates, Contact. **Contact is always last**: locked in the admin, enforced by the renderer, and enforced by a database constraint.
- Dynamic **Skills** and **Services**: create any categories, add/edit/delete/reorder items, optional icon, image, link and description.
- **Projects** on Kaggle, GitHub, Live Demo or Other, with "View on Kaggle / View on GitHub / Live Demo / View Project" buttons, notebook/dataset/competition detection for Kaggle URLs, thumbnail fallback, platform + category filters, publish/unpublish, reorder. External links open in a new tab with `rel="noopener noreferrer"`.
- Admin: every heading, label, button text, link and footer text is editable. Enable/disable and reorder sections, draft + **live preview** (desktop/tablet/mobile) + publish, image/CV upload with client-side compression, social links, message inbox, JSON backup/restore, one-click import from the previous version's tables.
- Motion: staggered hero entrance, floating portrait, drifting glow orbs, scrolling skills ticker, scroll-reveal, scroll progress bar, pointer spotlight on cards. All disabled automatically for visitors who prefer reduced motion.
- Contact shows LinkedIn, GitHub and other links as cards (edit them in the admin under Contact).
- Contact form stores messages in Supabase (honeypot, minimum fill time, send cooldown, DB length checks).

## One-time setup (required)

1. **Run the schema.** Supabase Dashboard → SQL Editor → paste `supabase/schema.sql` → Run. It is safe to re-run and does not touch your old tables.
2. **Create your admin user.** Authentication → Users → *Add user* (email + strong password). Then run, with your email:
   ```sql
   insert into public.admins (user_id)
   select id from auth.users where email = 'you@example.com' on conflict do nothing;
   ```
3. **Disable public sign-ups** (Authentication → Providers → Email → turn off *Allow new users to sign up*). Even if left on, only users in `admins` can write.
4. Open `/admin.html`, sign in, check the **Setup check** on the Dashboard, then **Publish**.
5. *(Optional)* Backup → **Import old content** copies your previous tables into the draft.

The config in the HTML files already points at your existing Supabase project and contains only the **publishable** key. Never add a `service_role` key, database password or any secret to this repository.

## How security works

| Concern | Protection |
|---|---|
| Who can edit | Supabase Auth login **and** a row in `admins`, checked by Row Level Security (`is_admin()`), not by hidden links or frontend passwords |
| Public data | Visitors can only `SELECT` from `portfolio_published` and `INSERT` unread messages |
| Drafts, messages, uploads | Admin-only read/write (RLS and storage policies) |
| Contact last | `CHECK` constraint on both content tables |
| XSS / unsafe links | All text escaped; only `http(s)`, `mailto:`, `tel:`, `#hash`, and safe relative image paths are rendered |

## Files

```
index.html    public site (HTML + CSS + JS in one file)
admin.html    private dashboard (HTML + CSS + JS in one file)
schema.sql    Supabase tables, RLS, storage
README.md
```

Nothing else is needed: no `assets/` or `images/` folder. The default portrait is embedded in the code; upload your own from the admin (Home / Hero) and it replaces it. Supabase URL and the **publishable** key are at the top of the script in each HTML file (`window.PORTFOLIO_CONFIG`). Keep both files in sync if you ever change them.

Content is one JSON document (`portfolio_draft` -> `portfolio_published`). Starter content is shown until the first publish.

## Run locally

```bash
python -m http.server 8000     # then open http://localhost:8000
```

## Known limitations

- The code was tested in Chromium against a mock Supabase that emulates the RLS rules; **`schema.sql` was syntax-checked but not executed**. Run it and use the Dashboard *Setup check* to confirm.
- The contact form has no CAPTCHA. For heavier spam, add Cloudflare Turnstile and verify it in a Supabase Edge Function.
- Previous in-page edit mode and FR/EN + light/dark toggles were removed (English-only, dark navy design); editing now lives in `/admin.html`. Old `site_content` text overrides are not migrated.
- Kaggle notebooks are linked, not embedded.
