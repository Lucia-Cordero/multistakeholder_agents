# Same conclusion, stronger proof: a case study in agentic evidence review

## The question

Agentic systems are showing up across biotech and AI-for-bio products, often justified by the assumption that more tools and more freedom to search make an output better. A 2025 study evaluating multi-agent systems in clinical therapy recommendations found otherwise: a single, well-reasoning model matched a full multi-agent setup on the same task ([arXiv:2507.10911](https://arxiv.org/abs/2507.10911)).

This project asks a narrower, more specific version of that question. Given a fixed evidence package and a fixed panel of expert reviewers, what does letting one of them search repeatedly, rather than once or not at all, actually change? Not whether agents are better in general, but what, specifically, repeated search buys you when it doesn't change the underlying conclusion.

The domain is deliberately a hard one: therapeutic risk assessment, a judgment call built from incomplete, sometimes conflicting evidence, not a fact-lookup task with a single correct answer waiting to be found.

## The case, and why it needed a contamination check first

The case is TRANSEURO (NCT01898390), a European trial redesigning fetal ventral mesencephalic tissue transplantation for Parkinson's disease. Grafting began in 2015, following two earlier NIH-funded trials (Freed 2001, Olanow 2003) that established graft-induced dyskinesia as a serious, unresolved risk. TRANSEURO reported its actual results in *Nature Biotechnology* in 2025.

Any project asking an LLM to assess a historical decision has an obvious problem: the model may already know how it turned out. Before building any evidence package, every candidate case in this project was checked directly, in a fresh, tool-free conversation, for what the model already recalled about the outcome.

TRANSEURO failed this check. Asked directly, the model recalled the trial's general design lineage and the approximate direction of its result, correct on substance, imprecise on specifics like the exact publication venue and date. That pattern, right shape, wrong details, is the signature of genuine contamination, not a lucky guess. Because of this, TRANSEURO's evaluation compares what each condition actually claimed, not whether the model happened to predict the correct outcome. The real question, run by run, was whether repeated search surfaced a new concern, or a better-supported version of an existing one, than a single search did, checked by reading outputs side by side and spot-checking whether the citations behind a given claim actually said what the claim said they did. A second case, Aspen Neuroscience's ASPIRO trial, passed the same check cleanly and was reserved for outcome-prediction testing instead (not covered in this write-up).

## Building the evidence package

The evidence cutoff is December 31, 2014, just before grafting began. Everything the reviewers see was published on or before that date.

The corpus was built from three independently pre-specified PubMed queries, decided and written down before any results were inspected, not adjusted afterward based on what came back:

- **Program-specific**: TRANSEURO and its lead investigator's other work
- **Disease and modality, general**: fetal ventral mesencephalic transplantation broadly, 1970 onward, wide enough to catch the field's foundational work
- **Field mechanism**: graft-induced dyskinesia specifically, the central risk the redesign was built to address

Selection from each query used the union of the top 10 results by relevance rank and the top 10 by citation count, not relevance alone. Relevance ranking alone buries older, foundational papers behind decades of more recent, more densely-matching literature, in early testing it placed Olanow's landmark 2003 trial 82nd out of 288 results for its own search term. Citation count corrects for that but under-weights genuinely relevant recent work. Using both catches what either alone would miss.

The date cutoff is enforced twice, not once. PubMed's own query-time date filter let two papers with true publication dates in 2015 pass through undetected during testing, an unexplained mismatch between the field the query filters on and the field the record actually reports. Every result is re-checked against its own reported publication date before being allowed into the corpus, and any date-filtered API call is treated as a first pass, not a guarantee.

No paper was added by hand. Every source in the corpus came from these three automated queries, deliberately, to avoid a very specific and easy-to-miss failure mode: manually adding papers you already know were important, because you know how the case turned out, quietly imports the outcome into the evidence package before the test even begins.

Full-text access was checked via PMC and Unpaywall for every paper in the corpus, mainly to see how much of the relevant literature was actually open. That check itself surfaced a real constraint: a large share of the corpus, including some of its most important papers, had no legitimately accessible full text anywhere. Rather than build a frozen context mixing full papers for some sources and abstracts for others, every condition was given abstract-only text, uniformly, across the whole corpus. Two reasons drove that choice. An uneven mix would have handed the model disproportionately more detail on whichever papers happened to be open, for reasons of publisher policy, not scientific importance, quietly biasing every reviewer's reasoning toward those sources. And processing full text for the papers that were accessible wouldn't have closed the access gap for the ones that weren't, at a real added token cost for no consistent gain. This is a genuine limitation of the evidence base, not a strength: abstracts omit the granular safety data, discussion of limitations, and full methodological detail that a real 2014 reviewer with institutional access would likely have had in front of them.

## Four conditions, one real comparison

Four reviewer personas, Biology, Clinical, Regulatory, CMC/Manufacturing, were given the identical evidence package and asked the identical question: is this program's redesign well-justified to proceed, given everything known as of the cutoff. A fifth, Commercial, was dropped: TRANSEURO is an academic, EU-funded consortium trial, not a company-sponsored program, and the persona had no real decision to make.

The four conditions vary exactly one thing at a time:

| Condition | Personas | Tool access | Called in this write-up |
|---|---|---|---|
| `week1` | Single call, all four sections | None | *(persona-separation check only)* |
| `week2` | Four independent calls | None | "No search" |
| `week3` | Four independent calls | Capped at one search round | "One search" |
| `week4` | Four independent calls | Unlimited search rounds | "Repeated search" |

The core comparison is `week3` against `week4`. Both have identical personas, identical evidence, identical tool, identical instructions, the only difference is whether a second search is structurally possible at all. `week2` against `week4` would have been a weaker test: it changes two things at once, whether a tool exists and whether it can be used iteratively, and any difference in outcome couldn't be attributed to either one specifically.

Each condition was run four times independently, given how much sampling variance a single run can carry.

## What actually happened

The only condition where "search rounds used" is a meaningful, varying number is `week4`, repeated search. `week3` is structurally capped at exactly one round, and `week1`/`week2` have no search tool at all, so the number there is always zero by design, not by choice. The table below is `week4` data, four independent repeats of the full experiment:

| Reviewer (week4, repeated search) | Run 1 | Run 2 | Run 3 | Run 4 |
|---|---|---|---|---|
| Clinical | 1 | 1 | 1 | 1 |
| Regulatory | 1 | 1 | 1 | 1 |
| CMC / Manufacturing | 3 | 2 | 2 | 2 |
| Biology | 1 | 1 | 3 | 1 |

Across three of the four personas, Clinical, Regulatory, and Biology in three of its four runs, one search round was sufficient every time; the model never chose to search again, even though `week4` placed no cap on how many it could use. CMC/Manufacturing was the exception: it used two to three search rounds in every single run, no exceptions.

CMC's repeated concern, across every condition and every run, was the same: no evidence anywhere in the package, or found through any search, that fetal tissue procurement and preparation had been solved at a scale the trial actually required. In "no search" conditions, this appeared as a plainly stated gap. Under repeated search, the reviewer tried several distinct phrasings, general procurement standardization, a specific named tissue-collection consortium, donor supply logistics, all returning nothing, before stating the absence directly.

TRANSEURO's actual 2025 outcome paper confirms this was the real problem: tissue shortage forced the team to cancel planned surgeries, and remained an unresolved source of variability through the end of the trial. One case, not a general pattern, but a real one.

**The conclusion did not change between one search and several.** What changed was how it was earned. A single search lives or dies by how well it happens to be phrased, a narrow or incomplete query can miss something genuinely findable (a risk demonstrated directly, earlier in this same project, when a single-term ClinicalTrials.gov query missed a real trial that a broader, differently-phrased query caught). Repeated search protects specifically against that failure mode. It is not a mechanism for discovering a different verdict when the honest answer is a judgment call rather than a hidden fact.

That distinction is the actual finding: *not provided in the evidence* became *searched once, not found* became *searched several ways, consistently not found*, the same claim, moving from an unverified gap to a verified absence.

## Limitations, stated plainly

This is one case, evaluated four times per condition, not a general claim about agentic search. A different case, with a genuinely unresolved evidence base rather than a well-covered one, could show repeated search changing a conclusion rather than just its confidence, and this project doesn't rule that out.

The contamination check itself has a known limitation: a general "what do you know about this" question may not surface knowledge that a more specific, task-embedded prompt would leak. Passing the check is reassuring, not proof of a clean model state.

Week 3's saved reasoning traces are sparse for most runs after the first, several are empty, a structural consequence of that condition's final call being explicitly told to stop searching and answer, rather than a gap in the answer's actual reasoning quality, which was checked directly against Week 4's output and held up.

Every reviewer worked from abstracts, not full papers, for reasons explained above. This means the evidence available here is thinner than what a real 2014 reviewer with institutional journal access would likely have had, and some of the reasoning in these results may be more conservative than it would be with full methodological detail in front of it.

## What's in this repository

- `multistakeholder_agents.ipynb`, the full corpus-building and experiment pipeline
- `Results/`, every saved reviewer output and reasoning trace, across all four conditions and all runs, unedited
- `results_index.md`, mapping each file in `Results/` to its condition and run, so any claim above can be checked directly against the source it came from
- `transeuro_frozen_context.txt`, the exact evidence package every condition read from
