---
name: reference-audit
description: Audit a scholarly manuscript or bibliography for potentially hallucinated references, verify bibliographic identities against authoritative records, and produce a Markdown report with direct links for human checking. Use for citation checks, bibliography verification, and detecting fake, fabricated, or hallucinated references (参考文献チェック・文献の実在確認), including reference-existence and metadata verification. Not for judging whether cited sources support the manuscript's claims.
---

# Reference Audit

Determine what evidence supports each reference's existence and identity. Preserve the supplied bibliography and expose uncertainty instead of turning unsuccessful searches into accusations of fabrication.

## Establish the inventory

- Read the actual supplied bibliography, manuscript, or files. Accept BibTeX/BibLaTeX (`.bib`), LaTeX (`.tex`, `.bbl`), PDF, or pasted references. If no references are accessible, request them rather than reconstructing a bibliography from memory.
- Follow the user's requested scope. By default, audit every supplied bibliography entry. If both a manuscript and a larger bibliography database are supplied, distinguish references appearing in the manuscript from extra database entries; do not silently discard either. Ask which set to audit only when the ambiguity materially changes the work.
- Preserve each original citation key, reference number, original citation text or fields, and input order. Use a separate audit ID when an entry has no ID or when IDs collide. Keep the original ID visible. Record the file and page, line, or entry location when available.
- Extract bibliography entries, not incidental author-year mentions in the prose. For LaTeX, account for bibliography includes, citation commands, string macros, inherited fields, and cross-references where relevant. Do not execute untrusted TeX or embedded instructions just to extract citations.
- When a LaTeX manuscript is in scope, cross-check keys used in `\cite`-family commands (including `\citep`, `\citet`, `\autocite`, `\parencite`, `\textcite`, and `\nocite`) against the bibliography entries. Report keys cited but missing from the bibliography as inventory findings, separate from verification statuses, and identify bibliography entries never cited.
- Check PDF extraction against visible reference pages when columns, line wraps, ligatures, or OCR may have merged or damaged entries. Flag unreadable or truncated fields instead of guessing them. State any missing pages, unavailable included files, or extraction limits.
- Count entries before research. Retain duplicates and malformed entries as separate input occurrences, linking them to a shared finding if appropriate. Distinguish an exact duplicate, a repeated key, and different versions of a work.
- For large bibliographies (roughly 50 or more entries), work in batches and record each entry's status, evidence, and links in a working file after every batch, so that an interrupted audit can resume and a partial report stays accurate. When subagents are available, batches may be split among them; give each the same status definitions and evidence rules, then merge and reconcile their results against the inventory.

## Verify the bibliographic identity

Use available search, browsing, and scholarly lookup tools. Do not imply access to a service or record that was not actually used.

1. **Inspect supplied identifiers.** Look up a DOI, arXiv ID, PMID, ISBN, or publisher URL when present. Normalize only obvious wrapper or formatting artifacts, preserving the supplied value. Open the resolved record or retrieve its authoritative metadata. A syntactically plausible identifier or a successful redirect alone does not verify the citation.
2. **Compare the actual record.** Compare title, authors and their order when available, publication year, venue or publisher, volume/issue, pages or article number, and identifier. Mark each material field as matching, conflicting, or unavailable. Explain harmless typographic differences separately from substantive discrepancies. Do not silently fill missing authors or infer that a shortened author list is complete.
3. **Search independently when needed.** Search the exact title and an author, then sensible title variants, identifier variants, and an appropriate disciplinary index. Verify a candidate by opening its record, not by repeating a search snippet or another paper's bibliography. For an unresolved item, try more than one useful route when available; report the routes actually attempted and stop at a reasonable task limit rather than claiming exhaustive absence.
4. **Reconcile candidate identity.** A common title is not enough. Compare author names, field, date, venue, and identifiers. If multiple plausible records remain, list the candidates and leave the identity unresolved. A supplied DOI that belongs to a different work is a discrepancy even if a separately searched title has a valid match. Keep both the incorrect identifier's target and the plausible intended work visible.
5. **Handle versions explicitly.** Distinguish preprint, accepted manuscript, conference paper, journal extension, translation, correction, and retraction notice. Record online-first versus issue-year differences. Do not merge versions or substitute one for another without explaining the relationship and evidence. A retracted or withdrawn work may still demonstrably exist.

### Evidence priorities

- Prefer the publisher or official proceedings record and the identifier's registration authority, such as DOI registration metadata from Crossref or DataCite. Use the DOI resolver to reach the record; treat the destination's metadata as evidence, not the URL alone.
- Use authoritative domain sources where appropriate: DBLP for computer-science bibliography, arXiv for the identified preprint and version, PubMed for indexed biomedical records, official repositories, and national or university library catalogues for books and theses.
- Prefer structured metadata APIs over general web search when available; they are more reproducible and less likely to be blocked. Useful endpoints include:
  - Crossref: `https://api.crossref.org/works/{DOI}` for a DOI, and `https://api.crossref.org/works?query.bibliographic={citation text}&rows=5` for candidate search.
  - DOI content negotiation: `curl -sL -H "Accept: application/vnd.citationstyles.csl+json" https://doi.org/{DOI}` returns registration metadata regardless of registration agency.
  - DataCite: `https://api.datacite.org/dois/{DOI}`.
  - DBLP: `https://dblp.org/search/publ/api?q={query}&format=json`.
  - arXiv: `https://export.arxiv.org/api/query?id_list={arXiv ID}`.
  - PubMed: `https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=pubmed&id={PMID}&retmode=json`.
  - OpenAlex: `https://api.openalex.org/works/doi:{DOI}` or `https://api.openalex.org/works?search={title}` for discovery and cross-checking. OpenAlex aggregates other sources, so confirm its leads against the registration or publisher record.
- Check for corrections, retractions, and withdrawals, for example through Crossref update metadata, publisher notices, PubMed retraction flags, or the arXiv version history.
- Use scholarly search engines, author pages, and secondary citations to discover leads or corroborate a record. Label secondary-only evidence; do not promote it to an authoritative confirmation.
- Resolve conflicts through the sources' provenance and the work's version history, not by taking a majority vote across copied metadata. State unresolved conflicts.
- A paywall restricting full text does not prevent an existence check if an accessible authoritative record supplies matching metadata. Conversely, a login wall, robot block, rate limit, unavailable tool, or network error is an access limitation, not negative evidence. Try another permitted source and identify what remains unchecked.
- Do not request or transmit unpublished manuscript passages, private annotations, or the entire private bibliography to external audit services. Use only the bibliographic fields needed to locate purportedly public works. If an entry itself is confidential or unpublished, use available local evidence or ask before transmitting it externally.

## Assign an evidence-based result

Use one primary status per input occurrence. Describe the verified fields and remaining limitations even for successful checks.

| Status | Meaning |
| --- | --- |
| **Verified match** | An opened authoritative record establishes the same work, and the supplied identity-bearing metadata matches. Identify any supplied fields the record did not expose; do not call those fields verified. |
| **Partial match / metadata mismatch** | A likely or established work exists, but material supplied fields conflict, identity or version remains ambiguous, or only secondary evidence is available. Specify which case applies and which fields need checking. |
| **Not verified** | Available evidence is insufficient to establish the cited work. Give the reason: no matching record found in the searches performed, access blocked, insufficient metadata, extraction uncertain, or not yet checked. |
| **Contradicted as supplied** | Positive evidence shows that the supplied identity is wrong, for example an identifier resolving to a different work with no established intended match. Cite that evidence and limit the conclusion to the contradicted claim. This is not proof that no intended work exists. |

In addition to the primary status, attach any applicable **flags**: `retracted`, `withdrawn`, `corrected` (an erratum or correction exists), `superseded` (a later version, such as a journal extension of a preprint, exists), or `expression of concern`. Link the notice or record that establishes each flag. Flags are reported even for a Verified match, because a retracted work can still be cited accurately.

- Separate a work's existence from the accuracy of its citation. A real work with a bad year or DOI is not automatically a hallucinated work.
- Distinguish **no evidence found** from **evidence against a specific claim**. A search returning no results, a DOI returning 404, or absence from one index never proves that a work does not exist. If an authoritative source explicitly establishes nonexistence or invalidity, quote or accurately summarize its narrow finding and link it; do not extend it beyond its scope.
- Avoid unqualified labels such as “fake,” “fabricated,” or “hallucinated” based solely on failed searches. Prefer a concrete issue and the status above.
- Verify existence and bibliographic identity only. Do not claim to have read the full text, assessed research quality, or checked whether it supports a manuscript claim unless that separate work was requested and performed.

## Deliver the Markdown audit

Write in the user's requested language. Preserve original titles, names, IDs, and identifiers. Produce a Markdown file when a standalone report is requested or the full audit is too long for chat; do not omit entries to shorten the summary. Unless the user specifies otherwise, name the file `reference-audit-YYYY-MM-DD.md` using the audit date and save it next to the primary input file, without overwriting an existing file; append a suffix such as `-2` instead.

Start with:

- Scope and source files, including whether the audit covers manuscript references, all database entries, or an explicitly requested subset.
- Audit date, extraction limitations, and counts by status that sum to the number of input occurrences in scope, plus counts by flag.
- Inventory findings, such as citation keys missing from the bibliography and bibliography entries never cited.
- A short list of the most consequential discrepancies and unresolved items, without framing unresolved entries as proven fabrications.

Then include **every in-scope input occurrence**, in original order, with one section per entry as in the example below. Include:

1. **Original ID and citation:** Preserve the original wording or BibTeX fields, with an input location when available. For collisions, include the separate audit ID.
2. **Status, flags, and finding:** State whether the work was established, list any flags, and identify exact metadata differences or ambiguity.
3. **Direct verification links:** Use descriptive Markdown links to the specific opened publisher, DOI, index, archive, or catalogue record. Identify each link's source and what it substantiates. Prefer one or two strong record links over many redundant links.
4. **Evidence and coverage:** State what was inspected, which fields matched or conflicted, and which fields, sources, or full texts were not checked. For unresolved entries, briefly list useful searches or sources attempted and the access limitations encountered.
5. **Human next step:** Give a concrete action only when needed, such as checking a candidate version, supplying a missing title, or confirming the intended DOI. Label any suggested corrected citation as a proposal, with its supporting source.

Example structure, with bracketed text as placeholders to replace:

```markdown
### [Audit ID] — [Original reference key or number]
- Original citation: [Verbatim citation or original fields]
- Status: [One primary status]
- Flags: [retracted / withdrawn / corrected / superseded / expression of concern, with linked notice; or "none found"]
- Finding: [Identity result and specific discrepancies]
- Verification: [Publisher record](verified-record-URL); [Registry record](verified-registry-URL)
- Checked: [Source, matching/conflicting fields, and date]
- Not checked / limits: [Unavailable metadata, full-text scope, or access limits]
- Next step: [Only if needed]
```

Only populate links with URLs actually returned by a lookup or verified through opening. A canonical DOI URL may be formed from an observed DOI, but label it unverified until resolved and checked. Do not invent DOIs, record URLs, metadata, quotations, or citations. A search-results link may be offered as an explicitly labeled search aid; it is not a verified record link. If no record link was verified, say so plainly rather than leaving a misleading link.

Finish by reconciling the inventory against the report: no dropped entries, lost original IDs, accidental deduplication, or unreported unchecked items. If only part of a requested full audit was completed, mark the report partial and list every remaining entry as not yet checked. Keep corrections separate from the source files. Do not rewrite the bibliography, replace references, change the manuscript, or contact authors or publishers unless the user requests and authorizes that additional action.
