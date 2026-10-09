# Truth_Santa_Hackathon
All documets and code related to Truth Santa Hackathon

Truth Santa: Project Proposal
A region-aware nutrition and health claim verification platform
 Status: Draft v1 | region -> UK
1. Executive Summary
Truth Santa is a web application that lets a user type or speak a nutrition or health question (for example, "Is vitamin D good for preventing colds?"), choose their region, and receive a structured, claim-level verdict in the form Claim | Evidence | Confidence. The system retrieves region-relevant official guidelines (NHS, WHO, national dietary guidelines) and scientific literature, analyses the evidence with a chain of specialised agents, and returns a transparent confidence rating with citations and plain-language explanations.
The core idea is to separate two things that existing tools blur together: what the evidence says (from studies) and what your region's guidelines recommend (which also reflect local food, resources and values). Both are shown side by side, and the system is designed to say "not enough studies for this region" rather than "no effect" when evidence is missing.

2. Problem and Motivation
* Nutrition and health misinformation spreads quickly, and many claims are technically true in one population but misleading in another.
* Most nutrition research comes from high-income countries, so users elsewhere are often given evidence that may not transfer to their diet, genetics or health context (what GRADE calls indirectness).
* Existing fact-checkers and general chatbots rarely show how confident they are, which evidence they used, or whether it applies to the user's region.
* A confidently wrong "High" rating on a health claim is the most harmful failure mode, so reliability and calibration must be central to the design rather than an afterthought.

3. Aim and Objectives
Aim: Build and rigorously evaluate a region-aware, citation-grounded claim verification system for nutrition and health questions.
Objectives
1. Convert free-text or voice questions into testable claims using the PICO structure (Population, Intervention/exposure, Comparator, Outcome).
2. Build a metadata-rich corpus of regional guidelines and scientific literature, searchable by semantic and keyword retrieval.
3. Design a multi-agent pipeline that retrieves, extracts, scores and explains evidence.
4. Produce an interpretable confidence score that accounts for source authority, evidence strength, relevance, regional applicability, recency, agreement and retrieval quality.
5. Validate the ratings against expert-graded claims using a contamination-resistant benchmark.
6. Test whether the tool actually helps users judge claims more accurately, and whether voice input is fair across accents and regions.

4. Proposed System
4.1 User flow
1. User enters a text or voice question and selects a region.
2. The system confirms its interpretation of the claim (including numbers and units, read back for voice input).
3. Evidence is retrieved and analysed.
4. The user sees a table of Claim | Evidence | Confidence, plus the region's guideline position, citations and a short explanation.
4.2 Chain of agents
Agent	Role
Orchestrator	Coordinates the pipeline, handles retries and refusals ("not yet rated")
Context Checker	Checks the question is in scope, clarifies vague claims, parses into PICO
Domain Mapper	Maps the claim to health topic, population, age group and relevant evidence types
Multilingual Support	Handles input and source documents in other languages, reducing language bias
Rubrics Agent	Applies the evidence-grading rubric and computes the confidence factors
Checker Agent	Verifies that every statement is supported by a cited source (faithfulness) and flags unsupported claims
4.3 Data and metadata
Every document ingested carries a standard metadata record: document_id, title, source, country, region, organisation, document_type, publication_date, last_updated, language, health_topic, population, age_group, evidence_level, source_authority and url. For example, an NHS vitamin D guideline would be tagged United Kingdom / UK Government / NHS, adults, clinical/public health guideline, authority score High.
4.4 Retrieval
Hybrid retrieval combining semantic search (embeddings) and keyword search, filtered and re-ranked using the metadata above (region, population, recency, document type). A documented search strategy, in the spirit of PRISMA, will be followed to limit selection bias and language bias.
4.5 Confidence model
The working formulation is:
Confidence = Source Authority × Evidence Strength × Relevance × Regional Applicability × Recency × Agreement × Retrieval Quality
Each factor will be scaled to a defined range and the combined score mapped to ordered bands (for example Low / Moderate / High). Two design points will be examined during the project: a pure product can collapse quickly when one factor is low, so alternatives (geometric mean, weighted combination, rule-based caps) will be compared; and the numeric score will always be shown alongside the effect size, interval and an absolute-risk statement where available, not as a badge alone.
4.6 Technology
Python, a RAG pipeline with a vector store plus keyword index, LLM-based agents, speech-to-text for voice input, and a web front end (website or browser extension).
5. Key Research Questions
The proposal is organised around six open design questions. Each has a proposed approach and an evaluation method.
RQ1. Should region change the confidence rating, or only the guideline context?
If evidence comes from one country, should it carry the same confidence for a question about another country, and how should evidence transfer between regions?
* Proposed approach: Keep two outputs: an evidence confidence and a regional guideline position. Treat region as an indirectness adjustment in evidence confidence, down-weighting studies whose population, baseline diet or setting differs from the user's region, using a transparent mapping (for example, same region, similar region, different region) rather than a hidden penalty.
* Evidence gaps: When a region has little or no research, report "insufficient evidence for this region" and show the closest transferable evidence, clearly labelled, never "no effect".
* To decide: the down-weighting factors and any minimum number of regional studies. These will be tested through sensitivity analysis rather than fixed in advance.
RQ2. Is counting independent clusters acceptable, or must we weight by precision?
* Problem: Papers analysing the same cohort, or overlapping meta-analyses, share information, so the effective sample size is smaller than the paper count suggests.
* Proposed approach: Define an evidence cluster as a group of publications sharing a dataset or cohort (with research group and funder as secondary flags), and count clusters rather than papers. Compare this rubric against precision-weighted pooling (with I² for heterogeneity) on a set of topics with published meta-analyses, and report where they disagree.
* Output: a justified hybrid, for example cluster-based independence checks feeding into precision-weighted summaries, with sensitivity to the thresholds reported.
RQ3. Which numbers must be extracted, and how do we prove extraction is accurate enough?
* Minimum fields: effect measure and type (RR, OR, HR, mean difference), confidence interval, sample size, population, exposure/dose with units, comparator, outcome, study design and follow-up.
* Proposed approach: Build a manually extracted gold set, measure field-level precision, recall and exact-match accuracy, and run explicit unit-conversion checks (a common source of error). Set accuracy thresholds per field, and route low-confidence extractions to a "not rated" outcome instead of guessing.
RQ4. How do we build and size a validation benchmark without contamination?
* Problem: LLMs may have seen published reviews during training, inflating results.
* Proposed approach: Build the benchmark from systematic reviews and guidelines published after the model's training cutoff, and keep a held-out private test set that never enters prompts, examples or retrieval indexes. Also run a retrieval recall test: hide a review's conclusions and check what share of its included studies the system recovers.
* Metrics: weighted kappa against expert grades (ratings are ordered), rerun consistency, and a safety metric: of everything labelled High, the share experts also label High (a false High is the worst error). Results will be reported separately by region and claim type, with benchmark size chosen by a power or precision calculation on the safety metric.
RQ5. How do we measure whether users judge claims better with the tool?
* Proposed approach: A randomised A/B study: one group sees claims with ratings and citations, the other without. Measure judgement accuracy, trust calibration and intended sharing behaviour, and watch for backfire effects. Include a comprehension test comparing absolute risk ("2 in 1,000 to 3 in 1,000") against relative risk ("50% higher").
* Answer quality: Separately sample AI answers for faithfulness, citation accuracy and correct refusals, using two human reviewers and reporting inter-rater agreement (Cohen's kappa). Sample size will be set by a power calculation for the primary outcome.
RQ6. How do we measure the fairness of voice recognition across accents and regions?
* Proposed approach: Measure word error rate (WER) by accent, language, region and noise condition on a recorded test set, with extra attention to numbers and units ("15 mg" versus "50 mg"). Compare WER gaps between groups and test whether a read-back confirmation step reduces dangerous misrecognitions.
* Audio uncertainty: Test spoken phrasing of confidence (for example, "moderate confidence, based on limited evidence") for comprehension, since there is no table or colour in audio.
6. Evaluation Summary
Component	Metric	Target to be agreed
Claim parsing (PICO)	Agreement with expert PICO (kappa)	TBD
Retrieval	Recall of studies in reference systematic reviews	TBD
Extraction	Field-level accuracy vs manual extraction	TBD
Confidence ratings	Weighted kappa; consistency on rerun	TBD
Safety	Precision of High ratings vs experts	TBD
Grounded answers	Faithfulness, citation accuracy, correct refusal rate	TBD
Voice	WER by accent and region	TBD
User impact	Judgement accuracy and trust calibration (A/B)	TBD
7. Scope and Phasing
Phase 1 (core prototype): Text input, a small set of health topics (for example vitamin D) and two or three regions (starting with the UK/NHS and WHO), the full confidence pipeline, and the benchmark evaluation.
Phase 2: Voice input, multilingual support, more regions and topics, and the user study.
Out of scope: Personalised medical advice or diagnosis. Truth Santa assesses claims against evidence and is not a substitute for a clinician.
8. Indicative Work Plan
Stage	Activities	Duration (adjust to your deadline)
1. Foundations	Finalise claim schema, metadata schema, topic and region selection, rubric draft	2 weeks
2. Data and retrieval	Ingest guidelines and literature, build hybrid retrieval, test recall	3 weeks
3. Agents and extraction	Build agent chain, extraction module, gold-set evaluation	3 weeks
4. Confidence model	Implement scoring, cluster logic, sensitivity analysis	2 weeks
5. Interface	Web front end, citations and explanation display, read-back for voice	2 weeks
6. Validation	Benchmark, expert grading, safety metric, voice WER	3 weeks
7. User study and write-up	A/B study, analysis, final report	3 weeks
9. Risks and Mitigations
Risk	Mitigation
False High ratings on health claims	Conservative thresholds, safety metric as a primary target, "not yet rated" refusals
Sparse regional evidence	Explicit "insufficient evidence here" wording and labelled transferable evidence
Benchmark contamination	Post-cutoff reviews and a private held-out set
Extraction and unit errors	Field-level checks, unit validation, human review of low-confidence cases
Language and popularity bias in retrieval	Documented search strategy, multilingual retrieval, regional sources
Rating flip-flopping on new studies	Change thresholds and cumulative meta-analysis to separate signal from noise
Over-trust in the tool	Show uncertainty clearly, test trust calibration
10. Ethics, Privacy and Safety
Health questions are sensitive. The tool will display a clear not-medical-advice notice, log only what is needed for evaluation (minimised, with consent, and de-identified where possible), avoid storing voice recordings beyond the study need, and be transparent about its limitations and sources. Ethics approval will be sought for the user study and any recorded voice data.
11. Expected Outcomes and Deliverables
* A working prototype (web app or extension) producing Claim | Evidence | Confidence outputs with citations.
* A documented, metadata-rich evidence corpus and retrieval pipeline.
* A justified confidence methodology, including the region and independence decisions from RQ1 and RQ2.
* A contamination-resistant validation benchmark and results report.
* Findings from the user study and the voice fairness analysis.
* A final report and demonstration.
12. Open Items to Confirm
* Project deadline and team size, to fix the timeline.
* Initial health topics and regions for Phase 1.
* Whether the deliverable is a website or a browser extension.
* Access to expert graders for the benchmark.
