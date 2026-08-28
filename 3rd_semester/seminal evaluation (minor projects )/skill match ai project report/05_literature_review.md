<div style="page-break-before: always;"></div>

# 5. Literature Review / Background Study

This section reviews secondary sources — industry survey data, technical documentation, and
established concepts in HR-technology and software engineering — relevant to AI-driven recruitment
platforms.

## 5.1 From Applicant Tracking Systems to AI Matching

Traditional Applicant Tracking Systems (ATS) filter resumes primarily through keyword matching:
resumes are scanned for exact or near-exact matches against terms in a job description. Industry
commentary on ATS technology widely notes two weaknesses of this approach — first, it penalizes
qualified candidates who describe their experience in different words than the job posting; second,
it provides no ranking logic beyond binary keyword presence. This gap is the primary business
opportunity that AI-based matching addresses: large language models can read a resume the way a
human recruiter would, inferring skills and seniority even when the wording differs from the job
description.

Industry adoption confirms this shift is well underway. SHRM's 2025 Talent Trends research found that
69% of HR professionals now use AI in recruiting — up from 51% a year earlier — with resume screening
the second most common use case at 44% (Society for Human Resource Management, 2025a). Gartner
similarly identifies the "AI revolution" as one of the two forces reshaping talent-acquisition
strategy through 2026, noting that AI recruitment features have matured from isolated pilots into a
standard set of use cases spanning the full hiring lifecycle (Gartner, 2025). LinkedIn's own
platform-scale data reinforces this: recruiters using AI-assisted messaging see a measurable
9-percentage-point increase in quality-of-hire likelihood, and 60% of surveyed talent professionals
believe AI will improve recruiting outcomes overall (LinkedIn, 2025) — evidence that the trend is
being validated by outcome data, not adoption figures alone.

## 5.2 Large Language Models in Document Understanding

The release of transformer-based LLMs (e.g., OpenAI's GPT series) made it practical for software
products to extract structured information — skills, years of experience, education, role
seniority — from unstructured text such as a résumé, at a cost per document low enough for
commercial products to use routinely. Smaller, cost-efficient model variants (e.g., GPT-4o-mini) are
generally preferred for this kind of high-volume, low-latency extraction task over larger,
open-ended-generation models.

## 5.3 Weighted Multi-Factor Scoring

Literature on candidate-ranking systems generally favors multi-factor scoring over a single
similarity number, because it lets recruiters see *why* a candidate ranked where they did. A typical
approach combines **required-skills overlap**, **experience level**, **semantic similarity** between
resume and job description, and **nice-to-have skills**, weighted so that hard requirements count more
than soft/preferred criteria while overall contextual fit is still rewarded. This mirrors common
HR-tech practice of prioritizing precision (matching what a job explicitly requires) over recall,
since false positives in a recruiter-facing shortlist waste more reviewer time than false negatives.

## 5.4 Cost and Reliability Considerations in AI Products

A recurring theme in secondary sources on production AI systems is **cost governance** — AI API calls
have a real, variable, per-request cost that can spike unpredictably under load. Two patterns recur in
the literature: (a) **usage logging**, where every AI call's cost is recorded so spend can be
monitored and capped; and (b) **graceful degradation**, where a system falls back to a cheaper,
deterministic method — such as TF-IDF keyword matching — when an AI budget threshold is exceeded,
rather than failing outright or incurring unbounded cost.

## 5.5 Security and Access Control in Multi-Role Platforms

Because a recruitment platform holds personally identifiable information (resumes, contact details,
employment history) and separates capabilities by user type, secondary sources on web application
security consistently recommend: JWT-based stateless authentication with short-lived access tokens
and refresh tokens; role-based access control (RBAC) so job seekers, recruiters, and admins can only
reach functionality appropriate to their role; and defensive measures (rate limiting, input
validation, security headers) to reduce the attack surface of a publicly reachable API.

## 5.6 Bias Risk in AI Screening

The literature also documents a well-known failure mode that any organization adopting AI screening
must account for: AI tools can encode and amplify bias present in their training data. Amazon's
internal recruiting tool famously penalized resumes containing the word "women's" after training on
a decade of male-skewed hiring data (Sanford Heisler Sharp, 2024), and SHRM's own 2025 benchmarking
data shows 19% of organizations using AI in hiring report their tools overlooked or screened out
qualified applicants (Society for Human Resource Management, 2025b). This motivates the human-oversight
and bias-review recommendations discussed in Section 9.

## 5.7 Peer-Reviewed Evidence of Algorithmic Bias

Beyond individual trade cases, peer-reviewed research quantifies this risk directly. An, Huang, Lin,
and Tai (2025), in a controlled study of roughly 361,000 fictitious resumes scored by five leading
LLMs (including GPT-4o and Claude 3.5 Sonnet), found systematic intersectional bias by gender and
race — lower scores for otherwise-identical Black male candidates, translating to an estimated
1–3 percentage-point difference in modeled hiring probability. An earlier systematic review of 36
peer-reviewed studies on algorithmic hiring similarly concluded that discrimination risk in
algorithmic HR decision-making was, at the time, "mostly unexplored" and called for stronger fairness
auditing (Köchling & Wehner, 2020). Read together with the trade-press evidence in Section 5.6, this
shows algorithmic bias in hiring is an empirically measured effect, not a speculative concern.

## 5.8 Regulatory Response

Bias risk has already produced binding regulation, not just guidance. New York City's Local Law 144
requires any employer using an "Automated Employment Decision Tool" to commission an independent
annual bias audit and publish its results before the tool can legally be used in hiring, with civil
penalties of $500–$1,500 per violation for non-compliance (New York City Department of Consumer and
Worker Protection, 2023). This signals that the bias-review recommendation developed later in this
report (Section 9) is not merely best practice but, in some jurisdictions, an emerging legal
requirement for platforms of exactly this kind.

## 5.9 Synthesis

Taken together, the literature frames AI-driven recruitment as an application of three converging
trends — cheap, accurate LLM document understanding; established multi-factor scoring practice from
ATS/HR-tech research; and standard security/cost-governance patterns — balanced against a
well-documented, empirically measured, and increasingly regulated risk of algorithmic bias that
adopters must actively manage.
