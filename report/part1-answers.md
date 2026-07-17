# Part 1 - Assessing Models with Alternative Data

## Q1. Data Understanding

Sagaceta-Mejia, Sanchez-Gutierrez, and Fresan-Figueroa study daily market data for three ETFs: ECH, EWZ, and IVV. The raw data consist of the standard end-of-day fields available from Yahoo Finance: Open, High, Low, Close, Adjusted Close, and Volume. These six fields are the base dataset. The larger modeling dataset is created by transforming those fields into technical indicators, so most of the predictors are engineered rather than directly observed.

The authors apply a technical-analysis library to compute 210 technical indicators, giving 216 input features after the six raw fields are included. These indicators summarize different aspects of price and volume behavior. Moving averages describe trend and smoothing, RSI and Williams %R describe momentum, Bollinger Band measures describe volatility and location within a price envelope, and volume indicators such as On-Balance Volume describe buying or selling pressure. The paper groups the indicators into categories such as momentum, trend, volatility, volume, overlap, cycles, candles, performance, statistics, and utility indicators (Sagaceta-Mejia et al., sec. 2.3).

We understand the purpose of these indicators as feature engineering. Daily prices by themselves are noisy and difficult to model directly. Technical indicators compress the raw series into quantities that may be more stable, more interpretable, and more useful for classification. A broad indicator panel also creates a feature-selection problem: the goal is not to assume that every indicator is useful, but to identify the smaller subset that carries the most information about future market direction.

## Q2. Security Understanding - iShares MSCI Chile ETF (ECH)

ECH is an equity exchange-traded fund issued by BlackRock's iShares platform. It tracks the MSCI Chile IMI 25/50 Index and gives investors exposure to Chilean equities. Because it is a single-country emerging-market ETF, it is more concentrated than a broad U.S. market fund. In the paper's sector table, the largest exposures are Financials, Materials, Utilities, Consumer Staples, and Energy, which is consistent with the structure of the Chilean economy and the importance of mining, power, and banking activity (Sagaceta-Mejia et al., table 1).

For the paper's 2009-2020 sample, ECH's Open price ranges from 29.30 to 80.25, with a median of 46.48 and a mean of 50.10 (Sagaceta-Mejia et al., table 2). In our notebook replication over 2010-2019, the mean Adjusted Close is 35.30, the median Adjusted Close is 33.96, annualized volatility is 20.8%, cumulative return is -30.1%, and maximum drawdown is -60.2%. These numbers show why ECH is a useful example for this assignment: it has enough volatility and drawdown risk for directional prediction to matter, but it is still a listed ETF with clean daily price and volume data.

The paper treats the task as classification rather than regression. Instead of estimating the exact next price, the model predicts the direction of movement. This is a practical choice because trading and risk decisions often begin with the question of whether the asset is more likely to move up or down, while exact price forecasts are highly sensitive to noise. The paper defines its class variable from the change in Open price. Two other reasonable target definitions would be: first, a three-class return threshold where positive, negative, and flat days are separated by a small dead band; second, a k-day-ahead Close-to-Close direction, which would better match a multi-day holding period.

## Q3. Methodology Understanding

If we reorganize the paper, the first part of Section 2 should be treated as a Data section. That section would include the ETFs analyzed, the daily OHLCV fields, the construction of the technical indicators, the class-label definition, normalization, and cleaning. These steps describe how the data are obtained and prepared before any predictive model is estimated.

The model-related material belongs in a separate Methodology section. That section would describe the feature-selection methods, the neural-network classifier, and the cross-validation design. The feature-selection methods include Low Variance, Chi-squared, LASSO, Extra-Trees, Pearson correlation, Principal Feature Analysis, Mean Absolute Difference, and Dispersion Ratio. The classifier is a multilayer perceptron, and performance is evaluated with 10-fold stratified cross-validation (Sagaceta-Mejia et al., secs. 2.8-2.17).

It is useful to distinguish descriptive-statistical methods from model-based methods. Pearson correlation, Low Variance, Mean Absolute Difference, Dispersion Ratio, and Chi-squared are filter methods: they rank or screen variables using statistical properties of the data. LASSO and Extra-Trees are model-based because feature importance is obtained through an estimated model. Principal Feature Analysis uses the structure of the feature space to reduce redundancy. The MLP is the final predictive model rather than a feature itself.

Our proposed outline for the reorganized Section 3 (Methodology) is therefore:

- 3.1 Descriptive-statistical (filter) selectors: Low Variance, Chi-squared, Mean Absolute Difference, Dispersion Ratio, Pearson correlation.
- 3.2 Model-based selectors: LASSO, Extra-Trees, Principal Feature Analysis.
- 3.3 Consensus selection: the Selected(n) sets formed from features that several selectors agree on.
- 3.4 Predictive model: the multilayer perceptron classifier.
- 3.5 Validation: 10-fold stratified cross-validation and the accuracy comparison across selected sets.

The word "optimized" in the title does not mean that the authors tune every technical indicator's internal lookback period. It means that they optimize the selection of indicators. The procedure is to compute the full feature set, rank features under several selection methods, retain the most informative quartile from each method, and then define consensus sets based on how many selection methods agree. The model is then evaluated on each consensus set. In our reading, the main methodological idea is that a small set of repeatedly selected indicators can reduce noise and computational cost while preserving accuracy.

## Q4. Feature Understanding

In this project, a feature is one input variable used by the classifier. A raw field such as Open or Volume can be a feature, and an engineered technical indicator such as RSI, Balance of Power, or Bollinger Band Percent can also be a feature. A method is different from a feature: Pearson correlation and LASSO are procedures for selecting features. A model is different again: the MLP is the trained function that maps selected features into a directional prediction.

The feature categories in the paper reflect common technical-analysis groupings. Momentum features measure recent strength or weakness. Trend and overlap features summarize the direction and smoothness of price movement. Volatility features describe how widely prices move. Volume features measure trading pressure. Candlestick and utility indicators encode more specific market patterns or transformations. These categories matter because they help us interpret what kind of market information the final model is using.

The paper's feature-selection result is important because it shows that more variables do not automatically improve a model. Many indicators are correlated with one another, and some add noise rather than signal. The consensus-selected set, especially Selected(5), gives the model a smaller and cleaner input space. This improves interpretability and reduces computation while maintaining, and in some cases improving, predictive accuracy.

## Q5. Optimization Understanding

Cross-validation is a way to estimate how well a model generalizes beyond the data used to fit it. Instead of relying on one train-test split, the data are divided into folds. The model is repeatedly trained on most folds and tested on the remaining fold. The scores are then summarized across folds. This reduces dependence on a single split and gives a more stable view of out-of-sample performance.

In k-fold cross-validation, the data are split into k groups. The paper uses 10 folds and a stratified version of the procedure, which helps keep the class balance similar across folds. This matters because a directional classification problem can be misleading if one fold contains a very different proportion of up and down days than another.

Jaccard distance measures how different two sets are. The paper defines it as the size of the symmetric difference of the two sets divided by the size of their union, which is equivalent to one minus the ratio of the number of shared elements to the number of elements in either set. Its value always lies between 0 and 1: a distance of 0 means the two sets are identical, and a distance of 1 means they share no elements. In the paper, it is used to compare selected-feature sets across ETFs. The reported distances are J(ECH, EWZ) = 0.33, J(ECH, IVV) = 0.64, and J(EWZ, IVV) = 0.53, which show that the two emerging-market funds select more similar indicators to each other than either does to the developed-market benchmark (Sagaceta-Mejia et al., sec. 3). This measure is different from Euclidean distance, which measures the straight-line magnitude between numeric vectors, and from cosine distance, which measures the angle between numeric vectors while ignoring their magnitude. Jaccard distance is the more natural choice here because the objects being compared are sets of selected indicators rather than numeric vectors: it depends only on which indicators overlap, not on any numeric value attached to them.

The optimal solution in the paper is not simply the largest feature set or the model with the most inputs. It is the feature set that gives the best trade-off between predictive accuracy and model simplicity. The authors find that Selected(5), which is roughly 5% of the original feature set, performs as well as or better than the full feature panel while using far fewer inputs and less training time (Sagaceta-Mejia et al., secs. 3-4).

## Step 1 - Financial Problem

The financial problem is directional prediction for ETFs, especially emerging-market ETFs. Investors often need to decide whether to enter, exit, hedge, or reduce exposure. A model that estimates the direction of the next move can support those decisions, but only if the model is not overwhelmed by noisy and redundant indicators.

Emerging markets make the problem more important and more difficult. ECH and EWZ tend to be more volatile and more exposed to country-specific economic conditions than IVV, which tracks a broad developed-market benchmark. The paper's selected indicators differ across the ETFs, and the Jaccard distances show that the two emerging-market funds are more similar to each other than either is to IVV. For us, the practical implication is that feature selection should be performed by instrument rather than assumed to transfer unchanged from one market to another.

## Step 2 - Application

The main application is a more efficient technical-indicator model for trading and risk monitoring. The paper shows that the selected indicator set can reach roughly the same or better accuracy than the full 216-feature model, while sharply reducing the size of the neural network and the training burden. This is valuable because a smaller model is easier to maintain, easier to explain, and less likely to learn noise.

The most useful indicators vary by ETF, but the paper repeatedly identifies momentum, volatility, volume, and trend-related measures. This supports a practical interpretation: directional ETF prediction should combine several views of market behavior rather than rely on one indicator family. At the same time, the result should not be read as a standalone trading rule. A directional signal should be combined with transaction costs, liquidity, risk limits, and portfolio context before it is used for investment decisions.

## Step 3 - Replication

Our replication focuses on ECH and uses Pearson correlation as the feature-selection criterion. We do not reproduce the full 216-indicator design from the paper. Instead, we construct a smaller set of 21 technical indicators from daily OHLCV data and test whether the paper's central idea still appears in a simplified setting: a compact group of selected indicators can perform better than a larger, noisier panel.

The notebook uses ECH daily data from 2010-01-01 through 2020-01-01. After indicator warm-up rows are removed, the modeling sample contains 2,487 observations and 21 engineered indicators. The target is next-day Open direction, so the predictors are based on information available before the next Open movement is classified. The share of up days is 49.1%, which means the classes are close to balanced.

Pearson correlation ranks the indicators by absolute association with the class label. The strongest indicators in our run are Balance of Power, Increasing Close, daily return, Decreasing Close, and the 10-day EMA feature. This does not prove that each variable has causal forecasting power, but it does show which indicators have the strongest linear relationship with the next-day direction in this sample.

The cross-validation result supports the same broad conclusion as the paper. The best median 10-fold accuracy is obtained with only 2 selected indicators, at 80.12%. Accuracy remains near 79.7% with 4 to 8 indicators, then declines as more indicators are added. The full 21-indicator model reaches 74.70%. In our replication, adding every available indicator does not improve the model; it appears to add redundancy and noise.

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

We were careful not to take this high accuracy at face value. Our target is the next-day Open direction, but the next Open is very close to the current Close, so any indicator built from the close-minus-open gap, such as Balance of Power, the Increasing and Decreasing flags, and the daily return, nearly encodes the label instead of forecasting it. To test whether the accuracy reflects real skill, we re-ran the identical pipeline on a cleaner target, the next-day Close-to-Close direction, which cannot be recovered from the current close-minus-open gap. On that target the accuracy falls sharply, from 80.12% to 54.22% for the two-indicator model and to roughly 53% across the small subsets, only a little above the 49.3% base rate of up days. We read this as evidence that most of the headline accuracy comes from the mechanical overnight link between today's close and tomorrow's open, not from genuine predictive power. Importantly, the paper's central claim is unaffected: a small, well-chosen indicator set still matches or beats the full panel. It is the level of accuracy, not the feature-selection conclusion, that must be read with this caveat.

The three supporting figures for this section are the ECH adjusted-close price history, the ranked absolute correlations of each indicator with the target, and the accuracy-versus-number-of-indicators curve. Each is produced in the notebook with labelled and scaled axes.

Our recommendation is to treat a small, correlation-selected ECH indicator panel as a useful screening tool rather than as a complete trading system, and to judge it on the cleaner Close-to-Close target rather than the optimistic Open-to-Open one. The feature-selection result is stronger than the full-panel model and consistent with the paper's argument, but the directional signal itself is weak once the overnight artifact is removed. It should be evaluated with transaction costs, a time-aware (walk-forward) validation design, and live out-of-sample data before being used in portfolio decisions.
