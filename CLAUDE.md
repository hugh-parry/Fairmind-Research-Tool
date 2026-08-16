# Fairmind Research Tools

Internal tools for Fairmind Jewellery Valuations, Chester. Built by Hugh, summer intern.
Static site, hosted on GitHub Pages at `hugh-parry.github.io/Fairmind-Research-Tool/`.

## What this is

A set of open-source research tools for jewellery and watch valuers:
- watch market research — current asking prices for a watch, from live listings
- jewellery market research — auction results and retail listings for a piece
- diamond stock search — search a supplier's own loose-diamond stock list against
  criteria the valuer sets

**This is a starting point for research, not a valuation.** The tool never produces a
value, and nothing it outputs goes into a client document unaltered. The valuer's own
appraisal — inspection, condition, provenance, purpose of valuation — happens separately
and afterwards.

State this explicitly on the homepage (`index.html`) — that's the one place every valuer
is guaranteed to land before opening any tool. Individual tool pages don't need to repeat
the caveat in their own copy; saying it once, prominently, beats saying it quietly on
every page. Still never describe what a tool itself produces as a "valuation" anywhere —
that's a separate, permanent rule, independent of where the caveat sentence lives.

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
- `diamond-research.html` — diamond stock search tool (see dedicated section below)
- `vendor/xlsx.mini.min.js` — vendored copy of SheetJS, used only by the diamond tool
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

No build step, no framework. Plain HTML, CSS and vanilla JavaScript, deliberately, so it
stays maintainable after the internship ends. The one dependency is SheetJS, vendored at
`vendor/xlsx.mini.min.js` for the diamond tool's `.xlsx` reading — see below for why that
one exception exists. Don't add anything else without discussing it first.

## Diamond tool: how it's built and why

The tool loads a supplier's stock list (an `.xlsx` file, or `.csv`) entirely in the
browser and searches it locally. **Nothing is ever uploaded anywhere** — no fetch, no
server, no analytics on the file contents. This matters because supplier stock lists are
commercially sensitive; the whole point of doing this client-side is that the data never
leaves the valuer's machine. Keep it that way in any future change.

**Why there's a vendored library here despite the "no dependencies" rule elsewhere:**
`.xlsx` is a zip file full of XML — genuinely not something vanilla JavaScript can unzip
and parse unassisted. `.csv` and the pre-filtered URL approach used by the other two tools
don't have this problem, which is why they have no dependency. SheetJS is downloaded once
and committed into `vendor/`, not linked from a CDN, so the tool keeps working even if
SheetJS's own site goes down — there's still no build step and no `npm install`.

**Each supplier gets its own config, not a shared parser**, because Gemex, Josyfon and
Weissbart GIA each use different column names, different column order, and different
amounts of junk (title rows, blank rows) above their real data. The `SUPPLIERS` array in
`diamond-research.html` holds one entry per supplier: which column headers to look for,
and which of those map to shape/weight/colour/clarity/lab/price. Adding a new supplier
means adding one more entry, not touching the shared logic.

**The parser finds the header row by name, not by row number.** It scans the first 30
rows of the sheet for the first one containing all of a supplier's expected column names,
then reads columns by matching header text rather than a fixed position. This is what
lets Josyfon have ten blank/title rows above its header, or Weissbart have stray
"ROUND"-style section-divider rows and a repeated title row mid-sheet, without the parser
needing to know in advance how many junk rows there are — a row only becomes a diamond
if its weight column parses to a real number; anything else (a divider, a blank row, a
title) is silently skipped.

**Diamonds For Today is deliberately not included.** Their list is only available as a
PDF, and that PDF's text layer uses a non-standard font encoding — extracting text from it
produced garbled nonsense, not real values, when tested. Silently returning wrong diamond
data would be worse than not supporting the source at all. If Diamonds For Today can
provide a CSV or Excel export instead, add them as a fourth `SUPPLIERS` entry the same way
as the other three; don't attempt automated PDF parsing for this source.

**Colour and clarity are checkbox-dropdown multi-selects**, built from `COLOUR_SCALE` /
`CLARITY_SCALE`, not text inputs — a valuer ticks the grades they'll accept (e.g. D, E,
F) and a stone matches if its own grade is one of those ticked. Fancy colours aren't in
the list, so they're simply not selectable, but still appear in results whenever the
colour filter is left on "Any".

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
  Export the CSV instead. This includes the diamond tool: a loaded stock list lives only
  in memory for that page load and is gone on reload, same as everything else.
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
- [x] Diamond stock search tool built (`diamond-research.html`). Column mappings for
      Gemex, Josyfon and Weissbart GIA are based on one real sample file per supplier —
      re-check against a second sample from each before relying on it for real work, in
      case a supplier's layout varies file to file.
- [ ] Get a CSV or Excel export from Diamonds For Today (their PDF isn't parseable — see
      the diamond tool section above) and add them as a fourth `SUPPLIERS` entry
- [ ] Decide whether the homepage should keep a "coming soon" card for anything else

## Context to keep in mind

Hugh is an undergraduate, not a professional developer, and needs to be able to explain
how everything works. Prefer clear, commented, boring code over clever or compact code.
Explain changes rather than just making them.
