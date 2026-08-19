# Just Aircraft

Daily aggregator of Just Aircraft classified listings (Escapade, Highlander,
SuperSTOL, SuperSTOL XL) from [Barnstormers.com](https://www.barnstormers.com),
published as a static page (`docs/index.html`) meant to be embedded via
`<iframe>` on taildraggers.com.

Controller.com was evaluated (in the companion [Aeronca](https://github.com/taildraggers/aeronca)
repo) and dropped: its search results are only reachable through an internal
client-side widget (not a plain URL), which a headless browser can't drive
reliably for an unattended daily job.

Note: in the companion [Aviat](https://github.com/taildraggers/aviat),
[CubCrafters](https://github.com/taildraggers/cub-crafters),
[de Havilland](https://github.com/taildraggers/de-Havilland),
[Maule](https://github.com/taildraggers/maule),
[Van's RV](https://github.com/taildraggers/vans),
[RANS](https://github.com/taildraggers/rans), and
[Luscombe](https://github.com/taildraggers/luscombe) repos, Barnstormers'
single-manufacturer category pages turned out to include unrelated
listings mixed in with no distinguishing HTML markup. This repo is built
with the same fix from day one: `scraper/barnstormers.py` filters by title
against a small allowlist of Just Aircraft product names before publishing
(see `TARGET_MODEL_PHRASES` in `scraper/barnstormers.py`).

Just Aircraft has no numbered model-code system like RANS (`S-6`, `S-7`...)
or Luscombe (`8A`-`8F`) — model names ARE the identifiers. **SuperSTOL**
(and **SuperSTOL XL**) is a coined/branded term unique to this
manufacturer, so it's trusted standalone. **Highlander** and **Escapade**
are plain English words with real collision risk (Toyota Highlander,
generic use of "escapade") — a lesson learned the hard way in the
companion Piper repo, where a bare "Cub" mislabeled non-Piper homebuilts
as genuine Pipers — so each of those also requires the title to say "Just"
explicitly (see `_extract_model` in `scraper/barnstormers.py`). Titles
that read as parts, accessories, services, or raffles are dropped. Every
surviving listing's title is rewritten to a canonical **`YEAR JUST
AIRCRAFT MODEL`** form when the ad states a model year (e.g. `2019 Just
Aircraft SuperSTOL XL`), or just **`JUST AIRCRAFT MODEL`** when it doesn't
— a missing year isn't disqualifying, since plenty of genuine ads simply
don't state one in the title.

**taildragger-only exclusion:** the Highlander and SuperSTOL/SuperSTOL XL
are always conventional (tailwheel) gear by design, but the Escapade is
factory-offered as either tricycle or tailwheel — the model name alone
doesn't say which a given for-sale aircraft has. So, the same policy first
applied in the companion RANS repo (and carried into Luscombe): any
individual ad of any model whose own text explicitly says tricycle gear,
trike gear, or nosewheel is dropped.

## How it works

- `scraper/barnstormers.py` searches Barnstormers.com's Just Aircraft
  category for listings, follows pagination, then keeps only the ones
  whose URL slug matches the product-name allowlist (Barnstormers builds
  each listing's URL slug directly from the ad's own title, so this runs
  before any detail page is fetched). For the matches, it visits each
  listing's detail page to pull out the price, location, and posted date
  (falling back to regex heuristics over the visible text since the site
  doesn't expose structured data). The title is derived from the listing
  URL's own SEO slug, since every detail page shares one generic
  `<title>`/`<h1>`; the final parsed title is checked against the
  allowlist again as a safety net.
- `main.py` runs the scraper, de-duplicates results, sorts them
  newest-posted-first, and renders them into `docs/index.html` titled
  **"Other Just Aircraft Ads on the Web"**, with one row per listing:
  Title (linked to the original ad), Price, Location, Date Posted, and
  Site Posted On. Below phone width, each row collapses into a card
  (title + price on one line, location/date/site on a smaller line below)
  instead of a horizontally-scrolling table. Below the table, a "Search
  More Just Aircraft Listings" section links out to Trade-A-Plane,
  Controller, and ASO — sites that block automated scraping, but are
  still worth sending visitors to directly via a pre-filled search. Links
  use `rel="noopener noreferrer"` and the page sets a `no-referrer` meta
  policy, so none of these sites see that the click came from
  taildraggers.com.
- `.github/workflows/daily-scrape.yml` runs the whole thing once a day (13:00 UTC),
  commits the regenerated `docs/index.html` if it changed, and can also be triggered
  manually from the Actions tab (`workflow_dispatch`).

## One-time setup: enable GitHub Pages

This repo publishes `docs/index.html` as a plain static file — GitHub Pages just needs
to be pointed at it once:

1. Go to **Settings → Pages** in this repository.
2. Under **Build and deployment → Source**, choose **Deploy from a branch**.
3. Branch: `main`, folder: `/docs`. Save.
4. GitHub will publish the page at `https://taildraggers.github.io/just-aircraft/`
   (may take a minute or two the first time).

Also check **Settings → Actions → General**:
- **Actions permissions**: "Allow all actions and reusable workflows".
- **Workflow permissions**: "Read and write permissions" (needed so the daily
  job can commit the regenerated page back to the repo).

## Embedding on taildraggers.com

```html
<iframe
  src="https://taildraggers.github.io/just-aircraft/"
  title="Other Just Aircraft Ads on the Web"
  style="width: 100%; height: 800px; border: 0;"
  loading="lazy">
</iframe>
```

The page also posts its rendered height to the parent window on load/resize
(`{ type: "taildraggers:resize", height }`) so it can be auto-sized instead
of using a fixed guessed height - add a matching `message` listener on the
embedding page to pick this up.

## Running locally

```bash
pip install -r requirements.txt
playwright install --with-deps chromium
python main.py
```

This writes/overwrites `docs/index.html`.

## Notes

- If Barnstormers changes its markup or is briefly unreachable, the run logs will
  show a `[warn]`/`[error]` line pointing at what broke rather than failing silently.
- The scraper identifies itself with a browser-like `User-Agent` and adds a short
  delay between requests to be polite to the site.
- Only one Barnstormers category is currently configured
  (`category-18860-Experimental--Just-Aircraft.html`). If listings turn out
  to be split across additional categories, add more URLs to
  `CATEGORY_URLS` in `scraper/barnstormers.py`.
