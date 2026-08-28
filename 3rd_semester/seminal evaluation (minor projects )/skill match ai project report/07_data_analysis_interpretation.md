<div style="page-break-before: always;"></div>

# 7. Data Analysis & Interpretation

This section is applicable to the study because the secondary literature provides quantifiable
industry data — AI-adoption statistics, common-use-case breakdowns, and a typical scoring
methodology — which is analyzed below.

## 7.1 AI Adoption in Recruiting

| Metric | Value | Source |
|---|---:|---|
| HR professionals using AI in recruiting (2025) | 69% | SHRM (2025a) |
| Same metric, prior year | 51% | SHRM (2025a) |
| Organizations reporting AI screening overlooked qualified applicants | 19% | SHRM (2025b) |

**Interpretation:** The 18-point year-over-year jump shows this is a fast, active shift, consistent
with Gartner's framing of an "AI revolution" in talent acquisition (Gartner, 2025). The 19% figure for
missed qualified applicants is a material caution: adoption is outpacing quality assurance in a
meaningful share of deployments, reinforcing the safeguards discussed in Sections 5.6–5.8 and 9.

## 7.2 Most Common AI Use Cases in Recruiting

| Use case | Share of AI-using HR teams |
|---|---:|
| Writing job descriptions | 66% |
| Screening resumes | 44% |
| Automating candidate searches | 32% |
| Customizing job postings | 31% |
| Communicating with applicants | 29% |

*(Source: Society for Human Resource Management, 2025a)*

**Interpretation:** Resume screening is the second most common AI use case industry-wide, confirming
it as a mainstream application rather than a niche one. Notably, it sits below content-generation
tasks (writing job descriptions) — suggesting organizations trust AI more with drafting than with
directly filtering candidates, which is consistent with the literature's emphasis on treating AI
screening output as a ranked *shortlist* for human review, not an autonomous decision.

## 7.3 A Typical Weighted Scoring Model

A representative weighting scheme for AI-based candidate matching, drawn from common HR-tech practice
and illustrated through SkillMatch AI, is:

```
Required Skills Match       : 50%
Experience Level             : 25%
Description Semantic Match   : 15%
Nice-to-Have Skills          : 10%
```

**Interpretation:** Hard requirements (skills + experience) account for 75% of the score, while
softer/contextual signals (semantic fit + nice-to-have) make up the remaining 25%. This weighting
favors precision over recall, a defensible choice for a recruiter-facing shortlist tool where false
positives waste more reviewer time than false negatives — and it produces a score recruiters can
interpret component-by-component rather than as an opaque single number, directly addressing the
transparency concern raised in the literature (Section 5.3).

## 7.4 AI Processing Cost as a Design Constraint

Current LLM API pricing ranges roughly from $0.25–$15 per million input tokens and $1.25–$75 per
million output tokens depending on model tier (IntuitionLabs, 2025). A typical resume runs to a few
hundred tokens once parsed, so per-resume cost is small individually but scales linearly with
applicant volume — a popular posting attracting several thousand applications can generate a
non-trivial API bill.

**Interpretation:** This is why the cost-governance pattern in Section 5.4 (usage logging plus a
cheaper fallback) is a direct response to a real unit-economics constraint: applicant volume is both
the reason AI adoption is attractive (Section 3.2) and the driver of its cost.

## 7.5 Synthesis

Read together, these four data points — rapid adoption, resume screening as a mainstream (not
autonomous) use case, a transparent multi-factor scoring norm, and a real per-applicant cost
constraint — describe an industry that is moving quickly toward AI-assisted screening while still
treating full automation cautiously and managing cost deliberately. This shapes the recommendations
in Section 9, which favor human-in-the-loop review, explainable scoring, and built-in cost governance
over fully autonomous, unmonitored AI decision-making.
