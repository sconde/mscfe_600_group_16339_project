# Part 2 - User Guide: Social-Media Sentiment Data

*This guide explains how we would source, structure, assess, and explore social-media sentiment
as an alternative dataset for finance. We write it as a practical reference: what the data are,
where they come from, what can go wrong, and how we would begin an analysis responsibly.*

## 1. Sources of Data

Social-media sentiment is text-based behavioral data. It comes from public posts, comments, tags,
and reactions on platforms where investors and consumers talk about markets, companies, and
economic events. Sun et al. put it in the behavioral alternative-data category because it often
moves faster than formal information channels do - stocks can start reacting to social chatter
well before any analyst report or press release appears.

The most useful finance-specific source is StockTwits, where users tag securities with cashtags
and frequently label their own posts as bullish or bearish. Those self-reported labels are worth
having because they provide a ready-made sentiment target without any additional modeling. X
(formerly Twitter) is broader and faster, reaching a much larger population, but access has
become more restricted and expensive, and the finance conversation is heavily mixed with unrelated
content that needs to be filtered out. Reddit provides longer and more argumentative discussions
in communities like r/investing, r/stocks, and r/wallstreetbets, which are useful for tracking
retail narratives and crowding behavior. YouTube comments and transcripts can add context around
earnings calls, central-bank events, or widely followed market commentary. Commercial vendors
like RavenPack, Bloomberg, and the StockTwits enterprise feed sell cleaned, entity-tagged, and
time-stamped sentiment for institutional users, trading lower latency and broader coverage against
higher cost.

For a course project, static data beats a live feed. Public research datasets exist for exactly this purpose:
labeled StockTwits message collections and academic Twitter datasets tied to specific index
constituents are common starting points. Live platform interfaces change their terms, rate limits,
and pricing without notice, which makes results hard to reproduce. A cached dataset also lets us
document precisely which messages were analyzed, which matters for grading and audit.

## 2. Types of Data

The raw object is usually unstructured text: a post, reply, headline-style message, or comment.
This text may include slang, emojis, cashtags, hashtags, links, and informal abbreviations, and
it is rarely clean enough to use without preprocessing. Emojis and finance slang in particular
carry real sentiment that standard text pipelines will simply discard, so the cleaning step is
a genuine modeling decision rather than a mechanical formality.

The structured fields around the text are just as important as the text itself. A useful record
should include the timestamp (ideally to the second and in a documented time zone), the platform,
the ticker or topic tag, a post identifier, a user identifier where permitted, and engagement
measures like likes, replies, reposts, or views. These fields let us align messages with market
data, weight messages by reach, and separate broad attention from directional sentiment.
Time-zone handling matters more than it might seem: to avoid look-ahead bias, a message must be
assigned to the trading interval in which it was actually observable, not the one in which it was
later archived.

The final modeling variables are almost always derived rather than raw. Common examples include
daily message volume, average sentiment score, the share of bullish posts, the share of bearish
posts, the level of disagreement between bullish and bearish users, and lagged versions of each.
For trading or risk applications, the data are normally collapsed into an hourly or daily panel
indexed by ticker and time, so the sentiment series can be aligned against returns, volatility,
and traded volume. That panel form is what makes social data workable alongside the market data
an analyst already uses.

## 3. Quality of Data

Social-media data are timely and abundant. That does not mean the quality is there. Many posts
are jokes, repeated slogans, spam, or comments that merely mention a ticker without any real view
attached. Cashtags can be ambiguous, especially for short symbols that collide with common words
or other companies. Sentiment models can misread sarcasm, irony, and finance-specific slang: a
phrase that reads as positive in general English may carry a negative meaning in market context,
and the reverse is just as common. Because of this, sentiment scores should be validated against
a sample of human-labeled messages before being used.

A more serious issue is manipulation. Bots, coordinated campaigns, and pump-and-dump behavior
can manufacture both volume and sentiment on demand. This is especially dangerous for small or
illiquid securities, where a burst of online attention can actually move prices and create a
feedback loop that a naive model would mistake for genuine signal. Sampling is another concern:
rate limits, deleted posts, suspended accounts, and changing platform policies mean that the
dataset we observe can differ systematically from the true population of messages, and that
difference is rarely random.

Before treating any of this as a signal, we would remove duplicates and near-duplicates, filter
posts to the relevant tickers, and exclude obvious bots and low-quality accounts where possible.
Aggregating to a daily level reduces idiosyncratic noise, and lagging the sentiment variables
ensures we do not look ahead into the return window. None of these controls substitute for out-of-sample testing.
A compelling individual post is not evidence of a signal. A sentiment reading from June 2020 that
preceded a rally tells you nothing about whether the strategy would have worked in October 2019.

## 4. Ethical Issues

The first ethical issue is platform permission. Each platform's terms of service govern what can
be collected, stored, and redistributed, and those terms differ and change over time. A project
that respects them does not scrape data in ways the platform prohibits, even when doing so is
technically easy.

The second issue is privacy. Usernames should not be published from research datasets. Building
profiles of identifiable individuals from public posts is questionable even when it is technically
legal. Work at the aggregate level wherever possible. If a dataset contains personal identifiers,
remove or anonymize them - there is rarely a good reason to retain them for a finance application
- and treat any such data in line with prevailing data-protection standards.

The third issue is market integrity. Social-media sentiment reflects manipulation as readily as
genuine opinion, and a model that amplifies coordinated hype can increase the harm caused by
misleading campaigns rather than merely observing it. We treat social sentiment as an imperfect
measure of market attention, not as evidence of fundamental value. Any report built on these data
should disclose its limitations, the possibility of bias, and the specific risk of manipulation
clearly enough that a reader can weigh the signal for themselves.

## 5. Python Import and Data Structure

In practice, we would load a cached file with one row per post and convert it into a structured
panel. The key fields are the timestamp, the ticker, the text, and any existing sentiment label.
If no label exists, we score the text with a sentiment model and then aggregate by ticker and
date. The code below shows the shape of that workflow using the sample file included in the
project; the notebook uses the same kind of 24-post illustrative sample.

```python
import pandas as pd
import numpy as np

posts = pd.read_csv("data/social_sample.csv", parse_dates=["timestamp"])
posts["date"] = posts["timestamp"].dt.date

# Example if sentiment is already supplied:
daily = (
    posts.groupby(["ticker", "date"])
         .agg(
             message_volume=("text", "size"),
             mean_sentiment=("sentiment_score", "mean"),
             bullish_share=("label", lambda s: (s == "Bullish").mean()),
             bearish_share=("label", lambda s: (s == "Bearish").mean()),
         )
         .reset_index()
)

daily["net_bullishness"] = daily["bullish_share"] - daily["bearish_share"]
```

If the file has text but no score, a lexicon model like VADER gives a simple and transparent
first pass, and we use it in the notebook for that reason. For a more serious finance application,
a finance-tuned language model would typically do better because it handles market language,
negation, and mixed sentiment more accurately. Whichever scorer we use, the aggregation into a
ticker-by-date panel is the critical step, because that is the form that can actually be aligned
with return data and tested.

## 6. Exploratory Data Analysis

The notebook demonstrates the workflow with a small illustrative sample of 24 SPY-related posts.
That sample is only large enough to show the mechanics; it is far too small to support any trading
conclusion. We present it as a template, not as evidence. With a real dataset, we would run four
basic checks before any modeling.

Start with message volume plotted over time. Spikes often mark earnings announcements, macro
releases, central-bank comments, or viral narratives, and volume alone is frequently more
informative than tone. Next, look at the distribution of sentiment scores: are they balanced,
heavily skewed, or dominated by neutrals? A corpus where most posts score near zero limits how
much any sentiment signal can contribute. Compare bullish, bearish, and neutral counts to gauge
overall mood and its stability. Finally, align lagged sentiment against next-period returns to
see whether sentiment leads price or merely reacts to it. That last check is the one that
actually decides whether the data have any forecasting value.

Our interpretation stays deliberately cautious. Social sentiment tends to be a weak and noisy
signal, and it is often more useful as a measure of attention, crowding, or disagreement than as
a direct return forecast. A good exploratory analysis asks whether sentiment adds anything beyond
what price, volume, and volatility already tell you - not whether bullish posts predict positive
returns. This is the same principle we applied in Part 1: an attractive-looking signal has to
survive a fair, out-of-sample test before it means anything.

## 7. Short Literature Search

Sun et al. explain why alternative data can narrow information gaps in finance, while noting that
these datasets are heterogeneous, noisy, and hard to process. Their review places social-media
data in the behavioral category and connects it to stock prediction, investment decision-making,
and the measurement of market attention, which is the use case this guide is built around.

Bollen, Mao, and Zeng provided an early and widely cited demonstration that aggregated
social-media mood could relate to market movements, and their work helped establish the idea that
online mood might carry market-relevant information. Their result is worth knowing, though it has
proved difficult to replicate cleanly out of sample - which is part of why we treat it as
motivation rather than a firm finding. Hutto and Gilbert introduce VADER, a rule-based sentiment
model designed for social-media text, and we use it as a transparent baseline in the notebook.
Cookson and Niessner study investor disagreement on a social network and show that posts reveal
more than a single positive-or-negative tone; disagreement itself can be a signal. Renault studies
intraday online investor sentiment and return patterns and documents how sentiment and returns
interact within the trading day.

None of these papers claims social sentiment is a reliable stand-alone predictor. The consistent
finding is that the signal is real but conditional - it depends on how you clean the data, how
you aggregate it, when you measure it relative to the return window, and whether you test it on
genuinely fresh data. That is the picture we have tried to reflect in this guide.

## Practical Takeaway

The appeal of social-media sentiment data is real: it is faster than most fundamental data
sources, it captures what retail investors are actually discussing, and a basic version is not
expensive to build. But cheap does not mean clean. The noise, manipulation, and sampling
problems are not minor inconveniences - for smaller or more illiquid names, they can swamp the
signal entirely.

Use this as one additional lens alongside price, volume, volatility, and fundamental context. Do
not trade on it alone. And before relying on any such signal with real money, require out-of-
sample evidence from data that was not used to build the model.
