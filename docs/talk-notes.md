# MarketNoise — Talk Notes
> Phone-friendly. Short. Scroll as you talk.

---

## BEFORE YOU START
- Redis running ✓
- Django running ✓
- Frontend running ✓
- Visit /stock/NVDA, /stock/AAPL once to warm cache
- Log in to your account

---

## OPENING — say this first

> "You see NVDA trending on Reddit — up 18%, 200 articles in 24 hours. Is this real news or just hype? Yahoo Finance shows a chart. Google shows a wall of articles. Nothing tells you if the attention is real or manufactured. That's what MarketNoise solves."

---

## THE PROBLEM

- Too much noise, no signal filter
- Retail investors buy the hype peak, sell the crash
- General NLP (VADER) misreads finance — "beat estimates but cut guidance" = actually bullish, VADER calls it negative
- No tool shows how the narrative evolved over time

---

## OUR SOLUTION

- **Hype Score** → tells you if attention is manufactured
- **Narrative Velocity** → detects spikes before they peak
- **FinBERT** → financial NLP, 87% accuracy vs VADER's 65%
- **91-day Timeline** → shows exactly when the story shifted

---

## HOMEPAGE

- Search bar — main entry point
- Live narrative feed — stocks ranked by activity
- Daily challenge — optional gamification
- **No login needed for any of this**

---

## SEARCH

- Type NVDA, show autocomplete
- Pulls from Twelve Data API in real time
- Cached so repeat searches are instant

---

## STOCK PAGE — PRICE CHART

- Live price from Twelve Data
- Click through ranges: 1D → 7D → 1M → 3M
- Badge updates to show return for that range, not just today

---

## SENTIMENT SCORE (spend time here)

- Powered by **FinBERT** — BERT fine-tuned on 100k+ financial docs
- VADER counts words. FinBERT reads full sentence context.
- Output per article: positive / negative / neutral probabilities + compound −1 to +1
- We weight recent articles more — half-life of 3 days
- 87% accuracy on finance benchmark vs VADER's 65%

---

## NARRATIVE VELOCITY

- Measures how fast coverage is changing, not just volume
- Formula: (articles today − articles yesterday) / yesterday × 100
- Spikes in velocity often come **before** price reversals
- Labels: Accelerating / Stable / Decelerating

---

## HYPE SCORE (3 factors)

**Factor 1 — Sentiment Imbalance** (up to 40 pts)
→ 90% bullish = suspicious. Healthy = balanced.

**Factor 2 — Velocity** (up to 35 pts)
→ Fast-moving attention is itself a hype signal.

**Factor 3 — Source Concentration** (up to 25 pts)
→ 20 articles from 2 sites ≠ 20 independent opinions.

Levels: Low (<20) · Moderate (20–45) · High (45–70) · Extreme (>70)
GameStop Jan 2021 = Extreme on all 3.

---

## PATTERN CLASSIFIER (6 types)

- 🔥 **Short Squeeze** — extreme bullish + accelerating + imbalanced. GME setup.
- 😨 **Macro Fear** — sustained negative + broad coverage. Fed, regulation.
- 🌊 **Sector Wave** — positive + accelerating + diverse. AI boom, crypto cycles.
- 📉 **Narrative Cool-off** — velocity dropping, story fading.
- ⏳ **Pre-Catalyst** — mentions up but tone neutral. Earnings coming.
- ✅ **Balanced Coverage** — healthy, nothing unusual.

Rule-based not ML → users see exactly why a label fired. No black box.

---

## AI NARRATIVE SUMMARY

- Gemini reads current sentiment + velocity + hype + headlines
- Writes 3–4 sentence plain English paragraph
- No speculation, no price prediction
- Cached 15 min because LLM calls cost money

---

## NARRATIVE TIMELINE

- 91-day rolling chart of how sentiment evolved
- Shift detection: if sentiment moves >0.15 points, annotates the date on chart
- Window stats: 7d / 30d / 90d at a glance
- "Most tools show today. We show the whole story."

---

## AI CHAT

- Sticky sidebar, always visible
- Context injected: sentiment, velocity, hype, pattern, last 5 headlines
- Needs login + user's own Gemini/Groq key
- Key stored encrypted (Fernet AES encryption), never logged

---

## WATCHLIST

- Click WatchButton → green glow + toast
- /watchlist shows all saved tickers with live signals
- JWT auth — only returns current user's entries

---

## GAMIFICATION (quick mention)

- XP for every action: search +10, visit +15, chat +20, daily challenge +50
- Missions + badges + confetti
- All in localStorage, no backend needed
- Fully toggle-able

---

## CLOSING

> "MarketNoise gives retail investors what they've never had — a clear, transparent answer to whether market attention is real or manufactured. React, Django, FinBERT, Celery. No black-box predictions, no fake signals. Just honest narrative intelligence."

---

---

# AI CHAT DEMO QUESTIONS
> Type these in the chat panel during the demo. Each one shows a different strength.

---

### Question 1 — Basic narrative question
```
What is driving the current sentiment for NVDA?
```
**Why ask this:** Shows the AI uses real data (sentiment score, headlines) not generic knowledge.

---

### Question 2 — Hype check
```
Is the current attention around NVDA information-driven or hype? Should I be concerned?
```
**Why ask this:** Directly connects to our core value prop. Answer will reference hype score + factors.

---

### Question 3 — Pattern explanation
```
What does the current narrative pattern mean for NVDA and what typically happens next?
```
**Why ask this:** Shows the pattern archetype in action. AI explains the archetype in plain English.

---

### Question 4 — Timeline / history
```
How has the narrative around NVDA changed over the past month?
```
**Why ask this:** Shows temporal intelligence — references drift direction and shift detection.

---

### Question 5 — Comparison
```
Compare the sentiment and hype level for NVDA vs what you'd expect from a healthy stock.
```
**Why ask this:** Shows the AI can reason about what "normal" looks like vs current state.

---

### Question 6 — Risk question
```
What are the biggest narrative risks for NVDA right now based on the current data?
```
**Why ask this:** Shows the AI is grounded in real signals, not making things up.

---

### Question 7 — Simple and punchy (good for demo)
```
In one sentence, is NVDA narrative healthy or risky right now?
```
**Why ask this:** Quick, clean response. Good for showing off during a time-limited demo.

---

## QUICK Q&A ANSWERS

**Different from Yahoo Finance?**
→ They show charts and articles. We score sentiment, measure velocity, calculate hype, classify patterns, show 91-day drift. That layer doesn't exist there.

**Why not just use ChatGPT?**
→ ChatGPT gives prose. We need numbers — compound scores, velocity %, hype 0–100 — to chart and trend over time. LLMs only used where prose is the right output.

**Isn't predicting markets impossible?**
→ Agreed — we don't predict prices. We explain narratives. The ML classifier is labeled experimental.

**FinBERT accuracy?**
→ 87% on Financial PhraseBank vs VADER's 65%. Handles earnings language, guidance cuts, sector risk.

**Biggest technical challenge?**
→ Running 440MB FinBERT without lag. Fixed with singleton loader + batch inference + Redis caching.
