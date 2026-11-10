# Novadien SEO deployment checklist

This build includes a crawlable homepage title/description, social sharing metadata, dynamically generated Organization/WebSite JSON-LD, `robots.txt`, `sitemap.xml`, and a health endpoint. The sitemap intentionally lists only the public homepage because the current site is a single-page storefront; private API/admin/supplier routes are excluded from crawling.

## Render environment
- Set `NODE_ENV=production`.
- Set `PUBLIC_SITE_URL` to the final public origin, including `https://` and no trailing slash (for example `https://your-domain.example`). Update it when connecting the real domain.
- Keep `OPENAI_API_KEY`, admin credentials, session secret and supplier access key in Render Environment settings only. Never commit `.env`.

## After deployment
1. Open `/robots.txt`, `/sitemap.xml`, and `/health` on the live domain and confirm they return valid content.
2. Add the exact HTTPS domain/property in Google Search Console and verify ownership (DNS verification is recommended for a custom domain).
3. Submit `https://YOUR-DOMAIN/sitemap.xml` in Search Console.
4. Use URL Inspection on the homepage and request indexing.
5. Ensure the site is publicly accessible without login, returns HTTP 200, and has no accidental `noindex` header or meta tag.
6. Search for the brand and monitor Search Console's indexing and performance reports. Indexing and ranking are controlled by Google and are not guaranteed or immediate.

## Important production note
SEO changes do not make the current JSON-file order storage production-safe. The README still identifies PostgreSQL persistence, customer privacy/access review, supplier notification delivery, and end-to-end/security testing as launch blockers. Deploy privately for staging/testing until those are complete.
