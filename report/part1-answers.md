# Part 1 - Assessing Models with Alternative Data

## Q1. Data Understanding

Sagaceta-Mejia, Sanchez-Gutierrez, and Fresan-Figueroa study daily market data for three ETFs:
ECH, EWZ, and IVV. The raw inputs are the standard end-of-day fields from Yahoo Finance - Open,
High, Low, Close, Adjusted Close, and Volume. Those six fields are the base dataset.

The larger modeling dataset is built by passing those raw fields through a technical-analysis
library to compute 210 indicators, giving 216 inputs total once the six originals are included.
These indicators cover different facets of price and volume behavior: moving averages for trend
and smoothing, RSI and Williams %R for momentum, Bollinger Band measures for volatility and
position within a price envelope, On-Balance Volume for buying and selling pressure. The paper
organizes them into categories like momentum, trend, volatility, volume, overlap, cycles, candles,
performance, statistics, and utility indicators (Sagaceta-Mejia et al., sec. 2.3).

The purpose of computing all 210 is feature engineering. Raw daily prices are noisy and hard to
model directly, but transformed versions can stabilize the series and make patterns more legible
to a classifier. The broader question the paper is then asking is which of these 210 summaries
actually carry useful information about future direction and which are just adding noise. Computing
everything first and then selecting is the design choice.

## Q2. Security Understanding - iShares MSCI Chile ETF (ECH)

ECH is an equity exchange-traded fund from BlackRock's iShares platform. It tracks the MSCI Chile
IMI 25/50 Index and gives investors exposure to the Chilean equity market (BlackRock). Being a
single-country emerging-market fund makes it more concentrated than a broad U.S. market product.
In the paper's sector table, the largest exposures are Financials, Materials, Utilities, Consumer
Staples, and Energy, which reflects the structure of the Chilean economy - mining, power, and
banking are all important to the ETF story (Sagaceta-Mejia et al., table 1). Those weights should
be read as the paper-period description, not as a current holdings statement.

For the paper's 2009-2020 sample, ECH's Open price ranges from 29.30 to 80.25, with a median of
46.48 and a mean of 50.10 (Sagaceta-Mejia et al., table 2). In our notebook replication over
2010-2019, the mean Adjusted Close is 35.30, the median is 33.96, annualized volatility is 20.8%,
cumulative return is -30.1%, and maximum drawdown is -60.2%. The fund lost nearly a third of its
value over the decade and at its worst was down sixty percent from peak. For our purposes, ECH is
the most demanding of the three ETFs as a replication target. IVV tracks the S&P 500, a heavily
researched and relatively efficient market where technical indicators are unlikely to show strong
associations with next-day direction. EWZ would also work, but ECH has a sharper peak-to-trough
drawdown and more concentrated single-country structure, which gives the model sharper variation
to work with. The data are also clean, which matters for a reproducible replication.

The paper frames the problem as classification rather than regression - predicting direction
rather than an exact next price. That is a practical choice, since trading and risk decisions
often begin with the simpler question of which way an asset is more likely to move. Exact price
forecasts are sensitive to noise in ways that directional forecasts are somewhat less so. The
paper defines its class variable from the change in Open price. Two other reasonable target
definitions would be: first, a three-class return threshold that separates up, flat, and down days
with a small dead band around zero; second, a k-day-ahead Close-to-Close direction, which would
align better with a multi-day holding period.

## Q3. Methodology Understanding

The first part of Section 2 in the paper is really a Data section in disguise. It describes the
ETFs, the daily OHLCV fields, how the technical indicators are constructed, how the class label
is defined, and how normalization and cleaning are applied. Those are all data-preparation steps
and belong in a Data section, not mixed in with the modeling discussion.

The model-related material is separate. That belongs in Methodology: the feature-selection
methods, the neural-network classifier, and the cross-validation design. The selection methods
include Low Variance, Chi-squared, LASSO, Extra-Trees, Pearson correlation, Principal Feature
Analysis, Mean Absolute Difference, and Dispersion Ratio. The classifier is a multilayer
perceptron, evaluated using 10-fold stratified cross-validation (Sagaceta-Mejia et al.,
secs. 2.8-2.17).

It helps to separate the statistical-filter methods from the model-based ones. Pearson
correlation, Low Variance, Mean Absolute Difference, Dispersion Ratio, and Chi-squared are filter
methods: they rank or screen variables using statistical properties of the data alone, without
fitting a predictive model. LASSO and Extra-Trees are model-based because the importance ranking
comes from an estimated model. Principal Feature Analysis works differently, using the structure
of the feature space to remove redundancy. The MLP is the final predictive model rather than a
selection tool.

A reasonable outline for a reorganized Methodology section would be:

- 3.1 Descriptive-statistical (filter) selectors: Low Variance, Chi-squared, Mean Absolute
  Difference, Dispersion Ratio, Pearson correlation.
- 3.2 Model-based selectors: LASSO, Extra-Trees, Principal Feature Analysis.
- 3.3 Consensus selection: the Selected(n) sets formed from features that multiple selectors
  agree on.
- 3.4 Predictive model: the multilayer perceptron classifier.
- 3.5 Validation: 10-fold stratified cross-validation and accuracy comparison across selected
  sets.

One thing worth clarifying about the title: "optimized" does not mean the authors tune each
indicator's internal lookback period. It means they optimize the selection of which indicators
to use at all. The workflow is: compute the full set, rank features under multiple selection
methods, keep the top portion from each, then form consensus sets based on how many selectors
agree on a given feature. The model is then evaluated on each consensus set. The central claim
is that a small group of repeatedly selected indicators reduces noise and computation without
hurting accuracy.

## Q4. Feature Understanding

A feature is one input variable given to the classifier. Raw fields like Open or Volume can be
features, and engineered indicators like RSI, Balance of Power, or Bollinger Band Percent are
also features. A selection method is different from a feature: Pearson correlation and LASSO are
procedures for choosing which features to keep. A model is different again: the MLP is the
function that takes selected features and outputs a directional prediction.

The feature categories in the paper map onto standard technical-analysis groupings. Momentum
features measure recent price strength or weakness - essentially asking whether recent moves are
continuing or reversing. Trend and overlap features describe direction and smoothness. Volatility
features capture how wide the range of price changes has been. Volume features reflect trading
pressure: are buyers or sellers driving the move? Candlestick and utility indicators encode more
specific short-term patterns. These categories matter because they tell us what kind of market
information the model is actually relying on.

The feature-selection finding is that more inputs do not automatically produce a better model.
Many indicators move together, and some add confusion rather than clarity. The consensus-selected
set - especially Selected(5) - gives the classifier a smaller and cleaner input space. That
improves interpretability and reduces training cost, and in the paper's results it does not hurt
accuracy. That combination is the point.

## Q5. Optimization Understanding

Cross-validation estimates how well a model generalizes to data it was not trained on. Rather
than relying on a single train-test split, the data are divided into folds, the model is trained
on most of them and tested on the held-out fold, then the split rotates. Scores across all folds
are summarized. This gives a more stable view of out-of-sample performance than any single split
would provide. We report the median fold accuracy rather than the mean because the median is more
resistant to individual folds that cover an unusual market period - a crisis quarter or a
sustained rally could skew the mean in either direction and misrepresent the typical result.

In k-fold cross-validation, the data are divided into k groups. The paper uses 10 folds and a
stratified version that preserves class balance across folds. That matters here because a
directional classification problem can produce misleading aggregate accuracy if one fold has a
very different proportion of up and down days than the rest.

Jaccard distance quantifies how different two sets are. The formula is: the size of the symmetric
difference (elements in one set but not the other) divided by the size of the union (elements in
either set). That is the same as one minus the overlap fraction. The value sits between 0 and 1,
where 0 means the sets are identical and 1 means they share nothing. In the paper, Jaccard
distance is used to compare which indicators were selected for each ETF. The reported values are
J(ECH, EWZ) = 0.33, J(ECH, IVV) = 0.64, and J(EWZ, IVV) = 0.53 (Sagaceta-Mejia et al.,
sec. 3). ECH and EWZ, both emerging-market funds, end up selecting more similar indicators than
either does compared to IVV.

Jaccard is the right distance measure here because we are comparing sets of selected features,
not numeric vectors. Euclidean distance would require a numeric value attached to each feature,
and there is no natural one. Cosine distance captures the angle between numeric vectors, which
also does not apply. Jaccard depends only on which features overlap and which do not, and that is
the question being asked.

The "optimal" solution in the paper is not the model with the most inputs. The authors find that
Selected(5), roughly 5% of the original feature panel, performs as well as or better than the
full 216-feature model using far fewer inputs and less training time (Sagaceta-Mejia et al.,
secs. 3-4). Fewer features, comparable accuracy. That is the result.

## Step 1 - Financial Problem

The financial problem is directional prediction for ETFs, with a focus on emerging markets.
Investors need to decide whether to enter a position, exit, hedge, or reduce exposure. A model
that estimates the probable direction of the next move can support those decisions, provided it
is not overwhelmed by redundant or noisy inputs.

Emerging markets make the problem harder. ECH and EWZ are more volatile than IVV and more
sensitive to country-specific economic conditions. The paper's selected indicator sets differ
across the three ETFs, and the Jaccard distances make this concrete: the two emerging-market
funds are more similar to each other than either is to the developed-market benchmark. The
practical implication is direct - do not assume that an indicator set validated on one ETF will
transfer cleanly to another. Feature selection needs to be done by instrument.

## Step 2 - Application

The main application is a leaner, more interpretable technical-indicator model for trading and
risk monitoring. The paper shows that the selected indicator set reaches roughly the same accuracy
as the full 216-feature model while sharply reducing the number of inputs. A smaller model is
easier to maintain and harder to overfit. It is also easier to explain to a risk manager who
wants to know why the model took a position.

Across ETFs, the repeatedly selected indicators tend to come from momentum, volatility, volume,
and trend families. That supports a practical interpretation: ETF direction prediction should
combine several views of market behavior rather than rely on a single indicator type. But none of
this should be read as a ready-made trading rule. Any directional signal needs to be evaluated
alongside transaction costs, liquidity, risk limits, and portfolio context before it can inform
a real investment decision.

## Step 3 - Replication

Our replication focuses on ECH and uses Pearson correlation as the feature-selection criterion.
We do not reproduce the full 216-indicator design from the paper. Instead, we build a panel of 21
technical indicators from daily OHLCV data and test whether the paper's central claim holds in a
simplified setting: that a compact group of selected indicators performs better than a larger,
noisier panel.

The notebook uses ECH daily data from 2010-01-01 through 2020-01-01. After dropping warm-up rows
from rolling-window indicators, the modeling sample contains 2,487 observations and 21 engineered
indicators. The target is next-day Open direction, so the predictors are based on information
available before the next Open movement is classified. Up days account for 49.1% of the sample,
close to balanced.

Pearson correlation ranks indicators by absolute association with the class label. The strongest
in our run are Balance of Power, Increasing Close, daily return, Decreasing Close, and the 10-day
EMA feature. That ranking does not prove causal forecasting power - it shows which indicators
have the strongest linear relationship with next-day direction in this sample. Pearson is an
acceptable simplified filter here because it is fast, interpretable, and consistent with the
paper's design. What it misses is any non-linear association: a feature that predicts up
direction only when its value is either very high or very low - but not in the middle - would
score near zero in a correlation ranking even if it had real forecasting value. Pearson also
evaluates each indicator in isolation and cannot detect pairs of features that work together
while being individually weak. The paper uses eight selection methods precisely because no single filter covers all these
blind spots.

The first cross-validation run appears to support the paper's central claim. Best median 10-fold
accuracy comes from only 2 selected indicators, at 80.12%. Accuracy stays near 79.7% from 4 to 8
indicators, then drops as more indicators are added. The full 21-indicator model reaches 74.70%.

| Number of selected indicators | Median 10-fold accuracy |
|---:|---:|
| 2 | 80.12% |
| 4 | 79.72% |
| 6 | 79.68% |
| 8 | 79.72% |
| 10 | 78.92% |
| 12 | 78.51% |
| 15 | 76.71% |
| 21 | 74.70% |

The 80.12% figure looked good. Too good, honestly. A model predicting tomorrow's open direction
from today's technical indicators should not clear 80% unless something structural is happening,
and it turned out something was. Our target is next-day Open direction, but the next Open is very
close to the current Close in most markets. Any indicator built from the close-minus-open gap -
Balance of Power, the Increasing and Decreasing close flags, the daily return - is essentially
approximating the label rather than forecasting it. We ran a robustness check to test this.

We re-ran the identical pipeline using next-day Close-to-Close direction as the target. That
target cannot be approximated from today's close-minus-open gap. On that target, accuracy falls
from 80.12% to 54.22% for the two-indicator model and stays near 53% across the small subsets -
barely above the 49.3% base rate of up days. The headline accuracy was almost entirely the
mechanical overnight link between today's close and tomorrow's open, not real predictive skill.
The paper's central claim is not touched by this: a small, well-chosen indicator set still
matches or beats the full panel. The feature-selection conclusion holds; the level of accuracy
does not.

Before closing, two more issues need naming. First, we normalized features using the minimum and
maximum of the full dataset before splitting into folds. That means the test folds' ranges influence the normalization applied during
training - a mild look-ahead effect. In a production pipeline, normalization should happen inside
each fold using only training data. Second, we used StratifiedKFold rather than a time-series-
aware splitter. With shuffle=False, some training folds include observations that fall after their
test period, allowing future data to inform models tested on earlier dates. A proper time-series
setup would use an expanding-window walk-forward design: train on the first N days, test on the
next M, expand the training window by M, and repeat until the series is exhausted. That removes
future leakage entirely and would almost certainly reduce the reported accuracy further, because
the model would never see market conditions that fall after its test window during training. Both
issues compound the open-gap artifact: the 80.12% number is optimistic on at least three counts, and we flag all three here.

The three supporting figures are the ECH adjusted-close price history, the ranked absolute
correlations of each indicator with the target, and the accuracy-versus-number-of-indicators
curve. Each is produced in the notebook with labelled and scaled axes.

Do not take the 80.12% figure at face value. The cleaner target tells the real story, and on
that basis the directional signal is weak. For any real application, the next step would be a
proper walk-forward backtest with transaction costs and genuinely fresh out-of-sample data -
not a cross-validation result on the same decade of data that built the model.
