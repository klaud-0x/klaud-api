---
title: How to Build an AI Research Agent with 3 API Calls
published: false
description: Build a research agent that searches PubMed, finds arXiv preprints, and looks up drug candidates — all with free JSON APIs, no auth required.
tags: ai, api, tutorial, research
cover_image:
---

# How to Build an AI Research Agent with 3 API Calls

Most "AI agent" tutorials start with 200 lines of LangChain boilerplate. This one starts with `curl`.

We're going to build a research agent workflow that:

1. Searches **PubMed** for recent papers on a biomedical topic
2. Finds related **arXiv** preprints for computational angles
3. Looks up **drug candidates** targeting the genes/proteins discovered

Three API calls. Zero API keys. Real data.

## The API

[Klaud API](https://klaud-api.klaud0x.workers.dev) is a free JSON API built for AI agents. It wraps PubMed, arXiv, ChEMBL, and other sources into clean GET endpoints. No signup, no auth tokens — just HTTP.

We'll use three endpoints:

| Endpoint | Source | What it does |
|----------|--------|-------------|
| `/api/pubmed` | NCBI PubMed | Search biomedical literature |
| `/api/arxiv` | arXiv.org | Search preprints by topic/category |
| `/api/drugs` | ChEMBL | Look up drugs by name or target gene |

---

## Step 1: Search PubMed

Let's say our agent is investigating **GLP-1 receptor agonists and neurodegeneration** — a hot topic since semaglutide started showing neuroprotective signals.

```bash
curl "https://klaud-api.klaud0x.workers.dev/api/pubmed?q=GLP-1+neurodegeneration&limit=3"
```

Response:

```json
{
  "query": "GLP-1 neurodegeneration",
  "count": 3,
  "total_found": 251,
  "articles": [
    {
      "pmid": "41619567",
      "title": "Molecular evidence associating GLP-1 receptor agonists and brain-derived neurotrophic factors in neurodegenerative and psychiatric disorders: A systematic review.",
      "abstract": "A converging mechanistic theme across mental disorders involves impaired neuroplasticity and reduced brain-derived neurotrophic factor (BDNF). Glucagon-like peptide-1 receptor agonists (GLP-1RAs), used for type 2 diabetes and obesity, have shown neuroprotective potential, but whether these effects are mediated by BDNF is unclear.",
      "journal": "Asian journal of psychiatry",
      "year": "2026",
      "url": "https://pubmed.ncbi.nlm.nih.gov/41619567/"
    },
    {
      "pmid": "41599176",
      "title": "Liraglutide and Exenatide in Alzheimer's Disease and Mild Cognitive Impairment: A Systematic Review and Meta-Analysis of Cognitive Outcomes.",
      "abstract": "Glucagon-like peptide-1 receptor agonists (GLP-1 RAs) exhibit neuroprotective properties in preclinical models of Alzheimer's disease (AD), reducing amyloid accumulation, neuroinflammation, and insulin resistance within the brain...",
      "journal": "Pharmaceutics",
      "year": "2026",
      "url": "https://pubmed.ncbi.nlm.nih.gov/41599176/"
    }
  ]
}
```

We now have PMIDs, titles, abstracts, and journal info. From reading these, our agent can extract a key insight: **GLP-1 receptor (GLP1R)** is the target worth investigating further.

---

## Step 2: Find arXiv Preprints

Let's search arXiv for computational work on GLP-1 receptor modeling:

```bash
curl "https://klaud-api.klaud0x.workers.dev/api/arxiv?q=GLP-1+receptor+drug+discovery&limit=3"
```

Response:

```json
{
  "query": "GLP-1 receptor drug discovery",
  "category": "all",
  "count": 3,
  "papers": [
    {
      "id": "2602.00498v1",
      "title": "Harnessing the Peripheral Surface Information Entropy from Globular Protein-Peptide Complexes",
      "authors": ["Tyler Grear", "Donald J. Jacobs"],
      "abstract": "Predicting favorable protein-peptide binding events remains a central challenge in biophysics, with continued uncertainty surrounding how nonlocal effects shape the global energy landscape...",
      "categories": ["physics.bio-ph", "cs.IT"],
      "published": "2026-01-31T03:50:26Z",
      "url": "http://arxiv.org/abs/2602.00498v1",
      "pdf": "https://arxiv.org/pdf/2602.00498v1"
    }
  ]
}
```

You can also filter by arXiv category (`cs.AI`, `q-bio.BM`, etc.) using the `cat` parameter:

```bash
curl "https://klaud-api.klaud0x.workers.dev/api/arxiv?q=peptide+receptor+binding&cat=q-bio.BM&limit=5"
```

Each result includes the paper ID, authors, categories, and a direct PDF link — everything an agent needs to decide whether to read further.

---

## Step 3: Look Up Drug Candidates

Now the payoff. We identified **GLP1R** as the target. Let's find approved drugs that hit it:

```bash
curl "https://klaud-api.klaud0x.workers.dev/api/drugs?target=GLP1R&limit=3"
```

Response:

```json
{
  "target": "GLP1R",
  "target_name": "Glucagon-like peptide 1 receptor",
  "target_type": "SINGLE PROTEIN",
  "organism": "Homo sapiens",
  "target_chembl_id": "CHEMBL1784",
  "count": 3,
  "drugs": [
    {
      "name": "LIRAGLUTIDE",
      "chembl_id": "CHEMBL4084119",
      "mechanism": "Glucagon-like peptide 1 receptor agonist",
      "action_type": "AGONIST",
      "max_phase": 4,
      "url": "https://www.ebi.ac.uk/chembl/compound_report_card/CHEMBL4084119/"
    },
    {
      "name": "EXENATIDE",
      "chembl_id": "CHEMBL414357",
      "mechanism": "Glucagon-like peptide 1 receptor agonist",
      "action_type": "AGONIST",
      "max_phase": 4,
      "url": "https://www.ebi.ac.uk/chembl/compound_report_card/CHEMBL414357/"
    },
    {
      "name": "ALBIGLUTIDE",
      "chembl_id": "CHEMBL2107841",
      "mechanism": "Glucagon-like peptide 1 receptor agonist",
      "action_type": "AGONIST",
      "max_phase": 4,
      "url": "https://www.ebi.ac.uk/chembl/compound_report_card/CHEMBL2107841/"
    }
  ]
}
```

`max_phase: 4` = FDA approved. We've gone from "what's new in GLP-1 research" to "here are the approved drugs hitting this target" in three requests.

You can also search by drug name:

```bash
curl "https://klaud-api.klaud0x.workers.dev/api/drugs?q=semaglutide"
```

---

## Chain It: The Complete Script

Here's a Python script that chains all three calls into an automated research pipeline:

```python
import requests

API = "https://klaud-api.klaud0x.workers.dev"

def research_agent(topic, target_gene):
    # Step 1: Search PubMed for recent literature
    pubmed = requests.get(f"{API}/api/pubmed", params={"q": topic, "limit": 5}).json()
    print(f"📚 Found {pubmed['total_found']} PubMed articles for '{topic}'")
    for a in pubmed["articles"]:
        print(f"   - [{a['year']}] {a['title'][:80]}...")
        print(f"     {a['url']}")

    # Step 2: Find related arXiv preprints
    arxiv = requests.get(f"{API}/api/arxiv", params={"q": topic, "limit": 5}).json()
    print(f"\n📄 Found {arxiv['count']} arXiv preprints")
    for p in arxiv["papers"]:
        print(f"   - {p['title'][:80]}...")
        print(f"     {p['pdf']}")

    # Step 3: Look up drugs targeting the key protein
    drugs = requests.get(f"{API}/api/drugs", params={"target": target_gene, "limit": 5}).json()
    print(f"\n💊 Drugs targeting {drugs.get('target_name', target_gene)}:")
    for d in drugs["drugs"]:
        phase = "✅ Approved" if d["max_phase"] == 4 else f"Phase {d['max_phase']}"
        print(f"   - {d['name']} ({d['action_type']}) — {phase}")
        print(f"     {d['url']}")

    return {"pubmed": pubmed, "arxiv": arxiv, "drugs": drugs}

# Run it
results = research_agent("GLP-1 neurodegeneration", "GLP1R")
```

Output:

```
📚 Found 251 PubMed articles for 'GLP-1 neurodegeneration'
   - [2026] Molecular evidence associating GLP-1 receptor agonists and brain-deri...
     https://pubmed.ncbi.nlm.nih.gov/41619567/
   - [2026] Liraglutide and Exenatide in Alzheimer's Disease and Mild Cognitive I...
     https://pubmed.ncbi.nlm.nih.gov/41599176/

📄 Found 3 arXiv preprints
   - Harnessing the Peripheral Surface Information Entropy from Globular Protein-...
     https://arxiv.org/pdf/2602.00498v1

💊 Drugs targeting Glucagon-like peptide 1 receptor:
   - LIRAGLUTIDE (AGONIST) — ✅ Approved
     https://www.ebi.ac.uk/chembl/compound_report_card/CHEMBL4084119/
   - EXENATIDE (AGONIST) — ✅ Approved
     https://www.ebi.ac.uk/chembl/compound_report_card/CHEMBL414357/
   - ALBIGLUTIDE (AGONIST) — ✅ Approved
     https://www.ebi.ac.uk/chembl/compound_report_card/CHEMBL2107841/
```

### Node.js version

```javascript
const API = "https://klaud-api.klaud0x.workers.dev";

async function researchAgent(topic, targetGene) {
  const pubmed = await fetch(`${API}/api/pubmed?q=${encodeURIComponent(topic)}&limit=5`).then(r => r.json());
  console.log(`📚 ${pubmed.total_found} PubMed results for "${topic}"`);
  pubmed.articles.forEach(a => console.log(`   ${a.year} | ${a.title.slice(0, 80)}...`));

  const arxiv = await fetch(`${API}/api/arxiv?q=${encodeURIComponent(topic)}&limit=5`).then(r => r.json());
  console.log(`\n📄 ${arxiv.count} arXiv preprints`);
  arxiv.papers.forEach(p => console.log(`   ${p.title.slice(0, 80)}...\n   ${p.pdf}`));

  const drugs = await fetch(`${API}/api/drugs?target=${targetGene}&limit=5`).then(r => r.json());
  console.log(`\n💊 Drugs targeting ${drugs.target_name}:`);
  drugs.drugs.forEach(d => {
    const phase = d.max_phase === 4 ? "✅ Approved" : `Phase ${d.max_phase}`;
    console.log(`   ${d.name} (${d.action_type}) — ${phase}`);
  });

  return { pubmed, arxiv, drugs };
}

researchAgent("GLP-1 neurodegeneration", "GLP1R");
```

---

## Where This Goes Next

This three-call pattern is a building block. Plug it into:

- **An LLM agent** — feed the JSON results into GPT/Claude as context, ask it to synthesize findings
- **A RAG pipeline** — index the abstracts, retrieve them when users ask questions
- **A Slack/Discord bot** — `/research EGFR lung cancer` → instant literature + drug summary
- **Automated monitoring** — cron job that checks PubMed daily for new papers on your targets

The API also has endpoints for [Hacker News](https://klaud-api.klaud0x.workers.dev) (`/api/hn`), [GitHub trending](https://klaud-api.klaud0x.workers.dev) (`/api/github`), [crypto prices](https://klaud-api.klaud0x.workers.dev) (`/api/crypto`), and [web extraction](https://klaud-api.klaud0x.workers.dev) (`/api/extract`) — same pattern, same simplicity.

---

## Try It

The API is live and free (20 requests/day, no signup):

🌐 **API:** [klaud-api.klaud0x.workers.dev](https://klaud-api.klaud0x.workers.dev)
💻 **GitHub:** [github.com/klaud-0x/klaud-api](https://github.com/klaud-0x/klaud-api)

```bash
# Try it right now
curl "https://klaud-api.klaud0x.workers.dev/api/pubmed?q=CRISPR+cancer&limit=3"
curl "https://klaud-api.klaud0x.workers.dev/api/drugs?target=EGFR&limit=3"
```

Star the repo if you find it useful. PRs welcome.
