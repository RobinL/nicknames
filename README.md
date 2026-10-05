# Given-name and nickname relationships supported by Wikipedia

Evidence snapshot: **3 October 2026**. Publication layout updated: **5 October 2026**.

This dataset contains **1,993 distinct given-name → nickname pairs**, each supported by **at least three distinct people under the project's Wikipedia validation rules**. English Wikipedia biographies provide the name evidence. Linked Wikidata identifiers support identity resolution and deduplication; Wikidata human-status checks are also part of the reverse-search method.

The repository contains three UTF-8 CSVs with ordinary text cells and no JSON fields:

| File | One row per | Rows | Key |
|---|---|---:|---|
| [nicknames.csv](nicknames.csv) | Qualifying given-name/nickname pair | 1,993 | `pair_id` |
| [supporting_people.csv](supporting_people.csv) | Person supporting a pair, per validation record | 11,734 | `evidence_id` |
| [provenance.csv](provenance.csv) | Technical provenance of a validation record | 11,734 | `evidence_id` |

The evidence represents **11,067 distinct person–pair supports**, involving **10,892 different people**. Multiple records can describe one person, but that person contributes only one vote within a pair. Every supporting-person record has a Wikipedia URL and Wikidata person identifier.

No repository-wide licence has been selected. The source notices below explain the existing terms for Wikipedia and Wikidata material; they do not apply MIT or another blanket licence to the compilation.

## Results by validation method

| Main method | Qualification pathway | Pairs |
|---|---|---:|
| Original validation of recalled people | Article title and first 100 introduction words | 757 |
| Reverse Wikipedia discovery | Introduction-based evidence sufficient | 1,223 |
| Reverse Wikipedia discovery | Wider article context needed | 13 |
| **Total** | **At least three distinct supporting people** | **1,993** |

“Supported” and “validated” mean that the recorded rules passed. They do not guarantee that every name mention is correctly attributed or that a relationship is conventional in every culture. An unsuccessful lookup is insufficient evidence under the method, not proof that the relationship is false.

## Candidate generation

Four LLM agents were initially asked to propose conventional relationships using only pretrained knowledge, without browsing or retrieval. A pair could be proposed only when the agent could recall at least three different real people who exemplified it. Fictional characters, surnames, unrelated stage names, descriptive aliases, pure spelling variants and translations without nickname use were excluded by the prompt. Each relationship was considered independently; pairs were not inferred by following a chain of nicknames.

These generated examples were hypotheses requiring validation. Agreement between agents was not an extra supporting person. Consolidating and validating the four original outputs produced 702 qualifying pairs.

A later, broader offline prompt requested wide coverage across languages, historical usage and native scripts. It instructed the agent not to reproduce a particular dictionary or existing nickname table, copy dictionary prose, invent sources or generate merely plausible shortenings. Unlike the initial prompt, it did **not** require three named people during candidate generation. All rows were to be labelled `unvalidated`; high, medium or low model confidence represented uncertainty in recall, not verification. The exact prompt is reproduced below.

That broader list then received a separate offline supporting-person recall exercise, with abstention allowed when examples could not be recalled. Wikipedia validation of the additional examples increased the original-method list to 757 pairs. Reverse discovery subsequently added 1,236 pairs.

The workflow and prompts do not establish that model training data contained no existing nickname compilation, certify compliance with every generation instruction, or guarantee copyright or database-right clearance.

## Method 1: validate recalled people

Pair-level label: **`validation_method = original_validation`**.

The starting point was a claimed person, their fuller name and an identifying description. These helped resolve an English Wikipedia biography, allowing for public names and middle-name variations while avoiding obvious disambiguation pages and non-biographies.

The automated `enwiki_title_and_intro_first_100_words_v1` rule requires:

1. The base given name to match a whole word or phrase in the first **100 whitespace-delimited words of the plain-text introduction**.
2. The nickname to match a whole word or phrase in that window **or the canonical article title**.
3. At least three distinct resolved people with passing checks for the pair to qualify.

Matching uses Unicode NFC normalization and case folding. Accents and different spellings remain distinct. A short introduction is not extended into the body to reach 100 words.

This is a lexical check on a biography resolved using the recalled person's identity. It does not prove that every matching name refers to the subject: a relative or another person can appear in the introduction. The short window reduces that risk without eliminating it. The original evidence retains its original lexical-validation label.

## Method 2: reverse Wikipedia discovery

Pair-level label: **`validation_method = reverse_search`**.

The starting point was the proposed base name and nickname, rather than a recalled person. English Wikipedia searches sought biographies containing both terms. Previously cached introductions were also checked against additional pairs, with that reuse recorded separately.

The run attempted all 12,684 normalized candidate pairs still below the three-person threshold in its source list. It used batches of proximity phrases, split large result sets and bounded individual searches. Titles, redirects and naming cues in search extracts guided retrieval. This was a bounded discovery process, not an exhaustive search of all articles or languages.

New automated support required a human entity and an opening subject-name structure linking the fuller given name and nickname to the biography's subject. Rules screened out surname matches, relatives, family narrative and bases that were themselves quoted nicknames. These were automated structural checks, not individual human reviews. Wikidata supplied identity and human-status information; its nickname assertions were not the relationship evidence.

`qualification_method` distinguishes the two reverse-search pathways:

- **`reverse_search_introduction`**: three distinct people were supported with the introduction rule, including any existing original-method supports. This accounts for 1,223 additions.
- **`reverse_search_extended_text`**: reaching three required evidence from a longer introduction, article body or a retained pilot review using wider context. This accounts for 13 additions. Automated wider-text checks required an explicit subject-linked naming statement, a matching surname and an opening given-name anchor. Pilot decisions retain their actual assistant-review labels.

A reverse-qualified pair can include people originally found by recall. The pair-level method identifies how the relationship qualified, not the discovery route of every supporting person.

Two retained pilot records display short introduction excerpts but have wider-context review methods. Qualification follows the recorded decision method, not excerpt length alone.

## Deduplication and requests

Pairs are pooled across files after normalizing NFC, case and whitespace. Accents and distinct spellings remain separate. Candidate naming-tradition descriptions are retained, but support is pooled across those descriptions: three people for a pair does not establish three people in every listed culture.

People are counted by resolved identities, principally Wikidata IDs, rather than name-string similarity. The wider pipeline supports reconciled Wikipedia page IDs as a fallback; the people in this export have Wikidata IDs. Repeated source rows, descriptions, redirects, retrievals and agent agreement cannot inflate the distinct-person count. Evidence from two methods for the same person is retained without adding a second vote.

Wikipedia requests used a serial cached client, a descriptive User-Agent, `maxlag=5`, at least a one-second pause after each response, Retry-After handling and exponential backoff. Successful responses were reused. Request IDs, timestamps and raw-response hashes remain in the provenance records.

## Small online spot-check

A manual, AI-assisted review of 100 sampled reverse-discovered pairs against other online sources found no demonstrated errors: 98 pairs were corroborated and two remained inconclusive. This was not a blinded human evaluation and does not establish a zero error rate.

## CSV schemas and joins

Names are repeated alongside `pair_id` in all tables for readability. Use IDs for joins, and parse the files as CSV: some quoted cells contain line breaks. The main lookup table is about 267 KB.

### nicknames.csv

| Column | Meaning |
|---|---|
| `pair_id` | Stable identifier of the normalized directed relationship |
| `base_given_name`, `nickname` | Display forms of the base name and nickname |
| `validation_method` | `original_validation` or `reverse_search` |
| `qualification_method` | `original_validation`, `reverse_search_introduction` or `reverse_search_extended_text` |
| `supporting_person_count` | Distinct supporting people, always at least three |
| `candidate_traditions` | Original naming-context descriptions, one source label per line in a quoted cell; these are not verified cultural claims |
| `snapshot_date` | Date of the consolidated evidence snapshot, not a new retrieval |

### supporting_people.csv

The columns are `evidence_id`, `pair_id`, `base_given_name`, `nickname`, `person_id`, `person_name`, `discovery_method`, `validation_method`, `review_status`, `evidence_text` and `wikipedia_url`.

`person_name` is the article title; the retained passage supplies fuller-name context. The nickname can occur in the title even when absent from the passage. Count distinct `person_id` values within each pair, not rows, when counting people.

The readable discovery labels are:

| Label | Meaning |
|---|---|
| `recalled_person` | Offline recalled person subsequently resolved to a biography |
| `pair_search` | Full-run search using the name pair |
| `cached_introduction` | Previously retrieved introduction checked against another pair |
| `pilot_pair_search` | Pair-driven discovery in the earlier feasibility experiment |

The person-level `validation_method` values are `introduction_100_words`, `extended_subject_rule` and `extended_assistant_review`. These describe the evidence check; the same-named column in `nicknames.csv` describes the pair's qualification route. `review_status` distinguishes original lexical validation, automated subject checks and assistant-reviewed pilot evidence. The full technical method names remain in `provenance.csv`.

### provenance.csv

Join this table to `supporting_people.csv` using `evidence_id`. There is exactly one provenance row per evidence row. It preserves original method names, support origin, attribution method, reviewer, excerpt scope, review status, article revision URL/ID/timestamp, retrieval timestamp, page and entity IDs, identity-counting notes, request ID and raw-response path/hash.

Exact checked name spellings and original person/evidence ordering are also retained. Case-only variants such as JoJo/Jojo share a normalized pair without losing their recorded spelling. Evidence IDs are stable hashes of the pair ID, person ID and original evidence record; changing a record changes its ID.

The retained API response was the exact text checked. Its rendered extract was not separately reconstructed from its associated revision metadata. Internal raw-response paths identify files in the research working project; those cache files are not distributed here.

## Use for probabilistic record linkage

These pairs can support name comparisons when edit-distance methods miss relationships such as Elizabeth/Betty. They indicate possible name correspondence, not that two records identify the same person. Combine them with other fields and assess performance on representative data. A nickname may map to several fuller names or be a given name in its own right. Do not infer additional pairs by transitivity.

Coverage reflects offline recall, public-figure biographies and English Wikipedia availability. The dataset does not establish completeness, nickname frequency, contemporary prevalence or validity in every naming tradition.

```python
import csv
from pathlib import Path


def read_table(name):
    with Path(name).open(encoding="utf-8", newline="") as f:
        return list(csv.DictReader(f))


pairs = read_table("nicknames.csv")
people = read_table("supporting_people.csv")
provenance = read_table("provenance.csv")

robin_pairs = [r for r in pairs if r["base_given_name"] == "Robin"]
pair_ids = {r["pair_id"] for r in robin_pairs}
robin_people = [r for r in people if r["pair_id"] in pair_ids]
evidence_ids = {r["evidence_id"] for r in robin_people}
robin_provenance = [r for r in provenance if r["evidence_id"] in evidence_ids]
```

## Source attribution and existing terms

No blanket repository licence is granted by this notice. Existing third-party licences continue to govern the material they cover; no ownership of public-domain facts or third-party works is asserted.

**Wikipedia text.** The `evidence_text` passages are attributed to the contributors of the English Wikipedia articles identified by `wikipedia_url`, where contributor histories are available. Associated revision URLs and metadata are linked through `evidence_id` in `provenance.csv`.

Wikipedia-derived text is reproduced under its applicable [Creative Commons Attribution-ShareAlike 4.0 International licence](https://creativecommons.org/licenses/by-sa/4.0/), subject to any separately identified source terms or exceptions for particular material. See the [licence text](https://creativecommons.org/licenses/by-sa/4.0/legalcode.en) and [Wikimedia reuse terms](https://foundation.wikimedia.org/wiki/Policy:Terms_of_Use/en#7._Licensing_of_Content).

Articles were retrieved as plain text, passages were selected, and introductions were generally truncated to their first 100 whitespace-delimited words. Web formatting is not preserved. Those extraction and formatting changes to licensed text are supplied under the same applicable terms. No endorsement by Wikipedia contributors or the Wikimedia Foundation is implied.

**Wikidata metadata.** Identifiers such as `Q7340959` resolve at `https://www.wikidata.org/wiki/Q7340959`. Wikidata releases structured data under [CC0](https://creativecommons.org/publicdomain/zero/1.0/), as described in its [licensing policy](https://www.wikidata.org/wiki/Wikidata:Licensing). That status does not extend to Wikipedia prose or the compilation as a whole.

## Export verification

The source relationship and evidence snapshots in the research working project are `validation_reverse_20261002/outputs/validated_nickname_list.csv` and `validation_reverse_20261002/outputs/supporting_people_by_method.csv`.

Export checks reconcile all 1,993 pair IDs, every supporting-person ID and all 11,734 retained Wikipedia records to those snapshots. They check the three-person threshold, method-specific qualification, unique keys, foreign-key joins, distinct-person counts, source URLs and UTF-8/CSV round trips. The three CSVs preserve the qualified pairs and their Wikipedia evidence without a fresh validation run.


## Exact prompt for the larger candidate list

The user supplied the following as the prompt used to generate the larger candidate list. It applies to initial candidate generation, before the separate supporting-person recall and Wikipedia validation stages.

```text
Create a comprehensive candidate dataset of conventional given-name nicknames and diminutives for probabilistic record linkage, eventually intended for inclusion in an MIT-licensed DuckDB extension.

This is the initial generation stage. Every relationship will be independently validated later.

Use only your pretrained general knowledge. Do not browse, search, fetch URLs, consult external datasets, or use retrieval. Local file-writing tools are allowed. Do not attempt to reproduce a particular dictionary, published nickname table or existing dataset. Do not include dictionary prose, quotations, citations or invented sources.

Include relationships where multiple unrelated people conventionally use one given-name form in place of another, such as Elizabeth → Betty, Alexander → Xander, Robin → Rob, and Alexander → Sasha in relevant naming traditions.

Generate the widest credible coverage you can:

- Prioritise English usage, including British, Irish, Scottish, Welsh and American conventions.
- Then systematically cover other languages and naming traditions, including those represented in the UK.
- Include established relationships that are culturally specific, historical or less familiar.
- Include native-script pairs where you know them. Where useful, add corresponding Latin-script pairs and label the transliteration context.
- Include nicknames that are also independently used as given names.

Exclude:

- Person-specific aliases, stage names, usernames, descriptive sobriquets and surnames.
- Pure spelling variants, translations and transliterations without an actual nickname relationship.
- Hypothetical shortenings or suffix combinations that merely look plausible.
- Relationships inferred solely through another nickname. Validate each proposed edge from your own knowledge; never treat connected names as an equivalence class.

Represent each relationship as a directed edge from the fuller given-name form to its conventional nickname. A nickname may have several base names. Neither form must necessarily be someone’s legal name.

Produce UTF-8 CSV with these columns:\
candidate_id,base_given_name,nickname,language_or_tradition,relationship_type,model_confidence,usage_notes,generation_batch,validation_status

Use:

- relationship_type: shortening, diminutive, conventional_nickname
- model_confidence: high, medium, low
- validation_status: unvalidated

Confidence represents your own uncertainty, not external verification. Low-confidence entries should reflect a relationship you recall as established, rather than one you have invented. Keep usage notes brief, factual and in your own words. Preserve accents and script distinctions.

Work in systematic batches to maximise coverage. Deduplicate identical relationships within the same naming context. Do not stop after a small illustrative sample, and do not pad the dataset to reach an arbitrary row count.

If file-writing tools are available, save:

1. nickname_candidates.csv
2. generation_manifest.md containing the exact prompt, generation date, model/version if genuinely available, batch coverage, row counts, known gaps and the CSV’s SHA-256 hash.

Freeze the resulting candidate file before any external validation. Do not claim that offline generation guarantees copyright clearance.

If file-writing is unavailable, output the CSV in clearly numbered batches with one initial header. If an output limit prevents completion, state exactly which naming traditions remain and where to resume.
```
