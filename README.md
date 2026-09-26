# Customer QR Site

A standalone repair-tracking page for a phone repair shop's customers.
Not tied to any one shop or backend by design -- any repair shop's
system can use this, as long as it can push rows into the two Sheets
below in the expected shape. Separate project, own git history, no
shared code with anything else. Meant to be hosted for free on GitHub
Pages.

## How it works

A customer scans the QR on their receipt, which opens this page with
their ticket's token in the address (`index.html?token=...`). The page
reads two Google Sheets directly in the browser -- no server of its own:

- **Repairs sheet** -- one row per ticket, kept up to date by whatever
  shop-management system the shop uses (in this deployment, DropFix).
  Looked up by matching Token. Its `Notes` column is optional and
  hidden entirely when blank -- when it's not, it's shown as a "Note
  from the shop" message on the ticket's page, the one place on this
  page that's genuinely staff-written text shown to the customer, not
  just status/fault/payment data.
- **Shop Details sheet** -- a small sheet kept up to date by the same
  shop-management system as the Repairs sheet (in this deployment,
  DropFix pushes it directly from Tools > Shop Details -- no Form
  involved). One fixed row, overwritten in place on every save, not
  appended to.

### Shop Details sheet -- exact columns the page reads

Column headers (case and spacing don't matter -- "Shop Name" and
"shop_name" both work -- but keep the same words):

| Column | Required? | Example |
|---|---|---|
| `shop_name` | yes | `Mobile Tech Pro LTD` |
| `address` | no -- cosmetic label only, shown above the maps link if set | `494 Hoe Street, London E17 9AH` |
| `maps_url` | no -- maps row is hidden entirely if blank | `https://maps.app.goo.gl/...` |
| `phones` | no -- phone rows hidden entirely if blank | `+44 7344 544184` |
| `email` | no -- email row hidden entirely if blank | `shop@example.com` |

`phones` supports any number of lines in one cell: each `Label:
Number` pair separated by `|` (a bare number with no `:` gets a
generic "WhatsApp" label) -- e.g. `Manager: +44 7550 722762 | Shop:
+44 7344 544184`. Today this deployment only ever pushes one number.

The collection-policy notice at the bottom of the page (`COLLECTION_POLICY`
in `index.html`) is fixed in this page's own code on purpose, not read
from either Sheet -- it's a policy decision, not day-to-day shop data.

Nothing else on the page is Sheet-driven -- logo, colours, section
labels ("What we're fixing", "Contact & location"), and layout are all
fixed in `index.html`, not editable through the Sheet.

Both sheets are read with Google's public "gviz" endpoint, which works
on any Sheet shared as "Anyone with the link can view" -- no API key,
no backend, no build step.

## Testing locally, before the real Sheets exist

`sample-data.json` stands in for both Sheets. `USE_SAMPLE_DATA` at the
top of `index.html` switches between it and the real thing -- once the
two Sheets are set up for real, flip it to `false` and fill in
`REPAIRS_SHEET_ID` and `SHOP_SHEET_ID`.

Try these once the page is running locally:

- `?token=qP9wE3rT6yUiO0987654` -- just received
- `?token=xR7mN2qL8yUwPz1234567` -- in progress
- `?token=aB3dK9xQ1WudKgHMgaZ9A` -- ready for collection
- `?token=mK4pV8nZ2QwTs5678901` -- collected
- no token, or an unrecognised one -- the "not found" message
