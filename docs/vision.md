# Vision & Behavioral Foundation

## Executive Summary

Most people do not have a lack-of-financial-information problem. They have an **emotional aversion to money**. Spending-control apps do not solve this — they worsen it, because money hurts exactly when the user opens them.

Harpy Aerie inverts the equation: the system goes to the user, not the other way around. It captures any transaction (receipt photo, Telegram message, bank CSV, purchase confirmation email), classifies it, enriches it with real context, and delivers analysis in natural language. No forms. No dashboards that must be opened. No financial education.

---

## What it does

Five integrated modules driven by a **Signal Router** (LLM) that classifies the intention of every input and routes it to the correct module:

- **PFM** — captures, classifies and enriches transactions from any channel. The heart of the system.
- **Market Analysis** — macro context via BCB, IBGE and FEBRABAN.
- **Investments** — B3 and Tesouro Direto tracking.
- **Conversational Interface** — everything accessible via Telegram or CLI, in natural language.
- **OPS** — personal organization integrated with Todoist and an automated Daily Planner.

---

## Why it works where others fail

| Competitor | Gap |
|------------|-----|
| Cleo, Nubank, Inter | Channel with emotional load — money hurts exactly there |
| Monarch, Copilot | Passive dashboard — the user has to open it |
| Olivia AI | Closest approach. Acquired by Bradesco and died |

**No system combines:** behavioral choice architecture + modern LLM for natural language + Telegram as zero-friction channel.

---

## How it is built

**Stack:** Python · MongoDB · SQS FIFO · Lambda · Ollama (dev) / Bedrock (prod) · Telegram Bot API · Pluggy · Metabase

**Operating principle:** MVP runs 100% local — financial data never leaves the machine. Production migration swaps two environment variables. Zero rewrite.

---

## Project Vision

Democratize access to high-end financial planning — the kind of analysis that only those with expensive investment advisors get today. Harpy Aerie is a **democratic financial planner**: it aggregates real data from the user's financial life, removes the emotional friction of money, and acts based on applied behavioral psychology, not financial education.

> **Central principle:** It is not a spending-control app. It is an intelligent personal financial planner built for people who feel emotional aversion to money — not for those who already have financial discipline.

> **Why democratic?** Quality financial advisory is expensive and assumes the user is already comfortable with their own money. Harpy Aerie solves both problems at once: access and emotional comfort.

---

## The Problem It Solves

Today, to get a serious financial analysis you need to:

- Pay an advisor (expensive)
- Use apps that do not talk to each other
- Manually interpret bank reports
- Cross data from multiple sources in your head

Harpy Aerie does all of this automatically — and without generating anxiety in the process.

---

## Behavioral Foundation — Financial Psychology

> **Principle:** Harpy Aerie is not a spending-control app with AI. It is a choice-architecture system that removes the emotional friction of money — not the technical one.

### Why behavioral finance?

Most people do not stop looking at their finances because they lack an app. They stop because **money hurts to look at**. COM-B explains it: Capability and Opportunity exist — the gap is in Motivation. Emotional aversion, financial anxiety, shame.

Harpy Aerie attacks exactly that gap.

### Academic reference

**Bernardo Fonseca Nunes** (UFRGS, Nova SBE, Stirling) + **Daniel Kahneman** as theoretical base.

Nunes prescribes: **nudges + commitment > pure financial education.**

> *"Financial education takes a long time and sometimes a short-term solution is more efficient."*

Harpy Aerie does not educate. It acts.

### Applied frameworks

**EAST (BIT)**
- **Easy** → Telegram. Zero friction. Already open.
- **Attractive** → language without weight, without judgment
- **Timely** → notifies at the right moment, not when the system thinks it should

**MINDSPACE**
- **Affect** → positive emotional tone, never charging
- **Defaults** → automatic categorization — user decides nothing by default
- **Salience** → highlights what matters, not everything

**COM-B**
- The gap is not Capability or Opportunity — it is **Motivation**
- A well-designed environment replaces willpower

### Kahneman in practice

System 1 (automatic, emotional) is what makes the person close the bank app.

Harpy Aerie **never activates System 1 negatively:**
- No large numbers in highlight
- No painful comparisons thrown without context
- No "you spent X% more this month" without invitation

When something difficult must be shown, it is framed differently:

> *"You made 47 financial decisions this month. Most were small. Want to see the overview?"*

It is a choice. The user decides whether to look or not.

### Sophisticated vs. Naïve

Nunes distinguishes two profiles. Harpy Aerie identifies which the user is silently, by behavior:

- **Sophisticated** — knows they will fail and wants protection from themselves → offers commitment mechanisms defined by the user
- **Naïve** — thinks they will resist, fails → uses strong defaults without asking

### Commitment mechanisms

- **Conversational contract** — user defines rules in natural language, system enforces
- **Calendarized rebalancing** — fixed dates, not emotional triggers
- **Explicit mental accounting** — recurring vs. extra income, fixed vs. variable, reserve vs. investment never mix
- **Non-discretionary rules** — emotional stop-loss, fixed allocation, pattern alerts

### What Harpy Aerie never does

- Never uses the word "debt" without context
- Never compares against unmet goals
- Never sends unsolicited reports
- Never demands a reply
- Never educates — only acts

### Telegram as choice architecture

It is not a feature — it is the environment. Money reaches the user, not the other way around. Familiar, neutral, without the emotional weight of "I am going to open the bank".

### Measurable ROI

Meier & Sprenger (2010): present bias → +15-20% chance of extra accumulated debt.

Harpy Aerie is not only emotional comfort. It has real and measurable financial impact.

---

## Market gap

| System | What it does | Gap |
|--------|--------------|-----|
| Cleo (UK/US) | Financial chatbot with persona | No real behavioral rigor, no modern LLM |
| Monarch / Copilot | Premium PFM, beautiful | Passive dashboard — you have to open it |
| Nubank / Inter | Superficial gamification | They are banks — money hurts exactly there |
| Olivia AI | Closest approach | Acquired by Bradesco, died |

**No system combines:** Kahneman/Nunes rigor + commitment in natural language via LLM + zero-friction channel.
