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
self-contained HTML file that opens from `file://`, survives being emailed, and needs no
backend, network, or browser storage.

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
and put the disagreement in each row's `note`. **Never pick a winner.** Every such
conflict, and every flagged assumption from the sweep, belongs in `note` so it lands in
the artifact — not only in the chat.

## Step 4 — Score materiality against the approved summary

Score each entry against **the approved issues**, not against how dramatic it sounds.
Set `key: true` only where the entry does real work on a live issue. Every key entry
needs a `keyReason` of **15 words or fewer** stating what work the fact does —
"decisive clause on the central acceptance issue", not "this is about acceptance".

Sanity check: if **well over a third** of entries come out as key, the scoring is too
loose. Re-examine the middle band and demote — do **not** move the threshold.

## Step 5 — Build the artifact

Copy the **Template** at the bottom of this file verbatim into a new `.html` file, then
replace the three placeholders — `__TITLE__`, `__META__`, and `__DATA__` — with the case
data. `__META__` and `__DATA__` are JavaScript literals (no surrounding quotes). Copy the
template exactly; do not rewrite it from memory — the search, filters, sorting, and the
expandable source detail are already wired and tested, and only the data changes.

```js
const META = {
  title: "Ambleside Foods v Trentham Engineering — Coalville Line 3",
  documents: 5,                    // optional; falls back to a count of distinct sources
  prepared: "12 August 2026"       // optional; falls back to today's date
};
const ROWS = [
  {
    date: "2025-02-06",              // ISO, sortable
    certainty: "exact",              // "exact" | "derived" | "uncertain" — non-exact shows a ▲ marker
    dateNote: "Date relative: 'the same week' as PO receipt on 6 Feb 2025 (w/c 3 Feb).", // shown in the expanded detail
    description: "Trentham releases the long-lead items.",          // one neutral sentence: who did what
    party: "Trentham Engineering",   // actor — populates the Party filter
    issue: "Delay",                  // issue tag — populates the Issue filter
    source: "Email — S. Ruddock to N. Okafor, 4 Mar 2025, p.1",     // title + page; the clickable link
    passage: "We received your purchase order on 6 February 2025 and released the long-lead items the same week.", // verbatim, shown on expand
    key: false,
    keyReason: "",                   // ≤15 words, required when key is true
    note: "Trentham's letter of 2 Apr 2025 dates the PO 10 Feb 2025 — conflict."          // conflict / flagged assumption; shows inline under the description
  }
];
```

Field rules: **Date** is ISO. Set `certainty` to `derived` or `uncertain` for any date
you resolved or are unsure of — the table shows a marker (▲ derived, △ uncertain) and
the resolution you wrote in `dateNote` appears in the expandable detail. **Description**
is one neutral sentence naming who did what. **Source** is document title and page; it
is an underlined link that expands the verbatim `passage` and the date derivation
beneath the row. **Key** is Yes/No with the visible `keyReason`. Put every date conflict
and flagged assumption in `note` — it appears inline beneath the description with a ⚑
marker, so conflicts are visible in the table itself, not only in the chat.

The columns, in order, are **Date | Description | Source | Key**. The page provides global
search across all fields, a party filter, an issue filter, a key-only toggle, sortable
columns, and click-to-expand source passages. It is styled in a clean beige-and-white,
Times New Roman print aesthetic. There are **no file-download buttons** — they are blocked
inside the Artifact sandbox; if the user needs a hard copy, tell them to use the browser's
own print / save-to-PDF on the open file.

Save the finished file (e.g. `chronology.html`) and give it to the user. On Cowork,
send it with `SendUserFile`; on Claude Code, write it to the working directory and tell
them the path; on Claude Chat, present it as a downloadable file.

## Install

This skill is a single self-contained `SKILL.md` — the HTML template lives at the bottom
of this file, so there is nothing else to ship. **Claude Chat:** upload `SKILL.md` as a
skill in Settings → Capabilities, or attach it to a project. **Cowork:** drop it into the
workspace's skills directory (or your synced skills) and it loads on match. **Claude
Code:** place it at `~/.claude/skills/chronology/SKILL.md` (personal) or
`.claude/skills/chronology/SKILL.md` in the repo (project). No dependencies, no build
step, no network — one plain-text file that works identically on all three surfaces.

## Template

Copy everything inside the fence into a `.html` file and fill the three placeholders.

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>__TITLE__ — Chronology</title>
<style>
  :root {
    --bg:#faf8f3; --panel:#ffffff; --ink:#1c1a17; --mut:#7c7264; --line:#e6e0d4;
    --head:#efeae0; --detail:#f4f1ea; --hover:#f7f4ed; --rust:#a4592a; --flag:#8f5330;
    --link:#20508f; --quote:#cdc4b2;
  }
  * { box-sizing:border-box; }
  body { margin:0; font:16px/1.5 "Times New Roman", Times, Georgia, serif; color:var(--ink); background:var(--bg); }
  header { padding:30px 32px 6px; }
  h1 { margin:0 0 5px; font-size:26px; font-weight:700; letter-spacing:-.01em; }
  .metaline { color:var(--mut); font-size:15px; margin:0; }
  .controls { display:flex; flex-wrap:wrap; gap:12px; align-items:center; padding:18px 32px 6px; }
  .controls input[type=search], .controls select { font:inherit; font-size:15px; padding:8px 11px; border:1px solid var(--line); border-radius:5px; background:var(--panel); color:var(--ink); }
  .controls input[type=search] { flex:1; min-width:230px; }
  .controls input[type=search]:focus, .controls select:focus { outline:none; border-color:var(--rust); box-shadow:0 0 0 2px rgba(164,89,42,.14); }
  .chk { display:inline-flex; gap:7px; align-items:center; font-size:15px; color:var(--ink); cursor:pointer; user-select:none; }
  .chk input { accent-color:var(--rust); width:15px; height:15px; }
  .count { margin-left:auto; font-size:14px; color:var(--mut); white-space:nowrap; }
  .wrap { overflow-x:auto; padding:14px 32px 44px; }
  table { border-collapse:collapse; width:100%; min-width:680px; background:var(--panel); border:1px solid var(--line); }
  thead th { background:var(--head); text-align:left; font-size:12px; font-weight:700; letter-spacing:.09em; text-transform:uppercase; color:var(--mut); padding:12px 16px; border-bottom:1px solid var(--line); cursor:pointer; white-space:nowrap; position:sticky; top:0; z-index:1; }
  thead th:hover { color:var(--rust); }
  th .arrow { font-size:11px; color:var(--rust); }
  th[aria-sort=none] .arrow::after { content:" ↕"; color:var(--mut); opacity:.5; }
  th[aria-sort=ascending] .arrow::after { content:" ↑"; }
  th[aria-sort=descending] .arrow::after { content:" ↓"; }
  td { padding:13px 16px; border-bottom:1px solid var(--line); vertical-align:top; }
  tbody tr.row:hover > td { background:var(--hover); }
  td.date { white-space:nowrap; }
  .mark { color:var(--rust); cursor:help; margin-left:3px; }
  .flag { display:block; margin-top:7px; color:var(--flag); font-size:15px; }
  .flag .fi { margin-right:5px; }
  a.src { color:var(--link); text-decoration:underline; text-underline-offset:2px; cursor:pointer; }
  a.src:hover { color:var(--rust); }
  td.key { color:var(--mut); }
  td.key .yes { color:var(--ink); font-weight:700; }
  td.key .reason { display:block; color:var(--mut); font-size:14px; margin-top:3px; max-width:34ch; }
  tr.detail > td { background:var(--detail); }
  .doc { font-weight:700; margin-bottom:7px; }
  blockquote { margin:0 0 7px; padding-left:14px; border-left:2px solid var(--quote); color:#3a352d; }
  .deriv { color:var(--flag); font-size:15px; }
  @media print {
    .controls, .count, th .arrow { display:none !important; }
    header { padding:0 0 8px; } body { font-size:11px; background:#fff; }
    .wrap { overflow:visible; padding:0; } table { min-width:0; border:none; }
    tr.detail > td { background:#f4f1ea; -webkit-print-color-adjust:exact; print-color-adjust:exact; }
    thead th { position:static; background:#efeae0; -webkit-print-color-adjust:exact; print-color-adjust:exact; }
    td, th { padding:6px 9px; } tr { break-inside:avoid; }
  }
</style>
</head>
<body>
<header>
  <h1 id="ttl"></h1>
  <p class="metaline" id="meta"></p>
</header>
<div class="controls">
  <input id="q" type="search" placeholder="Search all fields…" aria-label="Search">
  <select id="party" aria-label="Party"><option value="">All parties</option></select>
  <select id="issue" aria-label="Issue"><option value="">All issues</option></select>
  <label class="chk"><input type="checkbox" id="keyonly"> Key only</label>
  <label class="chk"><input type="checkbox" id="flagonly"> Flagged only</label>
  <span class="count" id="cnt"></span>
</div>
<div class="wrap">
<table>
  <thead><tr>
    <th data-k="date">Date<span class="arrow"></span></th>
    <th data-k="description">Description<span class="arrow"></span></th>
    <th data-k="source">Source<span class="arrow"></span></th>
    <th data-k="key">Key<span class="arrow"></span></th>
  </tr></thead>
  <tbody id="rows"></tbody>
</table>
</div>

<script>
// ---- Data injected by the skill (replace the two placeholders) --------------
const META = __META__;
const ROWS = __DATA__;
// -----------------------------------------------------------------------------
const $ = s => document.querySelector(s);
const esc = s => String(s==null?"":s).replace(/[&<>"]/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c]));
ROWS.forEach((r,i)=>r._i=i);
let sortK="date", sortDir=1, open=new Set();

$("#ttl").textContent = META.title || "Chronology";
const prepared = META.prepared || new Date().toLocaleDateString("en-GB",{day:"numeric",month:"long",year:"numeric"});
const docs = META.documents ?? new Set(ROWS.map(r=>String(r.source||"").replace(/,?\s*p\.?\s*\d.*$/i,"").trim()).filter(Boolean)).size;
const plur=(n,w)=>`${n} ${w}${n===1?"":"s"}`;
$("#meta").textContent = `${plur(ROWS.length,"entry").replace("entrys","entries")} · ${plur(docs,"document")} · prepared ${prepared}`;
document.title = (META.title||"Case") + " — Chronology";
fill("#party","party"); fill("#issue","issue");
function fill(sel,k){ const vals=[...new Set(ROWS.map(r=>r[k]).filter(Boolean))].sort();
  vals.forEach(v=>{ const o=document.createElement("option"); o.value=o.textContent=v; $(sel).append(o); }); }

function filtered(){
  const q=$("#q").value.toLowerCase(), ko=$("#keyonly").checked, fo=$("#flagonly").checked, p=$("#party").value, is=$("#issue").value;
  return ROWS.filter(r=>{
    if(ko && !r.key) return false;
    if(fo && !r.note) return false;
    if(p && r.party!==p) return false;
    if(is && r.issue!==is) return false;
    if(q){ const hay=[r.date,r.dateNote,r.description,r.source,r.passage,r.party,r.issue,r.keyReason,r.note].join(" ").toLowerCase();
      if(!hay.includes(q)) return false; }
    return true;
  }).sort((a,b)=>{ const x=(a[sortK]??"")+"", y=(b[sortK]??"")+""; return x<y?-sortDir:x>y?sortDir:a._i-b._i; });
}

function render(){
  const list=filtered(), tb=$("#rows"); tb.innerHTML="";
  document.querySelectorAll("th").forEach(th=>th.setAttribute("aria-sort", th.dataset.k===sortK?(sortDir>0?"ascending":"descending"):"none"));
  list.forEach(r=>{
    const hasDetail = r.passage || r.dateNote;
    const mark = r.certainty && r.certainty!=="exact" ? `<span class="mark" title="${esc(r.certainty)} date — see source detail">${r.certainty==="uncertain"?"△":"▲"}</span>` : "";
    const src = hasDetail ? `<a class="src" data-i="${r._i}">${esc(r.source)}</a>` : esc(r.source);
    const flag = r.note ? `<span class="flag"><span class="fi">⚑</span>${esc(r.note)}</span>` : "";
    const key = r.key ? `<span class="yes">Yes</span><span class="reason">${esc(r.keyReason||"")}</span>` : "No";
    const tr=document.createElement("tr"); tr.className="row";
    tr.innerHTML =
      `<td class="date">${esc(r.date)}${mark}</td>`+
      `<td class="desc">${esc(r.description)}${flag}</td>`+
      `<td class="src">${src}</td>`+
      `<td class="key">${key}</td>`;
    tb.append(tr);
    if(hasDetail && open.has(r._i)){
      const d=document.createElement("tr"); d.className="detail";
      let h=`<div class="doc">${esc(r.source)}</div>`;
      if(r.passage) h+=`<blockquote>${esc(r.passage)}</blockquote>`;
      if(r.dateNote) h+=`<div class="deriv">${esc(r.dateNote)}</div>`;
      d.innerHTML=`<td colspan="4">${h}</td>`; tb.append(d);
    }
  });
  const kc=ROWS.filter(r=>r.key).length, fc=ROWS.filter(r=>r.note).length;
  $("#cnt").textContent = `${list.length} shown · ${kc} key · ${fc} flagged`;
}

document.addEventListener("click",e=>{
  const a=e.target.closest("a.src"); if(a){ e.preventDefault(); const i=+a.dataset.i; open.has(i)?open.delete(i):open.add(i); render(); }
});
document.querySelectorAll("th").forEach(th=>th.addEventListener("click",()=>{
  const k=th.dataset.k; if(k===sortK) sortDir=-sortDir; else { sortK=k; sortDir=1; } render();
}));
["input","change"].forEach(ev=>["#q","#keyonly","#flagonly","#party","#issue"].forEach(s=>$(s).addEventListener(ev,render)));
render();
</script>
</body>
</html>
```
