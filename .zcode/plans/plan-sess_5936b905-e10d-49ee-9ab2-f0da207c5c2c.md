Research and recommend the best tech stack for a personal blog hosted on GitHub Pages with custom domain (blog.sh1d0w.com).

## Recommendation Summary

**Tech Stack:** Astro.js + Tailwind CSS + Markdown/MDX + GitHub Pages (CI/CD via GitHub Actions)

**Why Astro (vs Next.js/Hugo):**
- Ships 0 KB JavaScript by default (vs 463 KB Next.js)
- Native Markdown/MDX support
- Every theme has dark/light mode built-in
- Official adapter-static for GitHub Pages
- 2026 consensus: "Astro wins on content projects"

**Recommended Theme: AstroPaper** (github.com/satnaing/astro-paper)
- Dark/light mode toggle with system preference detection
- Clean, minimal, readable typography
- Home page with post index, pagination, client-side search (Pagefind)
- Blog post view with collapsible TOC
- About/bio page via content collections
- SEO-optimized, accessible, Lighthouse perfect scores

**GitHub Pages Deployment (3 files needed):**
1. `astro.config.mjs` — `output: 'static'`, `site: 'https://blog.sh1d0w.com'`, `adapter: staticAdapter()`
2. `.github/workflows/deploy.yml` — GitHub Actions workflow with `withastro/action`, `pages: write` + `id-token: write` permissions
3. `public/CNAME` — Contains `blog.sh1d0w.com`
4. DNS — CNAME record: `blog.sh1d0w.com` → `yourusername.github.io`

**No implementation needed yet — user is in planning/research phase. Awaiting approval to scaffold the project.**