# Site
# VyaparSakhi
Built with AI Fiesta. React 19 + Vite + TypeScript + Tailwind v4 + React Router.

An AI accountant for kirana stores and small businesses. The owner uploads a
month of transactions and gets back, in plain English with the local flavour
intact: what every entry was, where the money went, what was routine and what
was not, a health score, and the few things that need attention this week.

The voice is a munim at the counter, not a bank. Every figure in the app is
sample data belonging to a fictional shop, **Sharma Kirana Store** (Karol Bagh,
Delhi), and is labelled as such wherever it appears on a page. Nothing here is a
claim about a real business.

## Setup
Requires Node 20+ and pnpm.

```sh
pnpm install
pnpm dev     # http://localhost:5173
pnpm build   # typecheck, then dist/
pnpm dev        # http://localhost:5173
pnpm build      # typecheck, then dist/
pnpm preview    # serve dist/ locally
pnpm typecheck  # tsc --noEmit, on its own
pnpm check:bundle
```

Everything under `src/` and `public/` is yours to edit. The build
configuration, the lockfile and `src/main.tsx` are locked so the site keeps
building and publishing.

There is no environment file, no database and no API key — the app runs entirely
in the browser. See **There is no backend** below.

There is no template here: no header, footer or navigation, no section blocks,
and no page beyond a placeholder home and a 404. `src/components/ui` holds
unstyled shadcn primitives; every page and every section is composed for this
site alone.

## Stack
React 19, Vite, TypeScript, Tailwind v4, shadcn/ui primitives on Radix, and
React Router. `vite.config.ts`, `tsconfig.json`, `index.html`, `package.json`,
the lockfile and `src/main.tsx` are locked so the build and the publisher keep
working; everything under `src/` and `public/` is editable.

## Routes
| Route | Page | What it is for |
| --- | --- | --- |
| `/` | Sahayak | The command centre — what needs action today: dues, attention, net flow. |
| `/overview` | Overview | Inflow, outflow and reserve; routine vs non-routine; category by category. |
| `/transactions` | Transactions | Every entry explained in one sentence, searchable and filterable. |
| `/cashflow` | CashFlow | Month-by-month bars, the recurring strip and the reserve trend. |
| `/budgets` | Budgets | A monthly limit per category, with progress and a warning when it is crossed. |
| `/goals` | Goals | Targets, saved so far, the monthly gap, and the month it lands. |
| `/enter` | Enter | Local workspace entry — shop name plus a PIN, kept in this browser only. |
| `*` | NotFound | The 404. The only route nobody asked for. |

Sahayak (`/`) is the only eagerly loaded page; every other route is lazy, so a
first visit carries the home screen alone.

## Project layout
```
src/
  App.tsx                 Routes + per-route <Meta>; everything but / renders in <Layout>
  theme.css               The token contract — the whole art direction
  fonts.css               The three faces, imported together
  index.css               Imports the above and sets the base layer
  data/
    ledger.ts             The ledger: transactions, totals, months, dues, flags,
                          insights, budgets, goals, and the inr()/lakh() formatters
    workspace.ts          Current upload summary + dueCount
  lib/workspace.ts        useWorkspace() — localStorage read/write, no server
  components/
    Shell.tsx             Rail, MobileBar, BottomTabs, UploadLedger, Footer, Layout
    Brand.tsx             The mark: rounded teal tile with rising bars + wordmark
    Icon.tsx              Lucide wrapper, one stroke width (1.6) set once
    Meta.tsx              Sets title, description and OG tags per route
    Section.tsx           Shell, Prose, PageHead, Panel, Money, Progress, StatusDot
    charts.tsx            Rings, bars, in/out chart and reserve line, drawn in code
    ui/                   Unstyled shadcn primitives — raw material, not a layout
  pages/                  One component per route
public/
  favicon.svg             Same rising bars, teal on a rounded square
  robots.txt
```

## How it works
**Data flow.** Every number on the site comes from `src/data/ledger.ts`. The
types there (`Txn`, `Category`, `Due`, `Flag`, `Budget`, `Goal`) are the shape a
real uploaded file would be normalised into — same field names, computed from
the owner's own rows instead of these. Pages import the aggregates they need and
render them; no page invents a figure.

**Shell.** The rail is the navigation on desktop (≥1024px): a 15rem sticky
column with the mark, the destinations and the upload foot. Below 1024px it
becomes a sticky top bar whose menu button opens a `Sheet` with a visible close.
Below 768px a fixed bottom tab bar carries five destinations plus "More", with
`pb-24` on `main` so nothing hides behind it. `Layout` declares the header, nav
and footer landmarks once; `/enter` renders outside it because a lock screen has
nothing to navigate past.

**The design is the interface.** There is no photography here. The subject is
the owner's own ledger, so every chart, ring, progress bar and ledger row is
drawn in code with divs and inline SVG, in the tokens in `src/theme.css`. Where
a picture would otherwise go there is either the chart that belongs there or an
empty state naming the space and what to do next.

**Token contract.** No component hardcodes a colour, a font stack, a radius or a
shadow. Change `--primary` in `src/theme.css` and every headline, the rail's
active pill, the dark work band and the rings follow. Marigold amber is the
single scarce accent and appears for exactly three things: money owed, something
that changed month-on-month, and the primary action. Success, warning and danger
exist only as state dots, bar segments, ring arcs and status words.

**Type.** Bricolage Grotesque for headings and figures, Public Sans for the
sentences that explain them, IBM Plex Mono for amounts, dates and file names —
`tabular-nums` on every money token is what makes a ledger column line up. Three
faces, loaded together from `src/fonts.css`; changing one means changing that
import and the matching `--font-*` value in `theme.css` together.

**Motion.** Bars grow from their baseline, the health ring draws itself over
320ms, panels lift 1px on hover. All of it is transform and opacity, all of it
is one-shot on mount, and all of it is disabled under
`prefers-reduced-motion: reduce`, handled globally in `src/index.css`.

## There is no backend
This is a front end with no server, no database and no authentication behind it.
That has two consequences worth stating plainly:

- **Uploads do not happen.** The app is built around reading a transactions
  file, and the file it reads is `src/data/ledger.ts`. A visitor's own upload is
  not parsed or transmitted, because there is nowhere to send it yet.
- **The PIN on `/enter` is a local lock, not authentication.** It is compared
  inside the tab and grants nothing, because there is nothing to grant. The
  workspace (shop name, file name, month) is written to this browser's
  `localStorage` under `vyaparsakhi.workspace` and is the whole of the
  persistence. Clearing site data resets it, and private mode means it will not
  survive a reload — the code catches that rather than throwing.

Server-side processing, real uploads and accounts are not built. Nothing on the
site pretends otherwise.

## Editing it
- **Change a figure or add a transaction** — `src/data/ledger.ts`. Pages follow.
- **Change the palette, type scale, spacing or motion** — `src/theme.css`, and
  update the matching paragraph in `DESIGN.md`.
- **Change a page's title or description** — the `<Meta>` for that route in
  `src/App.tsx`. Write them in the site's own voice, about what the page offers.
- **Add a route** — a component in `src/pages/`, an entry in `src/App.tsx`
  (lazy if it is not the home page), and a destination in `DESTINATIONS` in
  `src/components/Shell.tsx` if it belongs in the rail.

`DESIGN.md` is the brief behind all of the above: palette roles, the shell
decision, the brand mark, the imagery rule and the voice. Read it before
changing anything visual.

## What is verified
`pnpm build` typechecks and bundles. The publish check runs the same build plus
per-route SEO, accessibility, alt-text, colour-literal, font-count, link-text
and asset-budget checks, and screenshots the site at 360, 768 and 1440 pixels
wide — screenshots report layout overflow, axe violations and console errors,
not how the page looks.
