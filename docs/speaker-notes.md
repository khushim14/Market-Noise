# MarketNoise — Live Demo Script
> You're showing the running app, not slides. This is your full talking guide — what to say, how the models work, and how to explain everything confidently. Read it the night before, then speak naturally.

---

## GRADING CRITERIA — Keep these in mind the whole time
1. **Solution addresses the problem** → Keep connecting features back to "this is why retail investors get misled"
2. **MVP follows the customer journey** → Walk through the app in order: Home → Search → Stock Page → Analysis → Chat → Watchlist
3. **Smooth demo, no glitches** → Pre-warm the cache before you start (instructions below)
4. **Engaging presentation** → Start with a hook, not a feature list

---

## BEFORE YOU START — Do this 30 min early

```
# Terminal 1
redis-server

# Terminal 2
cd backend && source venv/bin/activate && python manage.py runserver

# Terminal 3
cd frontend && npm run dev
```

Then open `http://localhost:5173` and visit these pages — let each one **fully load**:
- `/stock/NVDA`
- `/stock/AAPL`
- `/stock/TSLA`

This warms Redis. Everything will be instant during your demo. Also log into your test account now so you don't fumble with credentials live.

---

## OPENING LINE — Say this before touching the app

> "I want to start with a quick scenario. You're on Reddit, you see NVDA is trending — up 18%, 200 articles in the last 24 hours. You want to know: is this real news driving the price, or is it just hype? You open Yahoo Finance — you see a chart. You open Google — you see a wall of articles. Nothing tells you whether this attention is information-driven or manufactured. That gap is exactly what MarketNoise fills."

---

## STEP 1 — Homepage

**What to click:** Show the homepage, don't navigate yet.

**What to say:**
> "This is the homepage. The search bar is the main entry point — any US stock ticker. Below is the live narrative feed — stocks that have been analyzed, ranked by narrative activity. There's also a daily challenge as part of our optional gamification system.

> Importantly — none of this requires a login. Any user gets full analysis without an account."

---

## STEP 2 — Search

**What to click:** Type "NVDA" in the search bar slowly.

**What to say:**
> "The autocomplete pulls from Twelve Data's symbol search API in real time — full company names, not just tickers. We cache these results too so repeat searches are instant."

Hit enter.

---

## STEP 3 — Stock Header + Chart

**What to click:** Show the header, then click 1D → 7D → 1M → 3M.

**What to say:**
> "Live price from Twelve Data's API. The change badge updates as I switch ranges — on 3M it shows the 3-month return, not just today's move. Small detail, but it gives real context.

> The WatchButton saves this to the user's watchlist — I'll demo that at the end."

---

## STEP 4 — SENTIMENT SCORE (explain the model fully here)

**What to click:** Scroll to the analysis cards, point at the Sentiment card.

**What to say:**

> "The Sentiment score is the heart of the product — let me explain exactly how it works.

> We use a model called **FinBERT**. To understand why it matters, you need to know a little about how NLP models work. Traditional sentiment tools like VADER — which is still used widely — are rule-based. They have a dictionary of words labeled positive or negative, and they count them. 'Great' is +0.7, 'terrible' is −0.9, and so on. The problem is that financial language is incredibly context-dependent. A phrase like *'beat estimates but cut forward guidance'* — VADER reads the word 'cut' and marks this negative. But any finance person knows that beating earnings estimates is very bullish. The word 'cut' here refers to guidance, which is forward-looking and nuanced. VADER completely misses this.

> **FinBERT** fixes this. It's built on BERT — which stands for Bidirectional Encoder Representations from Transformers — a transformer architecture from Google that reads the entire sentence in both directions at once, rather than left to right. This lets it understand context, not just individual words. Anthropic and OpenAI both use transformer architectures at their core. The key thing about BERT specifically is the bidirectional attention — it knows that 'guidance cut' in the context of 'beat estimates' means something different than 'guidance cut' alone.

> FinBERT is BERT **fine-tuned** on financial text. Fine-tuning means you take a pre-trained model — BERT already understands English grammar and context from training on Wikipedia and books — and you continue training it on a domain-specific dataset. For FinBERT, that dataset is over 100,000 financial documents: Reuters news articles, earnings call transcripts, analyst reports, and the Financial PhraseBank — a dataset of manually labeled financial sentences. After fine-tuning, FinBERT achieves about **87% accuracy** on the Financial PhraseBank benchmark. General BERT gets around 72%. VADER gets about 65%.

> For every news article we scrape, FinBERT outputs **three probability scores**: positive probability, negative probability, and neutral probability — all three sum to 1.0. We also compute a compound score from those, ranging from **−1 (most negative) to +1 (most positive)**. For example, an article might come back as: positive 0.82, negative 0.10, neutral 0.08 — compound +0.72.

> Now, we don't just average all the scores. We apply **exponential decay weighting**. The formula is: weight equals e to the negative lambda times age in days, where lambda equals log(2) divided by 3. What that means in plain English: an article from today has full weight. An article from 3 days ago has half weight. An article from 6 days ago has one-quarter weight. We chose a half-life of 3 days because market narratives are forward-looking — yesterday's news matters more than last week's news.

> The final sentiment is the weighted average of all article compound scores. If it's above +0.05 we call it Positive, below −0.05 is Negative, in between is Neutral."

---

## STEP 5 — NARRATIVE VELOCITY (explain the model)

**What to click:** Point at the Velocity card.

**What to say:**

> "Velocity measures how fast the volume of coverage is changing over time. Here's the math: we count how many articles were published about this stock in the last 24 hours — call that count_recent. We compare it to the 24 hours before that — call that count_prior. Velocity equals (count_recent minus count_prior) divided by count_prior, times 100.

> So if there were 10 articles yesterday and 20 today, velocity is 100 — coverage doubled. If articles dropped from 20 to 10, velocity is −50.

> We classify the trend as **Accelerating** if velocity is above +20%, **Decelerating** if below −20%, and **Stable** in between.

> Why does this matter? Spikes in narrative velocity are a leading signal — they often *precede* price reversals rather than follow them. When retail investors pile into a story, article volume surges. That surge peaks at maximum attention — which is historically close to the local price peak, right before the reversal. So a high velocity score, especially combined with a high hype score, is a warning sign that the narrative is overheated."

---

## STEP 6 — HYPE SCORE (explain the 3-factor model)

**What to click:** Point at the Hype Score card.

**What to say:**

> "The Hype Score is our most original contribution — a 3-factor model on a scale of 0 to 100.

> **Factor 1: Sentiment Imbalance — up to 40 points.** We look at what fraction of recent articles are positive versus negative. If 90% of coverage is bullish, that's a red flag — healthy stocks have balanced, mixed coverage reflecting genuine uncertainty. The formula is: take the dominant fraction, subtract 0.5, multiply by 2 to normalize it, and multiply by 40. So perfectly balanced coverage — 50/50 positive/negative — scores 0 on this factor. 80% one-sided scores 24. 95% one-sided scores 36.

> **Factor 2: Velocity Factor — up to 35 points.** Fast-moving attention is itself a hype signal, independent of what the articles say. The formula is: min(velocity divided by 100, 1.0) times 35. So a velocity score of 100 gives full 35 points. This captures the idea that even positive, accurate coverage can be hype if it's moving too fast for the information to be real.

> **Factor 3: Source Concentration — up to 25 points.** This is the one people don't think about. If 20 articles all came from 2 websites, that's not 20 independent data points — it's one story getting re-syndicated. The formula is: max(0, 1 minus unique_sources divided by total_articles) times 25. If every article came from a different source, concentration is 0. If all 20 articles came from 1 source, concentration is 25.

> We add all three factors. Below 20 is Low — healthy. 20–45 is Moderate. 45–70 is High. Above 70 is Extreme. GameStop in January 2021 would have scored in the Extreme range on all three factors simultaneously — extreme bullish imbalance, explosive velocity, and heavy concentration on Reddit and a handful of financial blogs."

---

## STEP 7 — PATTERN CLASSIFIER (explain the 6 archetypes)

**What to click:** Point at the Pattern card.

**What to say:**

> "On top of the hype score, we classify the narrative into one of 6 archetypes. This is a **rule-based classifier** — I want to be clear about that design choice. We deliberately chose rules over machine learning here for one reason: **explainability**. A user can see exactly which conditions triggered the label. There's no black box.

> The 6 patterns are:

> **Short Squeeze Dynamics** — this is the most extreme. It fires when: sentiment is strongly positive (above +0.4 compound), velocity is accelerating, sentiment imbalance is above 75%, and hype score is above 65. This is the GameStop/AMC setup — retail-driven pump with high reversal risk. We show this in red.

> **Macro Fear Pattern** — sustained negative sentiment (below −0.2), combined with broad multi-source coverage and elevated hype above 45. This is what you see around Fed rate decisions, sector regulation news, or systemic market fears. High risk label.

> **Theme/Sector Wave** — positive sentiment with accelerating velocity, diverse sources (not concentrated), and hype above 40. This is sector rotation — the AI boom, clean energy, crypto cycles. Positive but overheated.

> **Narrative Cool-off** — velocity is decelerating, hype was previously high but is dropping, coverage is thinning. The story is fading. Retail attention is moving on. Moderate risk.

> **Pre-Catalyst Build** — mentions are accelerating but the tone stays neutral or reserved, and volatility is low. This is anticipatory buildup — earnings, FDA decision, product launch. People are watching but not reacting yet.

> **Balanced Coverage** — none of the above triggers fire. Routine, diversified reporting. This is healthy. Low risk.

> The reason we chose rule-based over ML is that we don't have enough labeled pattern data to train a good classifier yet. We'd need thousands of manually labeled 'this was a short squeeze setup' examples. Rules let us deploy the feature immediately while being completely transparent about why a stock got a label."

---

## STEP 8 — AI NARRATIVE SUMMARY

**What to click:** Read the Narrative Summary card text.

**What to say:**

> "Below the cards is an AI-generated summary — we call Gemini or Groq with a prompt that includes the current sentiment score, velocity, hype level, pattern, and recent headline text. The prompt explicitly instructs the model to ground its answer in the data — no speculation, no price predictions, no trading advice. The result is a 3–4 sentence plain English paragraph that explains the narrative in terms a retail investor can actually act on. This is cached in Redis for 15 minutes because LLM calls are expensive."

---

## STEP 9 — NARRATIVE TIMELINE (explain drift detection)

**What to click:** Scroll to the Narrative Timeline chart.

**What to say:**

> "This is the 91-day Narrative Timeline — and it's the feature that most directly solves the 'no temporal context' problem.

> Here's how it works under the hood. Every day we aggregate all the sentiment scores for articles published about this stock on that day — weighted average of FinBERT compound scores. We store those daily aggregates. The chart plots them as an area chart over 91 days.

> But the more interesting piece is **drift detection and shift identification**. We compute rolling windows — 7-day, 30-day, and 90-day. For each window we calculate: the average sentiment, the direction of change (improving, worsening, or stable), and the magnitude of drift. If the 7-day average has shifted by more than 0.15 compound points from the prior 7-day average, we flag that as a **narrative shift** and annotate it on the chart.

> In practice this means you can see: this stock was bearish from early March, then on March 15th the narrative shifted bullish — probably around an earnings release or product announcement. That's information no other retail tool gives you.

> The window stats below the chart — 7d, 30d, 90d — tell you the average sentiment and drift direction for each period at a glance."

---

## STEP 10 — NEWS SECTION

**What to click:** Scroll to news articles briefly.

**What to say:**

> "News articles from GNews and our RSS scrapers — Yahoo Finance, MarketWatch, Google Finance News. Each article shows its FinBERT sentiment label and compound score. The scraping runs automatically every 30 minutes via Celery, our background task scheduler. New articles are immediately scored by FinBERT and stored — so by the time a user searches a stock, we already have fresh scored data ready."

---

## STEP 11 — AI CHAT PANEL

**What to click:** Type into the chat panel.

**Suggested message:** `"What is driving the current sentiment for NVDA and should I be concerned about the hype level?"`

**What to say:**

> "The chat panel on the right is sticky — always visible as you scroll. When a user sends a message, we build a context object that includes: the current sentiment score, velocity trend, hype score and level, the pattern archetype, and the last 5 news headlines. That context goes into the system prompt before the user's question.

> So when I ask 'what is driving the current sentiment' — the model isn't just drawing on its training data about NVDA in general. It's answering based on the specific data we computed today. That's what makes it useful rather than generic.

> This requires login and the user's own Gemini or Groq API key. We store those keys encrypted using **Fernet symmetric encryption** from the Python cryptography library. Fernet generates a secret key — stored in environment variables — and uses AES-128-CBC to encrypt the user's API key. It's stored as ciphertext in the database. When the user sends a chat message, we decrypt it in memory, make the LLM call, and immediately discard the plaintext. It's never logged, never returned through any API response."

---

## STEP 12 — WATCHLIST

**What to click:** Click the WatchButton. Then navigate to `/watchlist`.

**What to say:**

> "Saving to watchlist — watch the animation."

*(Click — show green glow + toast)*

> "Now the watchlist page — all saved tickers with their current signals. Sentiment, velocity, hype score at a glance. This is a JWT-authenticated endpoint — the backend returns only the watchlist entries belonging to the logged-in user."

---

## STEP 13 — GAMIFICATION (show quickly)

**What to say:**

> "We built an optional gamification layer — completely toggle-able. XP for every meaningful action: +10 for searching, +15 for visiting a stock page, +20 for sending a chat message, +50 for the daily challenge. Missions, badges, confetti on milestones.

> The design decision here was to keep it entirely in localStorage with useReducer and Context API — no backend tables, no migrations, no endpoints. For an MVP feature, that tradeoff was worth it. The experience is identical for the user, it persists across page refreshes, and it doesn't require an account. If we add a premium tier later, syncing it to the backend is straightforward."

---

## CLOSING LINE

> "So that's MarketNoise. The core thesis: retail investors deserve to understand the narrative context around a stock — not just the price chart. We built a full-stack platform using React, Django, FinBERT, and Celery that quantifies narrative sentiment, velocity, and hype with transparent, explainable scores. Not a black-box prediction. Not a buy or sell signal. Just honest intelligence about what the story is and whether it's real. Happy to take questions."

---

## THE FULL TECHNICAL PICTURE — Read this so you really understand it

### How the backend pipeline works end to end

Every 30 minutes, a **Celery Beat** scheduler fires a task for every active stock ticker we're tracking. Celery is a distributed task queue — think of it like a job scheduler that runs in the background, completely separate from the web server. These tasks run in parallel — one worker per ticker.

Each task does four things:
1. **Scrapes** articles from RSS feeds (Yahoo Finance, MarketWatch, Google Finance — unlimited, no quota), GNews API (100 requests per day free tier), and Reddit via PRAW (the Python Reddit API wrapper — pulls from r/wallstreetbets, r/stocks, r/investing). Articles are deduplicated by URL before storing — a database unique constraint ensures we never score the same article twice.
2. **Scores** each new article through FinBERT in batches of 32. Batching is important — the GPU (or CPU) processes 32 articles simultaneously using matrix operations, which is about 10x faster than running them one by one.
3. **Stores** the scores in the SentimentScore table, linked to each Narrative record.
4. **Creates PriceImpact records** — logs the stock price at the time of publication, then schedules follow-up tasks at 1 hour, 4 hours, and 24 hours to record what the price did. This is our ML training data.

When a user visits a stock page, the Django view checks Redis first. If cached (TTL 15 minutes), it returns immediately — under 100ms. If not cached, it runs the full analysis pipeline: queries the DB for the last 20 articles, runs FinBERT on any unscored ones, computes the weighted average, calculates velocity, computes the hype score, classifies the pattern, and caches everything. First call is 2–5 seconds. Every call after that is instant.

### How FinBERT is loaded

FinBERT is a **singleton** — it loads once when the Django process starts and stays in memory. It's 440MB. Loading it every request would take 3–4 seconds each time and would be completely impractical. The singleton pattern means: first request loads the model, every subsequent request reuses the same in-memory model instance. We also auto-detect hardware: if you have an Apple Silicon Mac, we route to MPS (Metal Performance Shaders). If you have a CUDA GPU, it uses CUDA. Otherwise it falls back to CPU. The inference speed difference between GPU and CPU for batch size 32 is roughly 5x.

### How the hype score aggregation works

The hype score is computed fresh every time we run analysis (and then cached). We pull all articles from the last 7 days for the stock. We split them into positive, negative, and neutral buckets based on FinBERT labels. The dominant fraction is whichever bucket is largest divided by total articles. We compute velocity by querying article counts for the last 24h vs prior 24h. We compute source concentration by counting unique domains (we extract the domain from each article URL) versus total article count. Three factors get summed into the final score.

### How the pattern classifier works

The classifier checks conditions in priority order — most extreme patterns checked first, fallback to Balanced Coverage if nothing else matches. It's essentially a big if-elif-else chain, but each condition is a combination of: the sentiment compound score threshold, the velocity trend enum (ACCELERATING / STABLE / DECELERATING), the hype level enum (LOW / MODERATE / HIGH / EXTREME), the sentiment imbalance ratio, and article count. The explainability comes from the fact that each branch has a fixed set of human-readable conditions attached — we return not just the pattern name but the exact conditions that triggered it.

### How JWT authentication works

When a user logs in, Django generates two tokens: a **short-lived access token** (default: 5 minutes) and a **long-lived refresh token** (default: 1 day). The frontend stores both in localStorage. The Axios interceptor on the frontend checks if the access token is expired before every request. If it is, it automatically calls the refresh endpoint to get a new access token — silently, without the user noticing. This means users are never logged out unexpectedly. If the refresh token is also expired, they get redirected to login.

### The ML classifier (experimental)

We train a Random Forest and XGBoost model using 12 features extracted from each article: the four FinBERT probability scores (positive, negative, neutral, compound), whether the source is news or Reddit, the hour of day and day of week (market hours matter a lot for price impact), and the current hype level and article count at time of publication. The label is derived from the PriceImpact records: if price moved more than +0.15% in 24 hours, label is 'up'. More than −0.15% is 'down'. Otherwise 'flat'. The model only activates when we have 50 or more labeled samples — before that we don't show it to users. We use scikit-learn's RandomForestClassifier and xgboost.XGBClassifier, trained with stratified cross-validation.

---

## TOUGH QUESTIONS — Be ready

**"How is this different from Yahoo Finance or Bloomberg?"**
> Yahoo Finance shows you charts and articles. Bloomberg shows you prices and terminal data. Neither scores sentiment with a financial NLP model, measures narrative velocity, calculates a 3-factor hype score, classifies narrative patterns, or shows 91-day temporal drift with shift detection. We're a narrative intelligence layer that genuinely doesn't exist in those products.

**"Why not just use ChatGPT for everything?"**
> ChatGPT gives you prose. We need structured, quantified outputs — compound scores from −1 to +1, velocity percentages, hype ratings — that we can weight, aggregate, trend, and visualize. You can't chart a ChatGPT response. You can't compare two ChatGPT scores over 91 days. LLMs are only used where prose is the right output: the summary card and the chat panel.

**"Isn't predicting markets impossible?"**
> Completely agreed — which is why price prediction is explicitly not our product. The ML classifier is labeled experimental, shown with a confidence score, and the codebase has a comment explaining we don't surface it as a recommendation. Our product is narrative intelligence: explaining why attention is moving, not whether to buy.

**"How accurate is FinBERT?"**
> On the Financial PhraseBank benchmark — 87% accuracy vs VADER's ~65% and general BERT's ~72%. More importantly, FinBERT handles the specific patterns that matter in practice: guidance language, earnings beats, analyst upgrades, sector risk language. That's why we chose it over building our own model.

**"What's your biggest technical challenge?"**
> Running FinBERT at scale without per-request latency. 440MB model, 2–5 second first inference. Solved with: singleton loader, batch inference, and aggressive Redis caching. The architecture means most production requests never touch the model at all.

**"Why rule-based patterns instead of ML?"**
> Two reasons. First, we don't have enough labeled data — we'd need thousands of manually tagged examples of 'this was a short squeeze setup' to train a classifier. Rules let us ship immediately. Second, explainability. A retail investor who sees 'Short Squeeze Dynamics' can click and see the exact conditions that triggered it. A 73% probability from a random forest tells them nothing about why.

**"Why localStorage for gamification?"**
> Deliberate MVP tradeoff. No migration, no endpoint, no login required. Survives refreshes. When we productionize, migrating it to the backend is a few hours of work. For a feature that's entirely optional, the simplicity was worth it.

**"What would you build next?"**
> First: pattern-change email alerts — notify users when a watched stock's archetype shifts from Balanced Coverage to Short Squeeze Dynamics. Second: statistical backtest of whether hype above 70 actually predicts reversals. That's the academic validation that turns this from interesting to trustworthy. Third: WebSockets to replace polling and make the feed truly real-time.
