# Fairmind Research Tools

Internal tools for Fairmind Jewellery Valuations, Chester. Built by Hugh, summer intern.
Static site, hosted on GitHub Pages at `hugh-parry.github.io/Fairmind-Research-Tool/`.

## What this is

A set of open-source research tools for jewellery and watch valuers. Currently one tool:
watch market research, which helps a valuer gather current asking prices for a piece.

**This is a starting point for research, not a valuation.** The tool never produces a
value, and nothing it outputs goes into a client document unaltered. The valuer's own
appraisal — inspection, condition, provenance, purpose of valuation — happens separately
and afterwards. Keep this framing in all copy. Avoid the word "valuation" for anything
the tool itself does.

## Hard constraint: no automated fetching

Chrono24 and Watchfinder have no public API and their terms of service prohibit
automated scraping. **Do not add any code that fetches, scrapes, or proxies listing data
from these sites.** This is a deliberate decision, not an oversight or a gap to fill.

The tool works by:
1. Building a pre-filtered search URL from the form values (no network request)
2. The valuer clicks through, reads the real listings themselves
3. The valuer types the prices back into the tool
4. The tool does arithmetic and record-keeping on what was entered

If a change would require the page to request data from a listing site, it is out of
scope — flag it rather than implementing it. The one exception is eBay, which has an
official Browse API; adding that is permitted but needs an API key and is not yet built.

## Files

- `index.html` — homepage, lists available tools
- `watch-research.html` — watch tool (live retail listings only)
- `jewellery-research.html` — jewellery tool (auction results + retail listings)
- `styles.css` — shared stylesheet for all pages
- `CLAUDE.md` — this file

## Jewellery tool: two rules that must not be relaxed

1. **Auction and retail figures are never averaged together.** A hammer price is a price
   paid in competition; a retail listing is a price asked. They are summarised in separate
   blocks and outliers are judged within each market separately. Any change that produces
   a single combined average is wrong.
2. **Auction figures include buyer's premium.** Hammer alone understates what the buyer
   actually paid, often by 25%+. The premium field is recorded per lot and applied in
   `gbp()`.

Jewellery has no equivalent of a watch reference number, so comparables match on
description and can differ materially. The outlier threshold is deliberately wider than
the watch tool's (30% vs 25%) and the caveat text says so explicitly. Don't tighten it
without discussing with Annabell.

No build step, no framework, no dependencies beyond Google Fonts. Plain HTML, CSS and
vanilla JavaScript, deliberately, so it stays maintainable after the internship ends.

## Styling

Matches fairmind.co.uk: thin wide-tracked uppercase wordmark, powder-blue nav bar and
page-title band, coral pink for buttons and the current nav item, light-weight
Montserrat on white.

All colours and fonts are CSS variables in `:root` at the top of `styles.css`. Change
them there only — never hardcode a colour elsewhere in the file.

Current values are estimated from a screenshot and are not yet confirmed against the
live site. `--blue`, `--pink` and `--font` still need verifying.

## Conventions

- British English throughout (valuation, jewellery, colour, organise)
- Prices in GBP; FX rates entered manually by the valuer with a stated source, never
  fetched live, so figures can be reproduced later
- No localStorage or browser storage — nothing persists between sessions by design.
  Export the CSV instead.
- Comments should explain *why*, especially where a constraint above is the reason

## Open work

- [x] Verify the search URL patterns in `SOURCES` in `watch-research.html`.
      All four confirmed: eBay, Watchfinder, Chrono24 (needs `dosearch=true`
      alongside `query=`), and Watches of Switzerland (`/search?q=`).
- [x] Verify the search URL patterns in `SOURCES` in `jewellery-research.html`.
      All nine confirmed. Date/status filters (past sales vs upcoming lots) are
      deliberately left to each site's own controls, not the URL.
- [ ] Agree default buyer's premium rates with Annabell, or leave per-lot as now
- [ ] Confirm `--blue`, `--pink` and `--font` against the live Fairmind site
- [ ] Review the outlier rule with Annabell. Currently a flat 25% from the median
      (`OUTLIER_THRESHOLD`), which is a blunt instrument and hasn't been agreed.
- [ ] Decide whether the two "coming soon" cards on the homepage stay

## Context to keep in mind

Hugh is an undergraduate, not a professional developer, and needs to be able to explain
how everything works. Prefer clear, commented, boring code over clever or compact code.
Explain changes rather than just making them.
