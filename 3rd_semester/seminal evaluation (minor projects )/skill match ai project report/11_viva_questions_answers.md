<div style="page-break-before: always;"></div>

# 11. Viva Answers (5 Descriptive Questions, 6 Marks Each)

*Per the guideline, viva questions are project-specific and only unlock after the report is uploaded.
The 5 questions below are anticipated based on this report's content and should be reviewed/adjusted
once the actual portal questions are revealed.*

## Q1. What business problem does your project address, and why is it significant?

Manual resume screening does not scale with the volume of applications online job postings now
receive; recruiters filtering resumes by hand, or with simple keyword-matching ATS tools, either miss
qualified candidates whose resumes are worded differently from the job description, or spend
disproportionate time on low-value manual review. This project studies how AI is used to address this
by reading and understanding resume content the way a human recruiter would, then producing a ranked
shortlist automatically. This matters because it is representative of a real, ongoing shift in the
HR-technology industry: current survey data shows 69% of HR professionals already use AI in
recruiting, up from 51% a year earlier, and understanding how to adopt it responsibly — with
transparent scoring, cost controls, and bias safeguards — has real professional value.

## Q2. Explain the typical system architecture of an AI-powered recruitment platform.

Such platforms generally follow a three-layer architecture. The **client layer** is a web application
providing separate views for job seekers and recruiters. The **API layer** handles authentication,
profile management, job postings, and applications behind a security middleware chain (access
control, rate limiting, input validation). The **data and AI services layer** stores structured
candidate/job data and integrates a large-language-model API to parse resumes and compute match
scores. SkillMatch AI, used as the illustrative example throughout this report, follows this pattern —
a React-based frontend, an Express/MongoDB backend, and an OpenAI-based AI engine — which lets each
layer be understood, and in principle replaced, independently of the others.

## Q3. How does AI-based resume matching typically work, and what methodology does it follow?

When a candidate uploads a resume, the text is extracted and sent to a language model, which returns
structured output — extracted skills, a summary, and experience signals. A match score is then
computed against each job posting using a weighted, multi-factor model: required-skills overlap
(around 50%), experience level (around 25%), semantic similarity between resume and job description
(around 15%), and nice-to-have skills (around 10%). This weighting favors precision over recall,
appropriate for a recruiter-facing shortlist tool. Because AI API calls carry a real, variable cost,
well-designed systems also log AI usage and fall back to a cheaper, deterministic method (such as
keyword matching) if a cost budget is exceeded, rather than failing outright.

## Q4. What security and access-control considerations apply to platforms like this, and why?

Because a recruitment platform stores personal data — résumés, contact details, and employment
history — across multiple user roles (job seeker, recruiter, admin), the literature is consistent
that certain protections are baseline requirements, not optional extras: JWT-based authentication with
refresh tokens, role-based access control so each role can only reach its own functionality, and
defensive middleware such as rate limiting and input validation to reduce the attack surface of a
public API. My analysis concludes that these should be designed in from the start of any such system,
since retrofitting security after a feature set is already built is a common and risky pattern in
early-stage software.

## Q5. What are your key recommendations, and what would you do differently or next?

My primary recommendation is that AI screening should be treated as decision support rather than
autonomous decision-making — industry data shows organizations already trust AI more for content
generation than for filtering candidates outright, and 19% of AI-adopting organizations report their
tools have overlooked qualified applicants, which argues for keeping a human in the loop. Second, I
recommend keeping the AI match score's components visible to recruiters individually, not just a
combined percentage, to preserve interpretability and trust. Third, I recommend building security and
AI-cost governance in from the outset rather than as later additions. Finally, since AI models can
reflect biases present in training data or in how job descriptions are phrased — as shown both by
documented cases such as Amazon's discontinued recruiting tool and by peer-reviewed research finding
a measurable 1–3 percentage-point bias swing across major LLMs (An et al., 2025) — I recommend a
periodic, formal bias-review process for any AI screening system's output before relying on its
rankings at scale. This is no longer purely aspirational: New York City's Local Law 144 already
requires an independent annual bias audit for automated hiring tools, which is a useful template for
what such a process should look like even outside jurisdictions where it is legally mandated.
