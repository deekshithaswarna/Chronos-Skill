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

Copy the **Template** at the bottom of this file verbatim into a new `.html` file, then
replace the three placeholders — `__TITLE__`, `__META__`, and `__DATA__` — with the case
data. `__META__` and `__DATA__` are JavaScript literals (no surrounding quotes). Copy the
template exactly; do not rewrite it from memory — the search, filters, sorting, expand,
print stylesheet, and exports are already wired and tested, and only the data changes.

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
  :root { --bg:#fff; --fg:#1a1a1a; --mut:#666; --line:#ddd; --hl:#fff8e1; --key:#8a6d00; --band:#f6f7f9; }
  * { box-sizing:border-box; }
  body { margin:0; font:15px/1.5 -apple-system,Segoe UI,Roboto,Helvetica,Arial,sans-serif; color:var(--fg); background:var(--bg); }
  header { padding:20px 24px 8px; }
  h1 { margin:0 0 4px; font-size:22px; }
  .summary { color:var(--mut); max-width:70ch; margin:0 0 14px; }
  .controls { display:flex; flex-wrap:wrap; gap:10px; align-items:center; padding:0 24px 14px; }
  .controls input, .controls select { font:inherit; padding:6px 9px; border:1px solid var(--line); border-radius:6px; background:var(--bg); color:var(--fg); }
  #q { flex:1; min-width:200px; }
  .controls label { display:flex; gap:5px; align-items:center; color:var(--mut); font-size:14px; }
  button { font:inherit; padding:6px 12px; border:1px solid var(--line); border-radius:6px; background:var(--band); color:var(--fg); cursor:pointer; }
  button:hover { border-color:var(--mut); }
  .count { color:var(--mut); font-size:13px; margin-left:auto; }
  table { border-collapse:collapse; width:100%; }
  th, td { text-align:left; padding:9px 14px; border-bottom:1px solid var(--line); vertical-align:top; }
  th { position:sticky; top:0; background:var(--bg); cursor:pointer; user-select:none; font-size:13px; letter-spacing:.02em; color:var(--mut); white-space:nowrap; }
  th[aria-sort] .arrow::after { content:" ↕"; opacity:.35; }
  th[aria-sort="ascending"] .arrow::after { content:" ↑"; opacity:1; }
  th[aria-sort="descending"] .arrow::after { content:" ↓"; opacity:1; }
  tr.key td { background:var(--hl); }
  td.date { white-space:nowrap; font-variant-numeric:tabular-nums; }
  .flag { cursor:help; color:var(--key); font-weight:700; }
  .exp { cursor:pointer; color:var(--mut); border:none; background:none; padding:0 6px 0 0; }
  .src { color:var(--mut); font-size:14px; }
  .keyreason { color:var(--key); font-size:13px; }
  .badge { display:inline-block; font-size:12px; font-weight:700; padding:1px 7px; border-radius:10px; }
  .yes { background:#fde68a; color:#7a5900; } .no { color:var(--mut); }
  tr.detail td { background:var(--band); font-size:14px; color:#333; }
  tr.detail dl { margin:0; display:grid; grid-template-columns:max-content 1fr; gap:2px 14px; }
  tr.detail dt { color:var(--mut); font-weight:600; } tr.detail dd { margin:0; }
  blockquote { margin:2px 0; padding-left:12px; border-left:3px solid var(--line); color:#333; }
  .note { color:#9a3412; }
  @media (prefers-color-scheme: dark) {
    :root { --bg:#1a1b1e; --fg:#e6e6e6; --mut:#9aa0a6; --line:#333; --hl:#2c2716; --key:#e0b64a; --band:#232428; }
    .yes { background:#4a3c10; color:#f0d894; } tr.detail dd { color:#cfcfcf; }
  }
  @media print {
    .controls, .exp, thead th .arrow, .count { display:none !important; }
    header { padding:0 0 10px; } body { font-size:11px; }
    tr.detail { display:none; } th { position:static; }
    tr.key td { background:#f4f4f4 !important; -webkit-print-color-adjust:exact; print-color-adjust:exact; }
    td, th { padding:5px 8px; }
    a[href]:after { content:""; }
  }
</style>
</head>
<body>
<header>
  <h1 id="ttl"></h1>
  <p class="summary" id="sum"></p>
</header>
<div class="controls">
  <input id="q" type="search" placeholder="Search all fields…" aria-label="Search">
  <label><input type="checkbox" id="keyonly"> Key only</label>
  <label>Party <select id="party"><option value="">All</option></select></label>
  <label>Issue <select id="issue"><option value="">All</option></select></label>
  <button id="xlsx">Excel</button>
  <button id="doc">Word</button>
  <button onclick="window.print()">PDF</button>
  <span class="count" id="cnt"></span>
</div>
<table>
  <thead><tr>
    <th data-k="date">Date<span class="arrow"></span></th>
    <th data-k="description">Description<span class="arrow"></span></th>
    <th data-k="source">Source<span class="arrow"></span></th>
    <th data-k="key">Key<span class="arrow"></span></th>
  </tr></thead>
  <tbody id="rows"></tbody>
</table>

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
  const q=$("#q").value.toLowerCase(), ko=$("#keyonly").checked, p=$("#party").value, is=$("#issue").value;
  return ROWS.filter(r=>{
    if(ko && !r.key) return false;
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
    const hasDetail = r.dateOriginal || r.dateNote || r.passage || r.note;
    const mark = r.certainty && r.certainty!=="exact" ? `<span class="flag" title="${esc(r.certainty)}: ${esc(r.dateNote||r.dateOriginal||"")}">${r.certainty==="uncertain"?"~":"*"}</span> ` : "";
    const tr=document.createElement("tr"); if(r.key) tr.className="key";
    tr.innerHTML =
      `<td class="date">${hasDetail?`<button class="exp" data-i="${r._i}">${open.has(r._i)?"▾":"▸"}</button>`:""}${mark}${esc(r.date)}</td>`+
      `<td>${esc(r.description)}</td>`+
      `<td class="src">${esc(r.source)}</td>`+
      `<td>${r.key?`<span class="badge yes">KEY</span><div class="keyreason">${esc(r.keyReason||"")}</div>`:`<span class="badge no">no</span>`}</td>`;
    tb.append(tr);
    if(hasDetail && open.has(r._i)){
      const d=document.createElement("tr"); d.className="detail";
      let dl="<dl>";
      if(r.dateOriginal) dl+=`<dt>Original</dt><dd>${esc(r.dateOriginal)}</dd>`;
      if(r.dateNote) dl+=`<dt>Resolved</dt><dd>${esc(r.dateNote)}</dd>`;
      if(r.passage) dl+=`<dt>Passage</dt><dd><blockquote>${esc(r.passage)}</blockquote></dd>`;
      if(r.note) dl+=`<dt>Note</dt><dd class="note">${esc(r.note)}</dd>`;
      d.innerHTML=`<td colspan="4">${dl}</dl></td>`; tb.append(d);
    }
  });
  $("#cnt").textContent = `${list.length} of ${ROWS.length} entries · ${ROWS.filter(r=>r.key).length} key`;
}

document.addEventListener("click",e=>{
  const ex=e.target.closest(".exp"); if(ex){ const i=+ex.dataset.i; open.has(i)?open.delete(i):open.add(i); render(); }
});
document.querySelectorAll("th").forEach(th=>th.addEventListener("click",()=>{
  const k=th.dataset.k; if(k===sortK) sortDir=-sortDir; else { sortK=k; sortDir=1; } render();
}));
["input","change"].forEach(ev=>["#q","#keyonly","#party","#issue"].forEach(s=>$(s).addEventListener(ev,render)));

// ---- Exports: all client-side, no libraries ---------------------------------
function tableHTML(){
  const rows=filtered(), h=["Date","Description","Source","Key","Reason / Note"];
  const cell=v=>`<td>${esc(v)}</td>`;
  const body=rows.map(r=>`<tr>${cell(r.date)}${cell(r.description)}${cell(r.source)}${cell(r.key?"Yes":"No")}${cell(r.key?(r.keyReason||""):(r.note||""))}</tr>`).join("");
  return `<table border="1"><thead><tr>${h.map(x=>`<th>${x}</th>`).join("")}</tr></thead><tbody>${body}</tbody></table>`;
}
function save(blob,name){ const a=document.createElement("a"); a.href=URL.createObjectURL(blob); a.download=name; a.click(); URL.revokeObjectURL(a.href); }
const base=()=> (META.title||"chronology").replace(/[^\w]+/g,"_").toLowerCase();
$("#xlsx").onclick=()=> save(new Blob(['﻿<html xmlns:x="urn:schemas-microsoft-com:office:excel"><head><meta charset="utf-8"></head><body>'+tableHTML()+'</body></html>'],
  {type:"application/vnd.ms-excel"}), base()+".xls");
$("#doc").onclick=()=> save(new Blob(['﻿<html xmlns:w="urn:schemas-microsoft-com:office:word"><head><meta charset="utf-8"><title>'+esc(META.title||"Chronology")+'</title></head><body><h1>'+esc(META.title||"Chronology")+'</h1>'+tableHTML()+'</body></html>'],
  {type:"application/msword"}), base()+".doc");
// -----------------------------------------------------------------------------
render();
</script>
</body>
</html>
```
