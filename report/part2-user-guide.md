# Part 2 - User Guide: Social-Media Sentiment Data

*This guide explains how we would source, structure, assess, and explore social-media sentiment
as an alternative dataset for finance. We write it as a practical reference: what the data are,
where they come from, what can go wrong, and how we would begin an analysis responsibly.*

## 1. Sources of Data

Social-media sentiment is a behavioral alternative dataset. It is built from public posts,
comments, tags, and reactions that reveal how investors or consumers discuss a company, security,
market event, or economic topic. Sun et al. classify social-media data as one of the behavioral
alternative-data categories because it captures public attention and sentiment more quickly than
many traditional sources, often ahead of formal disclosures or analyst updates.

The most useful finance-specific source is StockTwits, where users tag securities with cashtags
and frequently label their own views as bullish or bearish. Those self-reported labels are
valuable because they provide a ready-made sentiment target without any modeling. X (formerly
Twitter) is broader and faster, and it reaches a much larger population, but access has become
more restrictive and expensive, and the finance conversation is mixed with unrelated content that
must be filtered out. Reddit provides longer and more argumentative discussions in communities
such as r/investing, r/stocks, and r/wallstreetbets, which are useful for understanding retail
narratives and crowding. YouTube comments and transcripts can add context around earnings calls,
central-bank events, or widely followed market commentary. Finally, commercial vendors such as
RavenPack, Bloomberg, and StockTwits enterprise feeds sell cleaned, entity-tagged, and
time-stamped sentiment for institutional users, trading latency and cost against convenience and
coverage.

For a course project, and for reproducibility in general, we prefer a static research dataset or
a cached sample over a live feed. Public research datasets exist for exactly this purpose: labeled
StockTwits message collections and academic Twitter datasets tied to specific index constituents
are common starting points. Live platform interfaces change their terms, rate limits, and pricing
without notice, which makes results hard to reproduce. A cached dataset also lets us document
precisely which messages were analyzed, which is important for both grading and audit.

## 2. Types of Data

The raw object is usually unstructured text: a post, reply, headline-like message, or comment.
This text may include slang, emojis, cashtags, hashtags, links, and informal abbreviations, and
it is rarely clean enough to use without preprocessing. Emojis and finance slang in particular
carry real sentiment that ordinary text pipelines discard, so the cleaning step is itself a
modeling decision rather than a mechanical formality.

The structured fields around the text are just as important as the text itself. A useful record
should include the timestamp (ideally to the second and in a known time zone), the platform, the
ticker or topic tag, a post identifier, a user identifier where permitted, and engagement measures
such as likes, replies, reposts, or views. These fields let us align messages with market data,
weight messages by reach, and separate broad attention from directional sentiment. Time-zone
handling deserves special care: to avoid look-ahead bias, a message must be assigned to the
trading interval in which it was actually observable, not the one in which it was later archived.

The final modeling variables are almost always derived rather than raw. Common examples include
daily message volume, average sentiment score, the share of bullish posts, the share of bearish
posts, the level of disagreement between bullish and bearish users, and lagged versions of each.
For trading or risk applications, the data are normally collapsed into an hourly or daily panel
indexed by ticker and time, so that the sentiment series can be lined up against returns,
volatility, and traded volume. This panel form is what makes social data comparable with the
market data an analyst already uses.

## 3. Quality of Data

Social-media data are timely and abundant, but they are noisy, and their quality has to be earned
rather than assumed. Many posts are jokes, repeated slogans, spam, or unrelated comments that
merely mention a ticker. Cashtags can be ambiguous, especially for short symbols that collide with
common words or with other companies. Sentiment models can also misread sarcasm, irony, and
finance-specific slang: a phrase that reads as positive in ordinary English may be negative in
market context, and the reverse happens just as often. Because of this, sentiment scores should be
validated against a sample of human-labeled messages before they are trusted.

A second, more serious issue is manipulation. Bots, coordinated campaigns, and pump-and-dump
behavior can manufacture both volume and sentiment. This is especially dangerous for small or
illiquid securities, where a burst of online attention can actually move prices, creating a
feedback loop that a naive model would mistake for genuine signal. Sampling is a further quality
concern: rate limits, deleted posts, suspended accounts, and changing platform policies mean that
the dataset we observe can differ systematically from the true population of messages, and that
difference is rarely random.

Before treating any of this as a signal, we would apply several controls: removing duplicates and
near-duplicates, filtering posts to the relevant tickers, excluding obvious bots and low-quality
accounts where possible, aggregating to reduce idiosyncratic noise, lagging sentiment variables to
avoid look-ahead bias, and always comparing against a simple baseline such as past returns or
volume. The decisive test is out-of-sample performance on data not used to build the signal, not
a handful of individually persuasive posts.

## 4. Ethical Issues

The first ethical issue is platform permission. Each platform's terms of service govern
collection, storage, and redistribution, and those terms differ and change over time. A
responsible project respects them and does not scrape data in ways the platform prohibits, even
when doing so is technically easy.

The second issue is privacy. Even when posts are public, users generally did not write them to be
stored, modeled, and used in financial analysis. We should avoid publishing usernames, avoid
building profiles of identifiable individuals, and aggregate results wherever possible. If a
dataset contains personal identifiers, we would remove or anonymize them unless there is a clear,
documented, and permitted reason to retain them, and we would treat any such data in line with
prevailing data-protection expectations.

The third issue is market integrity. Social-media sentiment reflects manipulation as readily as
genuine opinion, and a model that amplifies coordinated hype can increase the harm caused by
misleading campaigns rather than merely observing it. For that reason we treat social sentiment as
an imperfect measure of market attention, not as evidence of fundamental value. Any report built
on these data should disclose its limitations, the possibility of bias, and the specific risk of
manipulation, so that a reader can weigh the signal accordingly.

## 5. Python Import and Data Structure

In practice, we would load a cached file with one row per post and convert it into a structured
panel. The key fields are the timestamp, the ticker, the text, and any existing sentiment label.
If no label exists, we score the text with a sentiment model and then aggregate by ticker and
date. The code below shows the shape of that workflow; the executable version is in the notebook.

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

If the file has text but no score, a lexicon model such as VADER gives a simple and transparent
first pass, and we use it in the notebook for that reason. For a more serious finance application,
a finance-tuned language model would usually do better, because it is more likely to read market
language, negation, and mixed sentiment correctly. Whichever scorer we use, the important step is
the aggregation into a ticker-by-date panel, because that is the object we can align with returns
and test.

## 6. Exploratory Data Analysis

The notebook demonstrates the workflow with a small illustrative sample of 24 SPY-related posts.
That sample is only large enough to show the mechanics; it is far too small to support any trading
conclusion, and we present it as a template rather than as evidence. With a real dataset, we would
run four basic checks before any modeling.

First, we would plot message volume over time, because spikes in volume often mark earnings
announcements, macro releases, central-bank comments, or viral narratives, and volume alone is
frequently more informative than tone. Second, we would plot the distribution of sentiment scores
to see whether the data are balanced, skewed, or dominated by neutral posts, since a heavily
neutral corpus limits how much a sentiment signal can do. Third, we would compare bullish,
bearish, and neutral counts to gauge the overall mood and its stability. Fourth, we would align
lagged sentiment with next-period returns to test whether sentiment leads price or merely reacts
to it, which is the question that decides whether the data have any forecasting value at all.

Our interpretation stays deliberately cautious. Social sentiment is usually a weak and noisy
signal, and it is often more useful as a measure of attention, crowding, or disagreement than as a
direct return forecast. A good exploratory analysis therefore asks whether sentiment adds
information beyond price, volume, and volatility, rather than assuming that positive posts should
predict positive returns. This mirrors what we found in Part 1: an attractive-looking signal has
to survive a fair, out-of-sample test before it means anything.

## 7. Short Literature Search

Sun et al. explain why alternative data can narrow information gaps in finance, while stressing
that these datasets are heterogeneous, noisy, and hard to process. Their review places
social-media data within the behavioral category and links it to stock prediction, investment
decision-making, and the measurement of market attention, which frames the use case for this
guide.

Bollen, Mao, and Zeng provide an early and widely cited example of using aggregated social-media
mood to study market movements, and their work helped establish the idea that online mood could
carry market-relevant information. Hutto and Gilbert introduce VADER, a rule-based sentiment model
designed specifically for social-media text, which is why we use it as a transparent baseline in
the notebook. Cookson and Niessner study disagreement on a social network of investors and show
that investor posts reveal more than a single positive-or-negative tone, pointing to disagreement
itself as a signal. Renault studies intraday online investor sentiment and return patterns and
documents how sentiment and returns interact within the trading day. Taken together, these papers
support a balanced view: social-media data can be informative, but their value depends on careful
cleaning, aggregation, timing, and honest out-of-sample validation.

## Practical Takeaway

Social-media sentiment is attractive because it is fast, cheap relative to many proprietary
datasets, and close to investor attention. Its weaknesses are that it is noisy, biased, and
vulnerable to manipulation. We would not use it as a standalone trading rule. We would use it as
one additional signal, combined with price, volume, volatility, and fundamental context, and we
would require out-of-sample evidence before relying on it in a portfolio decision.
