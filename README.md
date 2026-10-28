# Ward 53 voter analytics · Tirupati

React + Vite static site for the 4,158 voters on the Ward 53 roll (Tirupati
assembly constituency 167), with family trees worked out from the roll.
Read-only, mobile-first, light and dark themes.

## Pages

| Page | What it gives you |
|------|-------------------|
| **Overview** | Totals, gender split, households, families, first-time / youth / senior counts; one-click cards for every ward part, polling part and block; search box. |
| **Voters** | The whole list as a sortable table or cards. Search (Telugu or English spelling, voter ID, door no., serial no., several IDs or a serial range at once) plus filters with live counts. CSV download, print, "Analyse these". |
| **Voter** | Every attribute, copyable voter ID, family members labelled by relation (father, sister-in-law, grandson…), others at the same door no. Previous / next through the current results. |
| **Families** | Every family, largest first; search by any member; filter by block and size. |
| **Family tree** | The tree drawn generation by generation. Tap anyone to relabel everyone relative to them. Relatives who are not voters here are shown dashed; uncertain links carry a ⚠ with the reason. |
| **Houses / House** | Voters grouped by door number, then by family within the house. |
| **Analysis** | Age-gender pyramid, voters by single year of age, a breakdown table (group by ward part, polling part, block, age group, relation, family size, voter ID series), block bars, ward part × polling part matrix, family and household sizes, link quality, data checks, largest families and households. It analyses the same voters the list is filtered to; every bar, cell and row opens those voters. |

### Attributes used (the ward is the same for everyone)

- **Ward part** – the five printed parts of the roll (`Part_No` 53_1 … 53_5), with the PDF page.
- **Polling part** – the assembly-constituency part number and serial printed against each voter (`ac_part_no`, `ac_serial_no`); parts with fewer than 50 voters are grouped as "Other parts".
- **Block** – the street block taken from the door number: `20-48-S10-755` → `20-48-S10`, `20-3-15/A2` → `20-3`. Common OCR slips are repaired (`20-48-58-516`, `20-48-S10662`, `2048-S9-609`). Blocks under 15 voters are grouped as "Other blocks" in charts but stay selectable one by one.
- **Household** – the same door number however it was typed (`D NO 20-3-17/B5,` = `20-3-17-B5`).
- **Family** – from csv-family-tree: each voter's printed father, husband or mother matched by name to a voter at the same house or building, with age and gender checks.
- **Voter ID series** – the first three letters of the voter ID card number.
- **Data checks** – house no. not printed, extra (English) text in a name, family links that need checking.

## Updating the data

The app reads `src/data/voters.json`, made from the CSV and the family graph:

```
cd ../csv-family-tree
python3 make_family_tree.py combined_voter_list_ward_53.csv --out family_out --place "Tirupati"
cd ../praveen_tpty
npm run data      # = python3 scripts/prepare_data.py
```

`prepare_data.py` checks that the CSV and the family graph line up row for row
and stops if they do not (re-run `make_family_tree.py` after any CSV change).
It also removes the `�` marks OCR left in 160 names. Other paths:
`python3 scripts/prepare_data.py --csv <file> --graph <family_graph.json>`.

## Sign-in

The site asks for a user id and password. At build time the voter list is
encrypted with them (AES-GCM, key from PBKDF2), so the published site holds no
readable voter data and no password. The user id is not case-sensitive; the
password is. They come from two environment variables, `VOTER_USER` and
`VOTER_PASS` (a local `.env` file, never committed, or the host's settings).

**Keep this repository private.** `src/data/voters.json` is plain text, so
anyone who can read the repo can read the data, whatever the sign-in.

## Run on your laptop

```
cp .env.example .env   # then fill in VOTER_USER and VOTER_PASS
npm install
npm run dev            # http://localhost:5173
```

Sign-in needs a secure page (https or localhost); opening the dev server from a
phone by IP address will not sign in.

## Deploy on Render

1. Push this folder to a **private** GitHub repository.
2. On render.com: **New → Blueprint**, pick the repo, enter `VOTER_USER` and
   `VOTER_PASS`, click **Apply** (it reads `render.yaml`). Or **New → Static
   Site** with build command `npm ci && npm run build`, publish directory
   `dist`, and the two variables under **Environment**.
3. To change the password, edit `VOTER_PASS` and redeploy; everyone signs in again.

## Settings and extending

`src/config.js`: `SHOW_VOTER_ID` (show, search and export voter ID numbers;
default `true`), page sizes, and the thresholds for grouping small blocks,
polling parts and ID series.

- **New filter:** add a key to `EMPTY` and an entry to `FACETS` in
  `src/lib/filters.js`. The filter panel, its live counts, the removable chips
  and the analysis scope all pick it up.
- **New breakdown:** add an entry to `DIMS` in `src/components/Analysis.jsx`.
- **New voter attribute:** add the column in `scripts/prepare_data.py`
  (`COLS` and the row), then read it in `init()` in `src/lib/data.js`.
- **Relationship words:** `relationTo()` in `src/lib/kin.js`.
