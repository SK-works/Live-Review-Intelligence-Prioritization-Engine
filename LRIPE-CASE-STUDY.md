# Live Review Intelligence Pipeline
### A real-time automation for ranking customer feedback by business impact, not just volume

---

## The problem

Most eCommerce teams treat customer reviews as something to skim, not something to act on systematically. Reviews pile up unread, or get summarized in a monthly report that's already stale by the time anyone reads it. Meanwhile, a defect that's driving a small but sharp spike in negative sentiment can sit invisible next to a louder but harmless theme like "wish it came in more colors."

I built this as a self-contained, fully free automation to answer one question the moment new feedback arrives: **which issue actually deserves attention first, and what should we do about it?**

## What I built

A live pipeline that:
- Classifies every incoming review by sentiment and theme in real time
- Ranks themes by a weighted impact score (40% how often it comes up, 60% how negative it skews) — not just raw mention count
- Auto-drafts a specific corrective action for the top 3 highest-impact themes
- Runs every recommendation through an automated QA gate before it reaches a dashboard, rejecting vague or incomplete output
- Updates a live, shareable dashboard that product and category teams can open at any time — no access to the automation tooling required

Everything runs on free-tier infrastructure: n8n (self-hosted, open source), Groq's free-tier Llama 3.3 70B for the LLM calls, and Google Sheets as both the data log and the dashboard surface.

## Why the QA gate matters

It would have been easy to just pipe an LLM's output straight to a dashboard. I didn't, because the failure mode of unchecked LLM output — vague, generic advice that sounds actionable but says nothing ("improve quality," "monitor the situation") — is worse than no automation at all: it creates false confidence.

Instead, every recommendation is mechanically checked:
- Does it cover every theme it was asked to?
- Is it long enough to be specific, not just a one-line platitude?
- Does it avoid a list of known vague filler phrases?

Anything that fails gets routed to a "needs review" state on the dashboard instead of being silently presented as a finished recommendation. This is the same principle used across every automation I've built recently: **the system should be honest about what it doesn't know, not just fluent.**

## Why impact-weighted ranking, not just frequency

A theme that appears in 3 reviews, all 1-star, is often a bigger problem than a theme that appears in 10 reviews, mostly 4- and 5-star. Simple "most mentioned" dashboards miss this. The ranking formula:

```
priority score = 0.4 × (share of total review volume)
              + 0.6 × (share of all negative sentiment)
```

weights negative-sentiment concentration more heavily than raw volume, so a small but sharply negative theme can and does outrank a larger, mostly-neutral one. This mirrors the ROI-based prioritization approach used for scoring AI/automation initiatives — applied here to customer feedback triage instead of a project portfolio.

## Architecture

*(see architecture-diagram.svg / README.md for the full visual pipeline)*

In short: **Website → Webhook → Classify → Log → Re-rank entire history → Recommend → QA gate → Live Dashboard**, round-tripping in under 30 seconds per review.

## What I'd build next

- Replace the current keyword-based QA rubric with an LLM-as-judge evaluator for stricter, less brittle checks — and document the tradeoff between the two approaches, since that comparison is itself a useful demonstration of eval design maturity
- Add a scheduled daily/weekly executive digest alongside the real-time dashboard, for stakeholders who want a periodic summary rather than a live feed
- Connect it to a real review source (Shopify, a marketplace API, or a CSV export pipeline) instead of the demo storefront used here

## Try it

Full workflow file, demo website, and setup instructions are in the accompanying GitHub repo.

