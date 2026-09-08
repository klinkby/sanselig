# Archived site: sanselig.dk

Static offline snapshot of the old sanselig.dk website (HTML pages, images,
and stylesheets), scraped on 2026-09-08 for archival purposes.

- Source: https://www.sanselig.dk/ (per its sitemap.xml)
- Fetched with `wget --mirror --page-requisites --convert-links`
- Links between pages have been rewritten to relative paths so the archive
  can be browsed offline by opening `index.html` in a browser.
- Pages behind login/checkout (`/my`, `/_account`, `/shop/checkout`, etc.)
  were excluded, matching the site's own `robots.txt` disallow rules.
- A handful of internal links pointed at `sanselig.dk` without the `www`
  prefix; those pages were merged into the same tree so the archive is
  self-contained.
