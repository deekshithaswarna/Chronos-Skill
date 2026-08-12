---
name: chronology
description: >-
  Turn uploaded case documents into a cited, filterable litigation chronology — a
  sortable, searchable HTML timeline where every entry links to its source passage.
  Use this whenever the user wants a chronology, timeline, or sequence of events; asks
  to "put these in date order", "work out what happened when", or "build a timeline of
  the case"; or hands over a document bundle (pleadings, disclosure, exhibits, witness
  statements, correspondence, contracts, attendance notes, invoices) and wants the
  events extracted and dated. Trigger on the document-bundle language even when the
  word "chronology" is never used.
---

# Chronology

Build a cited, filterable litigation chronology from case documents. The output is one
self-contained HTML file (from `assets/chronology.html`) that opens from `file://`,
survives being emailed, and needs no backend, network, or browser storage.

Work through the steps **in order**. Each of the first two ends with a hard stop — do
not run ahead. The value of this skill is accuracy and completeness, not speed.

## Step 1 — Ask for the documents, then stop

Do **not** build a chronology from whatever is already in context, and do **not** work
from a description of documents. Ask the user to upload the actual files:

> Upload the documents you want in the chronology. Anything dated helps — pleadings,
> witness statements, correspondence, contracts, attendance notes, invoices, orders,
> emails. Scans and PDFs are fine; I'll read them.

Then **stop and wait** for a reply. Nothing below happens this turn.

## Step 2 — Read, draft a provisional summary, get approval, then stop

Read **every** document in full. Then write a short provisional case summary in plain
prose (no headings, no bullets): one paragraph covering **what the dispute is**, **the
parties and their roles**, and **the live issues framed as questions** (e.g. "Was the
software accepted before 30 September?"). Keep it tight.

Present it, then say plainly:

> Materiality is scored against this summary, so correcting it now is cheap — a
> correction after extraction means re-scoring every entry. Does this read right?

**Stop and wait for approval. Do not extract in the same turn.** If the user waves it
through without engaging, proceed — but say once, in the final output, that the summary
was not confirmed. The issues you settle on here become the filter list in Step 4.

## Step 3 — Extract every date

The chronology must capture **every date in the documents**. This is the core promise;
completeness matters more than tidiness. Work through each document twice.

**Pass 1 — the date sweep.** Go through the document top to bottom against this
checklist, and create a row for each hit:

- **Explicit full dates** — record as-is.
- **Partial dates** ("March 2023", "2019") — normalise to the **first** of the period
  (`2023-03-01`, `2019-01-01`) and record that you did in the date note.
- **Relative dates** ("the following day", "two weeks later", "shortly after go-live")
  — resolve against the nearest anchor **in the same document** and record the
  derivation.
- **Computed dates** ("thirty days from receipt", "ten working days after delivery") —
  record the computation; **flag it** if the premise is one party's assertion rather
  than an agreed fact.
- **Multiple dates in one sentence or paragraph** — each gets **its own row**.
- **Dates inside quoted or attached material** (a letter quoting an earlier letter) —
  extract them too.
- **Event date vs document date** — date the row by the event. A 12 October minute
  describing a 30 September missed deadline is dated **30 September**.
- **Recurring or period dates** (cure periods, limitation windows) — record start and
  end.
- **Events clearly dated relative to something undated** — keep them, with the date
  marked **uncertain**, rather than dropping them.

**Pass 2 — the miss sweep.** A second pass over the same document whose only job is to
find dates Pass 1 missed. For each document, **state the count**: "Doc 3 — pass 1: 14
dates; pass 2: +2".

Where two documents give **different dates for the same event**, record **both rows**
and note the disagreement in each row's `note`. **Never pick a winner.**

## Step 4 — Score materiality against the approved summary

Score each entry against **the approved issues**, not against how dramatic it sounds.
Set `key: true` only where the entry does real work on a live issue. Every key entry
needs a `keyReason` of **15 words or fewer** stating what work the fact does —
"decisive clause on the central acceptance issue", not "this is about acceptance".

Sanity check: if **well over a third** of entries come out as key, the scoring is too
loose. Re-examine the middle band and demote — do **not** move the threshold.

## Step 5 — Build the artifact

Copy `assets/chronology.html` and replace the three placeholders — `__TITLE__`,
`__META__`, and `__DATA__` — with the case data. `__META__` and `__DATA__` are
JavaScript literals (no surrounding quotes). Everything else — table, search, filters,
sortable columns, expandable detail, and the three exports — is already wired.

```js
const META = { title: "Acme Ltd v Beta Corp", summary: "One-paragraph approved summary…" };
const ROWS = [
  {
    date: "2023-09-30",              // ISO, sortable
    certainty: "exact",              // "exact" | "derived" | "uncertain" — anything but exact shows a marker
    dateOriginal: "the deadline",    // the words as they appeared (for derived/uncertain)
    dateNote: "Resolved: 30 days from 31 Aug delivery per cl. 4",  // how it was resolved/computed
    description: "Beta rejected the software as non-conforming.",   // one neutral sentence: who did what
    party: "Beta Corp",              // actor — populates the Party filter
    issue: "Acceptance",             // issue tag — populates the Issue filter
    source: "Particulars of Claim, p.6",                            // title + page
    passage: "…the Claimant rejected the Software…",                // verbatim, shown on expand
    key: true,
    keyReason: "Rejection fixes the acceptance issue",              // ≤15 words, required when key
    note: "Defence dates this 5 Oct — see row 24"                   // conflicts / flagged assumptions
  }
];
```

Field rules: **Date** is ISO; set `certainty` to `derived` or `uncertain` for any date
you resolved or are unsure of — the table shows a marker (`*` derived, `~` uncertain)
and reveals `dateOriginal`/`dateNote` on expand. **Description** is one neutral sentence
naming who did what. **Source** is document title and page; put the verbatim quote in
`passage`. **Key** is yes/no with the visible `keyReason`. Put every date conflict and
flagged assumption in `note` — it surfaces in the row and is searchable.

The table columns are, in order: **Date | Description | Source | Key**. The page already
provides global search across all fields, a key-only filter, party and issue filters,
sortable columns, and exports to **Excel** (`.xls`, native), **Word** (`.doc`, native),
and **PDF** (via `window.print()` with a print stylesheet that hides the controls).

Save the finished file (e.g. `chronology.html`) and give it to the user. On Cowork,
send it with `SendUserFile`; on Claude Code, write it to the working directory and tell
them the path; on Claude Chat, present it as a downloadable file.

## Install

Put the `chronology/` folder (this `SKILL.md` plus `assets/chronology.html`) wherever
your surface loads skills. **Claude Chat:** zip the folder and upload it as a skill in
Settings → Capabilities, or attach the files to a project. **Cowork:** drop the folder
into the workspace's skills directory (or your synced skills) and it loads on match.
**Claude Code:** place it under `~/.claude/skills/chronology/` (personal) or
`.claude/skills/chronology/` in the repo (project). No dependencies, no build step, no
network — the skill is two plain-text files and works identically on all three.
