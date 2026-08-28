<div style="page-break-before: always;"></div>

# 8. Results & Discussions

## 8.1 AI Adoption Solves a Real, Documented Scale Problem

The analysis confirms the industry premise in Section 3.2: manual, keyword-based resume review does
not scale. Rapid year-over-year growth in AI adoption (7.1) and resume screening's status as a
top-five AI use case (7.2) both validate the core business opportunity in the literature — AI-based
matching is being adopted specifically to close the gap keyword-only ATS filtering leaves open.

## 8.2 A Weighted Scoring Model Is More Transparent and Defensible

Unlike a single opaque "AI match %," a four-factor weighted score (7.3) lets a recruiter see *why* a
candidate ranked highly — an important trust factor, since opaque AI scoring is a common criticism of
HR-tech products in the literature. This transparency is a genuine strength of weighted scoring as an
approach, independent of any specific implementation.

## 8.3 Adoption Is Outpacing Quality Assurance in Part of the Industry

The finding that 19% of AI-using organizations report their tools overlooked qualified applicants
(7.1) is a meaningful caution against treating AI screening as fully autonomous. Read alongside
resume screening trailing content-generation tasks in AI-adoption share (7.2), the literature
suggests the industry is more comfortable using AI to *assist* recruiters than to *replace* their
judgment — an important distinction for how such systems should be designed and evaluated.

## 8.4 Security and Cost Governance Are Necessary, Not Optional

Given that recruitment platforms hold personal data across multiple user roles, and that AI API calls
carry a real, variable cost (7.4), the literature is consistent that role-based access control,
defensive middleware, and AI usage tracking with a fallback path are baseline requirements for any
platform in this category (5.4–5.5), not later additions.

## 8.5 Bias Risk Requires an Active, Ongoing Response

Documented bias cases (5.6) show AI screening bias is a recurring, observed failure mode, and
peer-reviewed evidence (5.7) shows it is also a *measurable* one: An et al. (2025) quantify a 1–3
percentage-point swing in modeled hiring probability along racial and gender lines across five major
LLMs, not an isolated anomaly. Combined with binding precedent such as NYC's annual audit mandate
(5.8), this shows a defensible AI recruitment system needs an explicit, periodic bias-review
process — not a one-time check at launch, and increasingly not an optional one — a point developed
further in Section 9.

## 8.6 Summary of Findings

| Finding | Evidence | Assessment |
|---|---|---|
| AI-assisted screening addresses a genuine scale problem | 69% adoption, +18pp YoY (Section 7.1) | Confirmed |
| Weighted, multi-factor scoring improves transparency | 50/25/15/10 weighting pattern (Section 7.3) | Strength of the approach |
| Full automation is not yet the industry norm | Screening trails content-generation use cases (Section 7.2) | Human-in-the-loop still expected |
| AI screening can miss qualified candidates | 19% of adopters report this (Section 7.1) | Caution — quality assurance needed |
| Bias is a documented, measured, and regulated risk | Amazon case, An et al. (2025), NYC LL 144 (5.6–5.8) | Requires active mitigation |
