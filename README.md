# Visor AI Chat Monetization Dataset v1: Ad Intent & IAB Categories for AI Chat Messages

**Released by:** Visor Research / the Visor team  
**Project:** [Visor, AI chat monetization](https://visor.host) · **Advertisers:** [AI chatbot ads](https://visor.host/advertisers)  
**Version:** 1.0.0 · **Date:** 25 November 2026 · **License:** [MIT](LICENSE)

This release is a research dataset for AI chat monetization: contextual ad matching for AI chat apps. Visor is a US-only ad network and SSP (supply-side platform) that monetizes AI chat apps.

This dataset is a public sample of the data the Visor team uses to train its ad-matching models. **All messages are synthetic.** The release contains no real user chat logs, cookies, or personal data.

**This file** is the GitHub repository landing page. The Hugging Face dataset card is [`DATASET_CARD.md`](DATASET_CARD.md). The same release is published at [visor-research/ai-chat-monetization-dataset-v1](https://huggingface.co/datasets/visor-research/ai-chat-monetization-dataset-v1).

## TL;DR

20,080 synthetic US-English chat messages, each labeled with an official [IAB Tech Lab Content Taxonomy 3.0](https://iabtechlab.com/standards/content-taxonomy/) tier-1 and tier-2 category, commercial intent, funnel stage, brand safety, and ad eligibility. The task is contextual, cookieless matching of an ad to a chat turn — not to a person. A substantial share of turns should not see an ad at all (homework, debugging, emotional support, crisis, hate, adult content, crime, or anything involving minors).

| | |
| :--- | :--- |
| Rows (unique normalized messages) | 20,080 |
| IAB tier-2 labels | 70 |
| Split | train 18,072 (90.0%) / validation 2,008 (10.0%) |
| Ad-eligible | 16,324 (81.29%) yes · 3,756 (18.71%) no |
| Brand-safe | 19,965 (99.43%) yes · 115 (0.57%) no |
| Locale | `en-US` |
| Source | `synthetic` on every row |

## Contents

- [Repository contents](#repository-contents)
- [Why AI chat monetization needs intent data](#why-ai-chat-monetization-needs-intent-data)
- [Quickstart](#quickstart)
- [Data fields](#data-fields)
- [Label definitions](#label-definitions)
- [Example rows](#example-rows)
- [Distributions](#distributions)
- [Suggested tasks for AI chat monetization](#suggested-tasks-for-ai-chat-monetization)
- [Methodology](#methodology)
- [Datasheet](#datasheet)
- [Brand-safety approach](#brand-safety-approach)
- [Limitations and biases](#limitations-and-biases)
- [Ethics and intended use](#ethics-and-intended-use)
- [Versioning](#versioning)
- [How to cite](#how-to-cite)
- [License](#license)
- [Contact](#contact)

## Repository contents

| Path | Purpose |
| :--- | :--- |
| `data/train.jsonl`, `data/validation.jsonl` | Official splits (18,072 / 2,008) |
| `data/dataset.jsonl`, `data/dataset.csv` | Full 20,080-row corpus |
| `DATASET_CARD.md` | Hugging Face dataset card (YAML front matter plus the same documentation) |
| `labels.md` | Field and rubric definitions |
| `stats.md` / `stats.json` | Measured counts and distributions |
| `LICENSE` | MIT |
| `CHANGELOG.md` | Version history |
| `CITATION.cff` / `CITATION.bib` | Citation files |
| `dataset-schema.jsonld` | schema.org Dataset JSON-LD |

## Why AI chat monetization needs intent data

Contextual targeting in AI chat is not the same problem as classifying a web page. The unit of context is a short, messy, sometimes multi-turn message. Many turns have no commercial intent. Some turns should never see an ad.

Visor is a US-only ad network and SSP (supply-side platform) that monetizes AI chat apps. A team with over 10 years in the ad sector built it. The network currently monetizes about 10 million users, all in the United States, as of 21 November 2026 (people in US apps that passed review; not a count of apps). Ads are placed after a relevant answer, not inside the model's sentence. Targeting is contextual. Ad serving is cookieless and does not use personal data — no names, emails, cookies, or lists of people.

This dataset is a public sample of the data the Visor team uses to train its ad-matching models. It is released so others can train and evaluate models that:

1. assign an IAB Content Taxonomy 3.0 tier-2 topic to a chat message
2. estimate commercial intent and funnel stage
3. decide whether a turn is brand-safe and ad-eligible

It is not a substitute for human review of live traffic, and it is not a dump of production chat logs. Details of Visor's ad-matching system are not documented here.

## Quickstart

Load the dataset from the Hub at `visor-research/ai-chat-monetization-dataset-v1`.

```python
from datasets import load_dataset

ds = load_dataset("visor-research/ai-chat-monetization-dataset-v1")
print(ds)
print(ds["train"][0]["message"], ds["train"][0]["ad_eligible"])
```

Load the split files in this repository directly:

```python
from datasets import Dataset, DatasetDict
import json

def read_jsonl(path):
    with open(path, encoding="utf-8") as f:
        return [json.loads(line) for line in f]

ds = DatasetDict({
    "train": Dataset.from_list(read_jsonl("data/train.jsonl")),
    "validation": Dataset.from_list(read_jsonl("data/validation.jsonl")),
})
print(len(ds["train"]), len(ds["validation"]))
```

With pandas:

```python
import pandas as pd

df = pd.read_csv("data/dataset.csv")
print(df.shape)
print(df["split"].value_counts())
print(df["ad_eligible"].value_counts())
print(df["iab_tier2_name"].value_counts().head())
```

The official split is the `split` column (18,072 train / 2,008 validation). Do not reshuffle if you want results that are comparable to other work on this release.

## Data fields

See [`labels.md`](labels.md) for the full rubric. Summary of every column:

| Field | Type | Allowed values / format | Example |
| :--- | :--- | :--- | :--- |
| `id` | string | `visor-chat-ad-intent-v1-NNNNN` | `visor-chat-ad-intent-v1-00055` |
| `dataset_version` | string | `1.0.0` in this release | `1.0.0` |
| `split` | string | `train` or `validation` | `train` |
| `message` | string | Synthetic user utterance, US English | `how many days do i actually need in rome without rushing?` |
| `prior_context` | string | Optional synthetic previous turn; empty if standalone | `You asked me to name my top constraint. I have it now.` |
| `chat_app_type` | string | `companion` · `homework_helper` · `coding_assistant` · `general_assistant` · `roleplay` | `general_assistant` |
| `locale` | string | Always `en-US` | `en-US` |
| `iab_tier1_id` | string | Official IAB 3.0 tier-1 unique ID | `653` |
| `iab_tier1_name` | string | Official tier-1 name matching that ID | `Travel` |
| `iab_tier2_id` | string | Official IAB 3.0 tier-2 unique ID | `655` |
| `iab_tier2_name` | string | Official tier-2 name matching that ID and parent | `Travel Locations` |
| `commercial_intent` | string | `none` · `low` · `medium` · `high` | `medium` |
| `funnel_stage` | string | `none` · `awareness` · `consideration` · `purchase` | `awareness` |
| `brand_safe` | boolean | `true` or `false` (CSV: `true`/`false`) | `true` |
| `brand_safe_reason` | string | Empty if safe; otherwise `self-harm`, `medical crisis`, `adult`, `hate`, `minors`, `illegal drugs`, or `crime` | *(empty)* |
| `ad_eligible` | boolean | `true` or `false` | `true` |
| `source` | string | Always `synthetic` | `synthetic` |
| `category_key` | string | Internal template-family key; not an IAB id | `travel_locations` |

## Label definitions

### Commercial intent

Assigned from the message as written, not from a user profile.

- **none** — No shopping, booking, subscription, or product-research ask. Typical of homework help, debugging, emotional conversation, or unsafe requests. `funnel_stage` is always `none`.
- **low** — Topic interest without a buying problem (the user is learning or exploring).
- **medium** — Active comparison or planning, without an immediate checkout ask.
- **high** — A concrete choose-or-buy problem, often with a budget, retailer, timeframe, or “what should I get?”

### Funnel stage

- **none** — Used only with `commercial_intent = none`.
- **awareness** — The user is learning about a category or problem.
- **consideration** — The user is comparing options, features, or fit.
- **purchase** — The user is close to a transaction (booking, applying, ordering, or shopping now).

### Brand safety and ad eligibility

`brand_safe = false` is used when serving a labeled ad after the turn would be inappropriate or harmful, including self-harm or suicide-related crisis language, a medical emergency happening now, explicit adult sexual content, hate or harassment, sexual or exploitative context involving minors or other minor-safety issues, requests for help committing a crime, and requests for help obtaining illegal drugs or evading the law around drugs, tobacco, or alcohol (including for minors). These examples are synthetic and exist so a model can learn **not** to serve ads. They are not instructions and not real user reports.

General (non-crisis) health questions, ordinary homework, and ordinary emotional venting can be `brand_safe = true` and still `ad_eligible = false`.

`ad_eligible` is true only when `brand_safe` is true and there is some commercial or contextual reason to show a category-matched ad (`commercial_intent` is not `none`). Encoded rules in this release:

- If `brand_safe` is false, `ad_eligible` is false and `brand_safe_reason` is non-empty.
- If `commercial_intent` is `none`, `funnel_stage` is `none`.
- If `commercial_intent` is `none` and `brand_safe` is true, `ad_eligible` is false.

### IAB mapping

Each row has exactly one primary tier-1 and tier-2 pair, taken from the official IAB Content Taxonomy 3.0 TSV ([InteractiveAdvertisingBureau/Taxonomies](https://github.com/InteractiveAdvertisingBureau/Taxonomies), file `Content Taxonomies/Content Taxonomy 3.0.tsv`). IDs were not invented. Multi-topic messages are labeled with the dominant intent. Self-harm and suicide-related crisis language is mapped to Sensitive Topics > Death, Injury, or Military Conflict (`I4GWl6`), with `brand_safe_reason = self-harm`. That is a training convention for brand-safety eligibility, not a new IAB node. This package does not republish the full IAB TSV.

### Chat app types

- **general_assistant** — Everyday questions, shopping, travel, planning.
- **companion** — Conversational, social, or emotional talk.
- **homework_helper** — Study, essays, explanations, quizzes.
- **coding_assistant** — Debugging, refactors, tests, programming questions.
- **roleplay** — Requested scenes, study-partner or interview practice, book or show discussion in character.

## Example rows

The following records are taken from this release (fields abbreviated). They are synthetic.

**Ad-eligible, high consideration — Consumer Electronics** (`visor-chat-ad-intent-v1-00055`, train):

```json
{
  "message": "Buy a 65-inch OLED now or wait, prices feel high in Indianapolis. Id like to sort this out before Christmas.",
  "prior_context": "You warned me about fees or add-ons. I want to apply that to the actual choice.",
  "chat_app_type": "general_assistant",
  "iab_tier1_name": "Technology & Computing",
  "iab_tier2_id": "632",
  "iab_tier2_name": "Consumer Electronics",
  "commercial_intent": "high",
  "funnel_stage": "consideration",
  "brand_safe": true,
  "brand_safe_reason": "",
  "ad_eligible": true
}
```

**Ad-eligible travel — Travel Locations** (`visor-chat-ad-intent-v1-00045`, train):

```json
{
  "message": "how many days do i actually need in rome without rushing?",
  "prior_context": "",
  "chat_app_type": "general_assistant",
  "iab_tier1_name": "Travel",
  "iab_tier2_id": "655",
  "iab_tier2_name": "Travel Locations",
  "commercial_intent": "medium",
  "funnel_stage": "awareness",
  "brand_safe": true,
  "ad_eligible": true
}
```

**Ad-eligible purchase — Dining Out** (`visor-chat-ad-intent-v1-00101`, train):

```json
{
  "message": "Whats a good Mediterranean delivery option when I dont wanna cook, around $25 for two?",
  "prior_context": "I said I was just researching, not buying today — that changed.",
  "chat_app_type": "general_assistant",
  "iab_tier1_name": "Food & Drink",
  "iab_tier2_id": "218",
  "iab_tier2_name": "Dining Out",
  "commercial_intent": "medium",
  "funnel_stage": "purchase",
  "brand_safe": true,
  "ad_eligible": true
}
```

**Not ad-eligible — homework** (`visor-chat-ad-intent-v1-00012`, train):

```json
{
  "message": "I want to check that my evidence paragraph for physics kinematics uses the terms correctly. I'll paste it.",
  "prior_context": "",
  "chat_app_type": "homework_helper",
  "iab_tier1_name": "Education",
  "iab_tier2_id": "146",
  "iab_tier2_name": "Homework and Study",
  "commercial_intent": "none",
  "funnel_stage": "none",
  "brand_safe": true,
  "ad_eligible": false
}
```

**Not ad-eligible — coding help** (`visor-chat-ad-intent-v1-00006`, train):

```json
{
  "message": "I'm stuck on a wrong comparison in Go. I'll paste a minimal snippet — pls just debug it, no new libraries.",
  "prior_context": "You asked me to say a bit more about what I already tried.",
  "chat_app_type": "coding_assistant",
  "iab_tier1_name": "Technology & Computing",
  "iab_tier2_id": "599",
  "iab_tier2_name": "Computing",
  "commercial_intent": "none",
  "funnel_stage": "none",
  "brand_safe": true,
  "ad_eligible": false
}
```

**Not ad-eligible — companion conversation** (`visor-chat-ad-intent-v1-00060`, train):

```json
{
  "message": "I felt jealous of a friend's update and I dont like that about myself. Can I talk without being judged? Ill try to keep this short.",
  "prior_context": "",
  "chat_app_type": "companion",
  "iab_tier1_name": "Family and Relationships",
  "iab_tier2_id": "200",
  "iab_tier2_name": "Single Life",
  "commercial_intent": "none",
  "funnel_stage": "none",
  "brand_safe": true,
  "ad_eligible": false
}
```

**Not ad-eligible, not brand-safe — medical crisis** (`visor-chat-ad-intent-v1-00425`, train):

```json
{
  "message": "I cant catch my breath and my lips feel weird. I dont know if this is a panic thing or something worse. This is urgent.",
  "prior_context": "",
  "chat_app_type": "general_assistant",
  "iab_tier1_name": "Medical Health",
  "iab_tier2_id": "287",
  "iab_tier2_name": "Diseases and Conditions",
  "commercial_intent": "none",
  "funnel_stage": "none",
  "brand_safe": false,
  "brand_safe_reason": "medical crisis",
  "ad_eligible": false
}
```

**Not ad-eligible — literary roleplay** (`visor-chat-ad-intent-v1-00193`, validation):

```json
{
  "message": "Can you be a character from a classic novel and talk to me in that voice for a short scene? Keep it literary, not a spoiler dump.",
  "prior_context": "",
  "chat_app_type": "roleplay",
  "iab_tier1_name": "Books and Literature",
  "iab_tier2_id": "48",
  "iab_tier2_name": "Fiction",
  "commercial_intent": "none",
  "funnel_stage": "none",
  "brand_safe": true,
  "ad_eligible": false
}
```

## Distributions

All figures below are counted on the published 20,080-row file. Fuller tables are in [`stats.md`](stats.md) and [`stats.json`](stats.json).

### Split, eligibility, and safety

| | Rows | Share |
| ---: | ---: | ---: |
| train | 18,072 | 90.00% |
| validation | 2,008 | 10.00% |
| ad_eligible = true | 16,324 | 81.29% |
| ad_eligible = false | 3,756 | 18.71% |
| brand_safe = true | 19,965 | 99.43% |
| brand_safe = false | 115 | 0.57% |
| has prior_context | 6,904 | 34.38% |

### Commercial intent and funnel stage

| commercial_intent | Rows | funnel_stage | Rows |
| :--- | ---: | :--- | ---: |
| none | 3,756 | none | 3,756 |
| low | 3,117 | awareness | 3,198 |
| medium | 8,335 | consideration | 10,846 |
| high | 4,872 | purchase | 2,280 |

### Chat app type

| chat_app_type | Rows | Share |
| :--- | ---: | ---: |
| general_assistant | 15,804 | 78.71% |
| companion | 1,912 | 9.52% |
| homework_helper | 1,366 | 6.80% |
| coding_assistant | 845 | 4.21% |
| roleplay | 153 | 0.76% |

### Brand-safety reasons (when `brand_safe = false`)

| Reason | Rows |
| :--- | ---: |
| illegal drugs | 35 |
| crime | 19 |
| self-harm | 17 |
| hate | 13 |
| adult | 13 |
| medical crisis | 11 |
| minors | 7 |
| **Total** | **115** |

### Message length

| | Characters | Words |
| :--- | ---: | ---: |
| Minimum | 41 | 7 |
| Median | 107 | 20 |
| Mean | 110.4 | 21.1 |
| Maximum | 227 | 49 |

### IAB tier-2 category counts

| Count | Share | Tier-2 ID | Tier 1 | Tier 2 |
| ---: | ---: | --- | --- | --- |
| 1407 | 7.01% | `146` | Education | Homework and Study |
| 1201 | 5.98% | `599` | Technology & Computing | Computing |
| 1000 | 4.98% | `200` | Family and Relationships | Single Life |
| 423 | 2.11% | `287` | Medical Health | Diseases and Conditions |
| 384 | 1.91% | `192` | Family and Relationships | Parenting |
| 280 | 1.39% | `188` | Family and Relationships | Dating |
| 280 | 1.39% | `218` | Food & Drink | Dining Out |
| 280 | 1.39% | `324` | Entertainment | Movies |
| 279 | 1.39% | `278` | Home & Garden | Home Appliances |
| 279 | 1.39% | `280` | Home & Garden | Home Improvement |
| 279 | 1.39% | `655` | Travel | Travel Locations |
| 278 | 1.38% | `30` | Automotive | Auto Buying and Selling |
| 278 | 1.38% | `632` | Technology & Computing | Consumer Electronics |
| 278 | 1.38% | `477` | Shopping | Grocery Shopping |
| 278 | 1.38% | `159` | Attractions | Outdoor Activities |
| 278 | 1.38% | `560` | Style & Fashion | Women's Fashion |
| 277 | 1.38% | `663` | Travel | Travel Preparation and Advice |
| 276 | 1.37% | `221` | Food & Drink | Healthy Cooking and Eating |
| 276 | 1.37% | `431` | Pets | Pet Supplies |
| 275 | 1.37% | `275` | Home & Garden | Gardening |
| 275 | 1.37% | `478` | Shopping | Holiday Shopping |
| 274 | 1.36% | `170` | Personal Celebrations & Life Events | Birthday |
| 274 | 1.36% | `277` | Home & Garden | Smart Home |
| 273 | 1.36% | `410` | Personal Finance | Personal Investing |
| 270 | 1.34% | `151` | Attractions | Amusement and Theme Parks |
| 270 | 1.34% | `597` | Technology & Computing | Artificial Intelligence |
| 270 | 1.34% | `248` | Hobbies & Interests | Arts and Crafts |
| 270 | 1.34% | `31` | Automotive | Auto Insurance |
| 270 | 1.34% | `34` | Automotive | Auto Repair |
| 270 | 1.34% | `179` | Attractions | Bars & Restaurants |
| 270 | 1.34% | `553` | Style & Fashion | Beauty |
| 270 | 1.34% | `53` | Business and Finance | Business |
| 270 | 1.34% | `681` | Video Gaming | Console Games |
| 270 | 1.34% | `392` | Personal Finance | Consumer Banking |
| 270 | 1.34% | `216` | Food & Drink | Cooking |
| 270 | 1.34% | `474` | Shopping | Coupons and Discounts |
| 270 | 1.34% | `425` | Pets | Dogs |
| 270 | 1.34% | `577` | Style & Fashion | Fashion Trends |
| 270 | 1.34% | `225` | Healthy Living | Fitness and Exercise |
| 270 | 1.34% | `398` | Personal Finance | Insurance |
| 270 | 1.34% | `127` | Careers | Job Search |
| 270 | 1.34% | `147` | Education | Language Learning |
| 270 | 1.34% | `338` | Entertainment | Music |
| 270 | 1.34% | `229` | Healthy Living | Nutrition |
| 270 | 1.34% | `148` | Education | Online Education |
| 270 | 1.34% | `684` | Video Gaming | PC Games |
| 270 | 1.34% | `405` | Personal Finance | Personal Debt |
| 270 | 1.34% | `415` | Personal Finance | Personal Taxes |
| 270 | 1.34% | `451` | Real Estate | Real Estate Buying and Selling |
| 270 | 1.34% | `452` | Real Estate | Real Estate Renting and Leasing |
| 270 | 1.34% | `416` | Personal Finance | Retirement Planning |
| 270 | 1.34% | `551` | Sports | Sports Equipment |
| 270 | 1.34% | `664` | Travel | Travel Type |
| 270 | 1.34% | `165` | Personal Celebrations & Life Events | Wedding |
| 247 | 1.23% | `137` | Education | College Education |
| 208 | 1.04% | `48` | Books and Literature | Fiction |
| 205 | 1.02% | `640` | Entertainment | Television |
| 203 | 1.01% | `125` | Careers | Career Advice |
| 198 | 0.99% | `269` | Hobbies & Interests | Games and Puzzles |
| 189 | 0.94% | `484` | Sports | American Football |
| 189 | 0.94% | `683` | Video Gaming | Mobile Games |
| 189 | 0.94% | `130` | Careers | Remote Working |
| 189 | 0.94% | `231` | Healthy Living | Weight Loss |
| 189 | 0.94% | `232` | Healthy Living | Wellness |
| 188 | 0.94% | `547` | Sports | Basketball |
| 40 | 0.20% | `pg0WhF` | Sensitive Topics | Illegal Drugs, Tobacco, eCigarettes, Vaping, Alcohol |
| 21 | 0.10% | `XtODT3` | Sensitive Topics | Crime & Harmful Acts to Individuals, Society & Human Right Violations |
| 17 | 0.08% | `I4GWl6` | Sensitive Topics | Death, Injury, or Military Conflict |
| 13 | 0.06% | `Rm3SiT` | Sensitive Topics | Adult & Explicit Sexual Content |
| 13 | 0.06% | `HxqYV1` | Sensitive Topics | Hate Speech and Acts of Aggression |

Sensitive-topic labels are intentionally small and distinct so the corpus is not padded with near-duplicate unsafe paraphrases.

## Suggested tasks for AI chat monetization

This release is built for text classification. Natural tasks, using `message` (and, optionally, `prior_context`) as input:

- **Ad eligibility** — binary classification of `ad_eligible`
- **Brand safety** — binary `brand_safe`, or multiclass `brand_safe_reason` including the safe/empty class
- **Commercial intent** — four-way `none` / `low` / `medium` / `high`
- **Funnel stage** — four-way `none` / `awareness` / `consideration` / `purchase`
- **IAB topic** — 70-way `iab_tier2_id` (or a hierarchical tier-1 then tier-2 setup)
- **Joint models** — a single encoder with multiple heads, or a cascade that predicts safety/eligibility first and topic/intent only when eligible

A simple baseline, with no score reported here: train a linear classifier (for example TF-IDF plus logistic regression) or a small pretrained encoder on the `train` split only, and evaluate on `validation`. Report accuracy and macro-F1 for multiclass heads, and precision, recall, and F1 for `ad_eligible=false` and `brand_safe=false` (the minority classes that matter for “when not to serve”). Do not tune on the validation set if you want a clean number, and do not invent a test split by reshuffling `train`. This package does not include published model scores.

## Methodology

The methods below produce labels for the unit that matters in AI chat monetization: a single chat turn, not a user profile.

### Synthetic generation

Messages were written as US-English chat utterances from hand-authored templates per IAB tier-2 category and per non-commercial or unsafe situation; slot lexicons (US cities, prices in USD, products, retailers, timeframes, and similar); a constrained style layer (optional typo or slang, casing, light multi-turn prefixes); and optional synthetic `prior_context` (6,904 rows, 34.38%). No production logs were used. Lexicons use generic or fictional references, not real personal identifiers.

### IAB Content Taxonomy 3.0 mapping

Every `iab_tier1_*` and `iab_tier2_*` pair was validated against the official TSV in [InteractiveAdvertisingBureau/Taxonomies](https://github.com/InteractiveAdvertisingBureau/Taxonomies) (`Content Taxonomies/Content Taxonomy 3.0.tsv`): the ID exists, names match exactly, the tier-2 parent is the stated tier-1, and the node is a true tier-2 (no tier-3/4). Taxonomy terms remain subject to IAB Tech Lab terms.

### Labeling rubric

Labels are assigned with the template, not guessed after the fact. The rules in [`labels.md`](labels.md) are the source of truth. Price tiers, retailers, and style intensity are category-aware so that, for example, unsafe turns do not receive shopping clauses or joke suffixes.

### Deduplication

Rows were filtered with exact match after Unicode normalization, lowercasing, and punctuation stripping, plus within-category near-duplicate checks (token Jaccard and character shingles). The released file has 20,080 unique normalized messages, equal to the row count.

### Quality assurance (measured)

On the final 20,080-row release, a programmatic validator reported **0** violations of: official IAB ID/name/parent consistency; intent, funnel, brand-safety, and eligibility consistency rules; no leftover `{slot}` markers; no email, phone, or Social Security number patterns; unique IDs and unique normalized messages; and `locale=en-US` plus `source=synthetic` on every row.

A stratified 50-row self-review (including unsafe, non-commercial, and ad-eligible rows) was used during development to find systematic template errors. Fixes included removing commercial clauses and commercial priors from ineligible or unsafe turns, tightening article and price-tier filling, blocking slang insertion into thousands-separators, and restricting retailers used in crime-related examples to physical stores. Residual template artifacts are listed under Limitations.

This release does not include a multi-annotator agreement study, so no inter-annotator statistics are reported. What is reported is inter-label consistency of the encoded rules: **0 / 20,080** violations on the published file.

## Datasheet

Following Gebru et al., *Datasheets for Datasets*.

### Motivation

- **For what purpose was the dataset created?** To support research on contextual topic, intent, and ad-eligibility classification for US AI chat monetization, including when **not** to serve an ad. This dataset is a public sample of the data the Visor team uses to train its ad-matching models.
- **Who created it?** Visor Research / the Visor team. Project: [Visor](https://visor.host).
- **Who funded it?** Created and released by Visor.

### Composition

- **What do the instances represent?** One synthetic user message (optional prior turn) plus labels.
- **How many?** 20,080 instances; 70 IAB tier-2 classes; 18.71% not ad-eligible; 0.57% not brand-safe.
- **Is the data complete?** No. It is a v1 sample of English (US) chat-like text, not a census of live traffic.
- **Are there errors or noise?** Yes. See Limitations (template artifacts, single-label simplification, US-English bias).
- **Does it identify people?** No. Messages are synthetic and were screened for common personal-data patterns.
- **Is it confidential or offensive?** It contains synthetic unsafe examples (self-harm, hate, crime, adult requests, minors-related policy cases) so classifiers can learn to suppress ads. Those rows are labeled `brand_safe=false` and `ad_eligible=false`.

### Collection process

- **How was the data acquired?** Programmatic generation from written templates and lexicons, not collection from apps.
- **Over what timeframe?** Built and released 25 November 2026.
- **Was ethical review required for human subjects?** No human-subject data were collected.

### Preprocessing and labeling

- Template fill, style variation, label inheritance from the template rubric, taxonomy validation, deduplication, and consistency checks, as described in Methodology.
- Splits are a stratified random 90/10 train/validation partition (seed-fixed) by category and `ad_eligible`.

### Uses

- **Intended uses:** research and development of text classification for IAB topic, commercial intent, funnel stage, brand safety, and ad eligibility in US AI chat; academic comparison; evaluation of contextual (not profile-based) models. This dataset is a public sample of the data the Visor team uses to train its ad-matching models.
- **Out of scope:** training systems to produce ads that imitate the assistant; targeting people rather than chat context; using unsafe examples as how-to content; claiming the labels are a legal or clinical determination; using the set as if it were live Visor traffic; describing or reverse-engineering Visor's production ad-matching system from this file.

### Distribution

- **License:** MIT, for the messages, labels, and Visor documentation in this package.
- **How will it be distributed?** Public research release (Hugging Face, Kaggle, and/or Zenodo and GitHub, as published by Visor Research).
- **IAB terms:** category names and IDs are used for labeling and remain subject to IAB Tech Lab terms.

### Maintenance

- **Who maintains it?** Visor Research / the Visor team, via [Visor](https://visor.host).
- **Updates:** versioned releases (see Versioning). Breaking label-schema changes will increment the major version.
- **Contact:** [Visor](https://visor.host) or [AI chatbot ads](https://visor.host/advertisers). No personal inboxes are published with this dataset.

## Brand-safety approach

The brand-safety layer in this dataset is a **turn-level eligibility decision**, not a page rating and not a clinical triage tool.

- Unsafe turns are labeled `brand_safe=false` with a short reason, and are never `ad_eligible`.
- Brand-safe but non-commercial turns (homework, debugging, ordinary venting, non-crisis medical questions) stay `brand_safe=true` and `ad_eligible=false`, so a model can learn to withhold ads without treating those chats as unsafe content.
- Unsafe examples are synthetic, limited in count (115 rows), and not paraphrase-padded. They are present so a classifier can learn suppression, not so the corpus can be used as a source of harmful instructions.
- Publishers may block additional categories (for example alcohol, dating, or finance) in a live product. This file does not encode every publisher blocklist.

If a message in this file were a real crisis, the right response would be to seek emergency help, not to serve or discuss an ad. In the United States, the Suicide & Crisis Lifeline is 988.

## Limitations and biases

- **Synthetic, not live traffic.** Fluency and topical coverage reflect templates and lexicons. Real chat is messier, more code-switched, and more multi-intent.
- **Template artifacts remain.** Some slot combinations are slightly awkward (for example a time phrase that is not grammatical in every frame, or a homework subject that is an imperfect fit for a writing task). These are a minority of rows and are a known weakness of this release.
- **Single primary label.** Mixed-intent messages receive one IAB pair.
- **US English only.** Prices are in US dollars. Geographic and cultural coverage is US-centric.
- **Class imbalance.** Commercial consideration is the largest funnel class. Roleplay is the smallest chat-app type (153 rows, 0.76%). Brand-unsafe rows are few (115) by design: enough to learn a “do not serve” class, not enough to study rare-abuse subtypes in depth.
- **Intent and eligibility are rubric labels**, not measured conversions or policy-legal rulings.
- **Self-review, not a formal inter-annotator study.** A 50-row stratified review guided systematic fixes; a larger human review is recommended before any production use of a model trained on this file.
- **Unsafe content is present** so models can refuse ads. Do not treat those prompts as instructions.

## Ethics and intended use

- Synthetic only: **no real user data and no PII**.
- The intended research use is AI chat monetization: learning when a contextual ad may appear after a turn, and when it must not.
- This dataset is a public sample of the data the Visor team uses to train its ad-matching models. That does not make the messages real user logs.
- Unsafe classes exist solely to train and evaluate **suppression** of ads (and related safety classifiers).
- Do not use this corpus to build undisclosed native ads, to profile people, to generate harmful content, or to infer the internals of Visor's ad-matching system.
- Brand-safety labels are research labels, not a substitute for a publisher's policy, a clinician, or emergency services.
- If you find a row that looks like real personal data, or a labeling error, report it through [Visor](https://visor.host).

## Versioning

**v1.0.0 — 25 November 2026.** First public release: 20,080 synthetic, deduplicated, IAB 3.0-labeled rows; 70 tier-2 categories; train/validation split; documentation and citation files.

Breaking changes to the label schema will increment the major version. See [`CHANGELOG.md`](CHANGELOG.md).

## How to cite

Please credit **Visor Research / the Visor team** and this dataset if you use it. A machine-readable [`CITATION.cff`](CITATION.cff) is included in the package.

```bibtex
@misc{visor_ai_chat_monetization_dataset_v1,
  title        = {Visor AI Chat Monetization Dataset v1},
  author       = {{Visor Research}},
  year         = {2026},
  howpublished = {\url{https://visor.host}},
  note         = {Version 1.0.0. This dataset is a public sample of the data the Visor team uses to train its ad-matching models.}
}
```

## License

MIT. See [`LICENSE`](LICENSE). IAB Content Taxonomy names and IDs are used for labeling and are not relicensed by that file. They remain subject to IAB Tech Lab terms.

## Contact

- [AI chat monetization](https://visor.host)
- [AI chatbot ads](https://visor.host/advertisers)

Please do not use personal email addresses when citing or contacting the project about this dataset. Released by Visor Research / the Visor team.

---

*Messages in this dataset are synthetic. This dataset is a public sample of the data the Visor team uses to train its ad-matching models.*
