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
template exactly; do not rewrite it from memory — the search, filters, sorting, row
shading, and expand are already wired and tested, and only the data changes.

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
    note: "Defence dates this 5 Oct — see row 24"                   // conflict / flagged assumption; shades the row
  }
];
```

Field rules: **Date** is ISO; set `certainty` to `derived` or `uncertain` for any date
you resolved or are unsure of — the table shows a marker (`*` derived, `~` uncertain)
and reveals `dateOriginal`/`dateNote` on expand. **Description** is one neutral sentence
naming who did what. **Source** is document title and page; put the verbatim quote in
`passage`. **Key** is yes/no with the visible `keyReason`. Put every date conflict and
flagged assumption in `note` — it appears in its own **Notes** column and shades the
whole row light orange, so conflicts are impossible to miss.

The columns, in order, are **Date | Description | Source | Key | Notes**. The page
provides global search across all fields, key-only and flagged-only toggles, party and
issue filters, sortable columns, and expandable source passages and date derivations.
Key rows carry a purple accent; rows with a note are shaded orange. There are **no
file-download buttons** — they are blocked inside the Artifact sandbox; if the user needs
a hard copy, tell them to use the browser's own print/save-to-PDF on the open file.

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
    --bg:#ffffff; --fg:#1e1b2e; --mut:#6b6785; --line:#e9e5f6; --band:#faf9ff;
    --brand:#7c3aed; --brand-d:#6d28d9; --brand-soft:#f5f2ff; --brand-tint:#efeaff;
    --warn:#c2410c; --warn-soft:#fff4e8; --shadow:0 1px 2px rgba(76,29,149,.06);
  }
  * { box-sizing:border-box; }
  body { margin:0; font:15px/1.55 -apple-system,Segoe UI,Roboto,Helvetica,Arial,sans-serif; color:var(--fg); background:var(--bg); }
  header { padding:26px 28px 14px; background:linear-gradient(180deg,var(--brand-soft),var(--bg)); border-bottom:1px solid var(--line); }
  .eyebrow { font-size:11px; letter-spacing:.18em; text-transform:uppercase; font-weight:700; color:var(--brand); margin:0 0 6px; }
  h1 { margin:0 0 6px; font-size:23px; letter-spacing:-.01em; }
  .summary { color:var(--mut); max-width:74ch; margin:0; }
  .controls { display:flex; flex-wrap:wrap; gap:10px; align-items:center; padding:16px 28px 8px; position:sticky; top:0; background:var(--bg); z-index:3; }
  .controls input, .controls select { font:inherit; padding:8px 11px; border:1px solid var(--line); border-radius:8px; background:var(--bg); color:var(--fg); box-shadow:var(--shadow); }
  .controls input:focus, .controls select:focus { outline:none; border-color:var(--brand); box-shadow:0 0 0 3px var(--brand-tint); }
  #q { flex:1; min-width:220px; }
  .toggle { display:inline-flex; gap:7px; align-items:center; padding:8px 12px; border:1px solid var(--line); border-radius:20px; font-size:14px; color:var(--mut); cursor:pointer; user-select:none; background:var(--bg); box-shadow:var(--shadow); }
  .toggle:hover { border-color:var(--brand); color:var(--brand-d); }
  .toggle input { accent-color:var(--brand); }
  .controls .sel { display:flex; gap:6px; align-items:center; color:var(--mut); font-size:14px; }
  .count { margin-left:auto; font-size:13px; font-weight:600; color:var(--brand-d); background:var(--brand-soft); border:1px solid var(--line); padding:6px 12px; border-radius:20px; white-space:nowrap; }
  .legend { display:flex; gap:18px; padding:2px 28px 14px; font-size:12.5px; color:var(--mut); flex-wrap:wrap; }
  .legend span { display:inline-flex; gap:7px; align-items:center; }
  .swatch { width:22px; height:13px; border-radius:4px; border:1px solid var(--line); }
  .sw-key { background:var(--brand-soft); box-shadow:inset 3px 0 0 var(--brand); }
  .sw-flag { background:var(--warn-soft); }
  .wrap { overflow-x:auto; }
  table { border-collapse:collapse; width:100%; min-width:720px; }
  th, td { text-align:left; padding:11px 16px; border-bottom:1px solid var(--line); vertical-align:top; }
  th { position:sticky; top:64px; background:var(--bg); cursor:pointer; user-select:none; font-size:12px; letter-spacing:.04em; text-transform:uppercase; color:var(--mut); white-space:nowrap; z-index:2; }
  th:hover { color:var(--brand-d); }
  th[aria-sort] .arrow::after { content:" ↕"; opacity:.3; }
  th[aria-sort="ascending"] .arrow::after { content:" ↑"; opacity:1; color:var(--brand); }
  th[aria-sort="descending"] .arrow::after { content:" ↓"; opacity:1; color:var(--brand); }
  tbody tr:hover td { background:var(--band); }
  tr.key td { background:var(--brand-soft); }
  tr.key td:first-child { box-shadow:inset 3px 0 0 var(--brand); }
  tr.flag td { background:var(--warn-soft); }
  tr.key.flag td:first-child { box-shadow:inset 3px 0 0 var(--brand); }
  td.date { white-space:nowrap; font-variant-numeric:tabular-nums; font-weight:500; }
  .flagmark, .datemark { cursor:help; font-weight:700; }
  .datemark { color:var(--brand); }
  .exp { cursor:pointer; color:var(--brand); border:none; background:none; padding:0 6px 0 0; font-size:13px; }
  .src { color:var(--mut); font-size:14px; }
  .keyreason { color:var(--brand-d); font-size:13px; margin-top:3px; }
  .badge { display:inline-block; font-size:11px; font-weight:700; letter-spacing:.03em; padding:2px 9px; border-radius:20px; }
  .yes { background:var(--brand); color:#fff; } .no { color:var(--mut); font-weight:500; }
  td.notecell { color:var(--warn); font-size:13.5px; }
  .flagmark { color:var(--warn); margin-right:4px; }
  tr.detail td { background:var(--band); font-size:14px; color:#3a3550; }
  tr.detail dl { margin:0; display:grid; grid-template-columns:max-content 1fr; gap:4px 16px; }
  tr.detail dt { color:var(--brand-d); font-weight:600; } tr.detail dd { margin:0; }
  blockquote { margin:2px 0; padding-left:12px; border-left:3px solid var(--brand-tint); color:#3a3550; }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg:#17151f; --fg:#ece9f5; --mut:#a29dbc; --line:#312b45; --band:#1e1b2a;
      --brand:#a78bfa; --brand-d:#c4b5fd; --brand-soft:#241f38; --brand-tint:#2f2850;
      --warn:#fdba74; --warn-soft:#2e2114; --shadow:none;
    }
    .yes { color:#17151f; } td.notecell { color:var(--warn); }
  }
  @media print {
    .controls, .legend, .exp, thead th .arrow, .count { display:none !important; }
    header { padding:0 0 10px; background:none; } body { font-size:11px; }
    tr.detail { display:none; } th { position:static; }
    .wrap { overflow:visible; } table { min-width:0; }
    tr.key td, tr.flag td { -webkit-print-color-adjust:exact; print-color-adjust:exact; }
    td, th { padding:5px 8px; }
  }
</style>
</head>
<body>
<header>
  <p class="eyebrow">Chronology</p>
  <h1 id="ttl"></h1>
  <p class="summary" id="sum"></p>
</header>
<div class="controls">
  <input id="q" type="search" placeholder="Search all fields…" aria-label="Search">
  <label class="toggle"><input type="checkbox" id="keyonly"> Key only</label>
  <label class="toggle"><input type="checkbox" id="flagonly"> Flagged only</label>
  <span class="sel">Party <select id="party"><option value="">All</option></select></span>
  <span class="sel">Issue <select id="issue"><option value="">All</option></select></span>
  <span class="count" id="cnt"></span>
</div>
<div class="legend">
  <span><i class="swatch sw-key"></i> Key entry</span>
  <span><i class="swatch sw-flag"></i> Flagged — date conflict or assumption (see Notes)</span>
  <span><b class="datemark">*</b>&nbsp;derived date · <b class="datemark">~</b>&nbsp;uncertain</span>
</div>
<div class="wrap">
<table>
  <thead><tr>
    <th data-k="date">Date<span class="arrow"></span></th>
    <th data-k="description">Description<span class="arrow"></span></th>
    <th data-k="source">Source<span class="arrow"></span></th>
    <th data-k="key">Key<span class="arrow"></span></th>
    <th data-k="note">Notes<span class="arrow"></span></th>
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
$("#sum").textContent = META.summary || "";
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
    if(q){ const hay=[r.date,r.dateOriginal,r.description,r.source,r.passage,r.party,r.issue,r.keyReason,r.note].join(" ").toLowerCase();
      if(!hay.includes(q)) return false; }
    return true;
  }).sort((a,b)=>{ const x=(a[sortK]??"")+"", y=(b[sortK]??"")+""; return x<y?-sortDir:x>y?sortDir:a._i-b._i; });
}

function render(){
  const list=filtered(), tb=$("#rows"); tb.innerHTML="";
  document.querySelectorAll("th").forEach(th=>th.setAttribute("aria-sort", th.dataset.k===sortK?(sortDir>0?"ascending":"descending"):"none"));
  list.forEach(r=>{
    const hasDetail = r.dateOriginal || r.dateNote || r.passage;
    const mark = r.certainty && r.certainty!=="exact" ? `<span class="datemark" title="${esc(r.certainty)}: ${esc(r.dateNote||r.dateOriginal||"")}">${r.certainty==="uncertain"?"~":"*"}</span> ` : "";
    const tr=document.createElement("tr"); tr.className=[r.key?"key":"", r.note?"flag":""].filter(Boolean).join(" ");
    tr.innerHTML =
      `<td class="date">${hasDetail?`<button class="exp" data-i="${r._i}" aria-label="Toggle detail">${open.has(r._i)?"▾":"▸"}</button>`:""}${mark}${esc(r.date)}</td>`+
      `<td>${esc(r.description)}</td>`+
      `<td class="src">${esc(r.source)}</td>`+
      `<td>${r.key?`<span class="badge yes">KEY</span><div class="keyreason">${esc(r.keyReason||"")}</div>`:`<span class="badge no">no</span>`}</td>`+
      `<td class="notecell">${r.note?`<span class="flagmark" title="Flagged">⚠</span>${esc(r.note)}`:""}</td>`;
    tb.append(tr);
    if(hasDetail && open.has(r._i)){
      const d=document.createElement("tr"); d.className="detail";
      let dl="<dl>";
      if(r.dateOriginal) dl+=`<dt>Original</dt><dd>${esc(r.dateOriginal)}</dd>`;
      if(r.dateNote) dl+=`<dt>Resolved</dt><dd>${esc(r.dateNote)}</dd>`;
      if(r.passage) dl+=`<dt>Passage</dt><dd><blockquote>${esc(r.passage)}</blockquote></dd>`;
      d.innerHTML=`<td colspan="5">${dl}</dl></td>`; tb.append(d);
    }
  });
  const kc=ROWS.filter(r=>r.key).length, fc=ROWS.filter(r=>r.note).length;
  $("#cnt").textContent = `${list.length} of ${ROWS.length} · ${kc} key · ${fc} flagged`;
}

document.addEventListener("click",e=>{
  const ex=e.target.closest(".exp"); if(ex){ const i=+ex.dataset.i; open.has(i)?open.delete(i):open.add(i); render(); }
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
