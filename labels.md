# Visor AI Chat Monetization Dataset v1 — schema and label definitions

**Dataset version:** 1.0.0  
**Publisher:** Visor Research / the Visor team  
**Project source:** [Visor](https://visor.host)

This release is a research dataset for AI chat monetization: contextual ad matching for AI chat apps. Visor is a US-only ad network and SSP (supply-side platform) that monetizes AI chat apps.

This dataset is a public sample of the data the Visor team uses to train its ad-matching models.

This document describes the fields and label rules used in `dataset.jsonl` and `dataset.csv`. IAB category IDs and names are taken from the official IAB Tech Lab Content Taxonomy 3.0 and were checked against the published TSV in the [InteractiveAdvertisingBureau/Taxonomies](https://github.com/InteractiveAdvertisingBureau/Taxonomies) repository (`Content Taxonomies/Content Taxonomy 3.0.tsv`). IDs were not invented.

The IAB taxonomy file is not redistributed in this package. It remains subject to IAB Tech Lab terms. This dataset only uses the subset of tier-1 and tier-2 identifiers and names that appear as labels.

## Fields

| Field | Type | Description |
| :--- | :--- | :--- |
| `id` | string | Stable row id, `visor-chat-ad-intent-v1-NNNNN`. |
| `dataset_version` | string | Dataset version. This release is `1.0.0`. |
| `split` | string | `train` or `validation`. Stratified ~90/10 split by category and `ad_eligible`. |
| `message` | string | A synthetic user chat message, written as if typed by a US English speaker into an AI chat app. No real user logs and no real personal data. |
| `prior_context` | string | Optional previous-turn context (empty when the message is standalone). Synthetic. Included so a model can use short multi-turn context. |
| `chat_app_type` | string | One of `companion`, `homework_helper`, `coding_assistant`, `general_assistant`, `roleplay`. |
| `locale` | string | Always `en-US`. The dataset is scoped to US-English chat. |
| `iab_tier1_id` | string | Official IAB Content Taxonomy 3.0 tier-1 unique ID. |
| `iab_tier1_name` | string | Official tier-1 name, matching that ID. |
| `iab_tier2_id` | string | Official IAB Content Taxonomy 3.0 tier-2 unique ID. |
| `iab_tier2_name` | string | Official tier-2 name, matching that ID and parent. |
| `commercial_intent` | string | `none`, `low`, `medium`, or `high`. See rubric below. |
| `funnel_stage` | string | `none`, `awareness`, `consideration`, or `purchase`. See rubric below. |
| `brand_safe` | boolean | `true` if a labeled ad could be considered next to this message from a brand-safety point of view; `false` otherwise. |
| `brand_safe_reason` | string | Empty when `brand_safe` is true. When false, a short reason: `self-harm`, `medical crisis`, `adult`, `hate`, `minors`, `illegal drugs`, or `crime`. |
| `ad_eligible` | boolean | Whether a contextual ad should be allowed to serve after the assistant's answer for this turn. |
| `source` | string | Always `synthetic`. |
| `category_key` | string | Internal generation key for the row's template family. Useful for analysis; not an IAB identifier. |

## Commercial intent

Assigned from the message as written, not from a user profile.

- **none** — No shopping, booking, subscription, or product-research ask. Typical of homework help, debugging, emotional conversation, or unsafe requests. `funnel_stage` is always `none`.
- **low** — Topic interest without a buying problem. The user is learning or exploring ("how does a deductible work?").
- **medium** — Active comparison or planning, without an immediate checkout ask.
- **high** — A concrete choose-or-buy problem, often with a budget, retailer, timeframe, or "what should I get?"

## Funnel stage

- **none** — Used only with `commercial_intent = none`.
- **awareness** — The user is learning about a category or problem.
- **consideration** — The user is comparing options, features, or fit.
- **purchase** — The user is close to a transaction (booking, applying, ordering, or shopping now).

## Brand safety

`brand_safe = false` is used for turns where serving an ad would be inappropriate or harmful, including:

- self-harm or suicide-related crisis language
- a medical emergency happening now
- explicit adult sexual content
- hate or harassment targeting a protected or marginalized group
- sexual or exploitative context involving minors, or other minor-safety issues (for example, underage alcohol or nicotine)
- requests for help committing a crime
- requests for help obtaining illegal drugs, or evading the law around drugs, tobacco, or alcohol (including for minors)

These examples are synthetic and are included so a model can learn **not** to serve ads. They are not instructions, and they are not real user reports.

General (non-crisis) health questions, ordinary homework, and ordinary emotional venting can be `brand_safe = true` and still `ad_eligible = false`.

## Ad eligibility

`ad_eligible` is true only when all of the following hold:

1. `brand_safe` is true.
2. There is some commercial or contextual reason to show a category-matched ad (`commercial_intent` is not `none`).
3. The turn is not a non-commercial study, debug, or support conversation that should stay ad-free.

Hard rules encoded in the release:

- If `brand_safe` is false, `ad_eligible` is false and `brand_safe_reason` is non-empty.
- If `commercial_intent` is `none`, `funnel_stage` is `none`.
- If `commercial_intent` is `none` and `brand_safe` is true, `ad_eligible` is false.

## IAB mapping

Each row has exactly one primary tier-1 and tier-2 pair. Multi-topic messages are labeled with the dominant intent. Self-harm and suicide-related crisis language is mapped to Sensitive Topics > Death, Injury, or Military Conflict (`I4GWl6`), with `brand_safe_reason = self-harm`. That is a training convention for brand-safety eligibility, not a claim that IAB defined a separate "self-harm" node.

The 70 tier-2 labels used in this release are listed with counts in `stats.md`.

## Chat app types

- **general_assistant** — Everyday questions, shopping, travel, planning.
- **companion** — Conversational, social, or emotional talk.
- **homework_helper** — Study, essays, explanations, quizzes.
- **coding_assistant** — Debugging, refactors, tests, programming questions.
- **roleplay** — Requested scenes, study-partner or interview practice, book/show discussion in character.

## Source and locale

All messages are **synthetic**. They were generated from hand-written templates, slot lexicons, and a light style layer (typos, casual US English, optional prior turns). They are not sampled from production chat logs. No real names, emails, phone numbers, addresses, or account identifiers are used. Locale is `en-US` only.

## License

MIT, for this dataset's messages, labels, and documentation. IAB taxonomy names and IDs are used for labeling and remain subject to IAB Tech Lab terms.
