# Migration: Project Site → GitHub User Site

Migrating this [academicpages](https://github.com/academicpages/academicpages.github.io)
(Jekyll) site from a **project site** served under a subdirectory to a **user site**
served at the domain root.

| | Old (project site) | New (user site) |
|---|---|---|
| Repository | `Rasikamurali/Rasikamurali.github.io` | `Rasikamurali/Rasikamurali.github.io` |
| Production URL | `https://rasikamurali.github.io/Rasikamurali.github.io/` | `https://rasikamurali.github.io/` |
| `baseurl` | `""` | `""` (unchanged) |

The repository name **must** be exactly `rasikamurali.github.io` for GitHub Pages to
serve it at the root of `https://rasikamurali.github.io/`.

---

## Changes made (code)

All internal links, asset paths, CSS/JS, the CV iframe, favicons, and the sitemap page
already use the theme's `base_path` helper (derived from `site.url` + `site.baseurl`),
so correcting `site.url` fixes every path at once. No hard-coded subdirectory paths
existed anywhere except `_config.yml`.

1. **`_config.yml`**
   - `url` → `https://rasikamurali.github.io` (was the old subdirectory URL).
   - `repository` → `Rasikamurali/Rasikamurali.github.io` (new repo name; drives `site.github` metadata).
   - `baseurl` → left `""` (correct for a root-served user site).
   - `description` → updated to a research-focused summary (also the SEO/OG fallback).
   - `og_description` → set to the provided research description (Open Graph fallback).
   - `og_image` → intentionally left blank (setting it makes the theme emit a misleading
     `Organization` JSON-LD; the share image + `Person` schema are set in custom head instead).

2. **`_includes/seo.html`** — added support for a per-page `seo_title` front-matter key
   that fully overrides the `<title>` (falls back to the theme's default behavior otherwise).
   Non-invasive: only pages that set `seo_title` are affected.

3. **`_pages/about.md`** (homepage) — added front matter:
   - `seo_title: "Rasika Muralidharan | LLM Multi-Agent Systems, Social Norms, and AI Safety"`
   - `description:` a grounded research summary for the meta description / OG / Twitter card.
   - Visible page content and design are unchanged.

4. **`_includes/head/custom.html`**
   - Added a default `og:image`, and `twitter:card` / `twitter:title` / `twitter:description`
     / `twitter:image` (the theme only emitted Twitter tags when a Twitter username was set).
   - Added a homepage-only schema.org **`Person`** JSON-LD block. Every field is grounded in
     `_config.yml` or the info provided (name, `jobTitle`, affiliations, `knowsAbout` research
     areas, `description`, `email`, and `sameAs`: Google Scholar, GitHub, Bluesky). No awards,
     extra titles, or profiles were invented.
   - Changed the one favicon line that used a root-absolute path (`/images/favicon.ico`) to use
     `base_path` for consistency.

5. **`robots.txt`** (new, repo root) — allows all crawlers and points to the sitemap:
   ```
   User-agent: *
   Allow: /

   Sitemap: https://rasikamurali.github.io/sitemap.xml
   ```

### Not changed (already correct)
- **Sitemap** — generated automatically by the `jekyll-sitemap` plugin at `/sitemap.xml`.
- **Navigation** (`_data/navigation.yml`) — already uses root-relative URLs (`/publications/`, etc.).
- **Favicons** — already present in `images/` and referenced in custom head.
- **No `noindex`** directive exists anywhere in the site.

---

## Local build / validation command

The project builds with Jekyll via the included Docker/devcontainer setup (no local Ruby needed):

```bash
# Option A — Docker Compose (serves at http://localhost:4000)
docker compose up

# Option B — VS Code: "Reopen in Container" (uses .devcontainer), then it serves automatically

# Option C — local Ruby toolchain
bundle install
bundle exec jekyll build          # production build into _site/
bundle exec jekyll serve -l -H localhost   # live preview at http://localhost:4000
```

> Note: `_config.yml` changes are **not** hot-reloaded — restart the server after editing it.

In this migration, `_config.yml` (YAML) and the new `Person` JSON-LD were validated as
well-formed; the full Jekyll build must be run in the devcontainer/Docker or with a local
Ruby install, as no Ruby toolchain was available in the editing environment.

---

## GitHub Pages deployment steps

This site uses GitHub Pages' **native Jekyll build** (no custom Actions deploy workflow —
the only workflow, `scrape_talks.yml`, just scrapes talk data).

1. ✅ **Done** — the repository was renamed on GitHub to **`Rasikamurali.github.io`**
   (Settings → General → Repository name). GitHub serves the Pages host in lowercase, so the
   site URL is still `https://rasikamurali.github.io/`.
2. Update your local remote to match:
   ```bash
   git remote set-url origin https://github.com/Rasikamurali/Rasikamurali.github.io.git
   ```
3. Commit and push these changes to the default branch (`master`).
4. **Settings → Pages**: set **Source = Deploy from a branch**, Branch = `master` (or `main`),
   folder = `/ (root)`.
5. Wait for the Pages build to finish, then load **https://rasikamurali.github.io/**.
6. (Optional) The old project URL will 404 after the rename. GitHub auto-redirects the old
   repo name to the new one for git operations, but not for Pages URLs.

---

## Google Search Console

1. Go to <https://search.google.com/search-console> and add a property.
   - Easiest: **URL prefix** property for `https://rasikamurali.github.io/`.
2. **Verify ownership** using the *HTML tag* method: Search Console gives a
   `google-site-verification` content string. Paste it into `_config.yml`:
   ```yaml
   google_site_verification : "YOUR_TOKEN_HERE"
   ```
   (The theme already renders this meta tag when the value is set — see `_includes/seo.html`.)
   Commit, push, wait for deploy, then click **Verify**.
3. **Submit the sitemap**: in Search Console → *Sitemaps*, submit `sitemap.xml`
   (full URL `https://rasikamurali.github.io/sitemap.xml`).

### Request indexing
- In Search Console, use the **URL Inspection** tool: enter
  `https://rasikamurali.github.io/`, then click **Request Indexing**. Repeat for key pages
  (`/publications/`, `/cv/`, `/talks/`).
- Indexing is not instant — allow days to weeks.

---

## Verify the canonical URL

- Load the homepage, View Source, and confirm:
  ```html
  <link rel="canonical" href="https://rasikamurali.github.io/">
  <title>Rasika Muralidharan | LLM Multi-Agent Systems, Social Norms, and AI Safety</title>
  <meta property="og:url" content="https://rasikamurali.github.io/">
  ```
- Confirm the canonical does **not** contain `/rasikamuralidharan.github.io/`.
- Validate the structured data at <https://validator.schema.org/> or
  <https://search.google.com/test/rich-results> (paste the live URL) and confirm the
  `Person` entity is detected with the expected fields.
- Confirm no `noindex`: `curl -s https://rasikamurali.github.io/ | grep -i noindex` returns nothing.
- Confirm `robots.txt`: open `https://rasikamurali.github.io/robots.txt`.

---

## Test for broken links

After deploy, run a link checker against the live site:

```bash
# html-proofer against the built _site (run in the devcontainer / with Ruby)
bundle exec htmlproofer ./_site --disable-external

# or a crawler against the live URL
npx linkinator https://rasikamurali.github.io/ --recurse
# or
npx broken-link-checker https://rasikamurali.github.io/ -ro
```

Pay special attention to: the CV PDF iframe (`/files/cv.pdf`), navigation links,
publication/talk links, the profile image, and favicons.

---

## Assumptions
- GitHub username is `rasikamurali`; the repo will be renamed to `rasikamurali.github.io`
  and served at the root. (The current git remote owner is `Rasikamurali`; GitHub owner
  names are case-insensitive.)
- Default branch is `master` (current) — adjust the Pages source if you switch to `main`.
- The Bluesky `sameAs` URL was derived from the handle in `_config.yml`
  (`https://bsky.app/profile/rasikamurali.bsky.social`). Correct it if your handle differs.
- `orcid` / `pubmed` in `_config.yml` are still template placeholders and were **excluded**
  from the structured data (not invented). Fill them in and add to `sameAs` when ready.
