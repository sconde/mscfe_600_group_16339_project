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

The primary financial problem addressed in this study is improving the prediction of short-term stock market movements for exchange-traded funds (ETFs), particularly those representing emerging markets. Investors and portfolio managers continuously face uncertainty when deciding the optimal time to buy, hold, or sell financial assets. Because stock prices are influenced by numerous economic, political, and behavioral factors, accurately forecasting market direction is extremely challenging. Poor investment decisions may lead to significant financial losses, while more reliable predictions can improve portfolio performance and risk management.

The authors attempt to solve this problem by combining technical analysis with machine learning. Instead of relying on all available technical indicators, they first identify the most informative indicators using several feature-selection techniques and then use these selected indicators as inputs to a neural network model. This approach reduces unnecessary information, lowers computational complexity, and improves prediction accuracy.

Predicting stock market movements in emerging markets is considerably more difficult than predicting developed markets. Emerging markets such as Chile (ECH) and Brazil (EWZ) generally experience higher volatility, lower liquidity, greater political uncertainty, and stronger sensitivity to economic events than developed markets like the United States (IVV). These characteristics make market behavior less stable and more difficult to model. The study demonstrates that different ETFs require different combinations of technical indicators, indicating that a model designed for developed markets cannot simply be transferred to emerging markets without adjustment. Therefore, optimizing feature selection for each market is essential for producing more reliable predictions and better investment decisions. This conclusion is supported by the paper's comparison of ECH, EWZ, and IVV and the differences observed in their selected feature sets and Jaccard distances.

## Step 2 - Application

The study demonstrates that careful feature selection can improve both the efficiency and effectiveness of stock market prediction models. Rather than using all 216 available technical indicators, the authors found that a small subset of approximately 5% of the indicators achieved prediction accuracy that was equal to or better than the full feature set while substantially reducing computational cost. This makes the model faster to train, easier to interpret, and less likely to suffer from overfitting.

The results have practical applications for investors, portfolio managers, financial analysts, and fintech companies. The model can be used as a decision-support tool to identify probable market direction before making investment decisions. It may assist investors in determining more appropriate entry and exit points, improving portfolio allocation, and managing risk more effectively. However, the model should complement—not replace—fundamental analysis, economic research, and professional judgment.

Several technical indicators consistently proved valuable throughout the study. Among the most useful were Balance of Power (BOP), which measures buying and selling pressure; Bollinger Band Percent (BBP), which identifies price position within volatility bands; Correlation Trend Indicator (CTI), which captures trend strength; Archer's On Balance Volume (AOBV), which reflects volume-based buying and selling activity; Williams %R, KDJ, Even Better SineWave (EBSW), and simple increasing/decreasing price indicators. Most of these indicators belong to the momentum, trend, volatility, and volume categories, suggesting that combining different types of market information produces better predictions than relying on a single indicator alone. The findings highlight that selecting high-quality features is more important than simply increasing the number of features included in the model.

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


## Part 2: Evaluating one particular type of alternative data

## User Guide: Credit Data as Alternative Data in Finance
## Introduction

Credit data represents a transformative category of alternative data within the commercial data types, providing financial institutions with unprecedented insights into borrower creditworthiness beyond traditional credit bureau reports. As Sun et al. identify, credit data are "obtained from credit bureaus" and are "beneficial for banks or institutions in determining the credit scores of firms and individuals to reduce loan risks" (Sun et al. 7). This user guide provides a comprehensive examination of credit data as alternative data, addressing each of the seven required components for practitioners seeking to incorporate this data type into financial analysis and credit decision-making.

The Financial Innovation review by Sun et al. positions credit data as a distinct subcategory of commercial data, alongside consumption data, enterprise characteristics data, and government data. The review highlights that credit data "optimize the accuracy of the current credit risk assessment system" and can be used to "identify potential misconduct, reducing loan risk" (Sun et al. 7). This guide draws on the framework established by Sun et al. and incorporates recent academic research demonstrating the predictive power of alternative credit data for default prediction and financial inclusion.

## 1. Sources of Data

Credit data as alternative data originates from diverse sources that extend beyond traditional credit bureaus. These sources provide complementary information that enhances traditional credit asessment.
Credit Bureaus and Traditional Sources: Traditional credit bureaus remain foundational sources, providing historical credit data (HCD) including payment history, credit utilization, length of credit history, credit mix, and new inquiries. However, as Sun et al. note, traditional credit data alone "limits the opportunities for some capable individuals and companies to obtain loans, especially in the case of start-ups" (Sun et al. 7).
Peer-to-Peer (P2P) Lending Platforms: P2P lending platforms have emerged as significant sources of alternative credit data. Sun et al. reference research using "approximately five million investor-loan-hour data from a Chinese peer-to-peer website" demonstrating that "individuals associated with specific relationships were found to have comparable credit records" (Sun et al. 7). These platforms capture borrower behavior, repayment patterns, and socio network connections that traditional bureaus do not capture.

Digital Footprints and Web Browsing Data: The seminal work by Rozo et al., explicitly referenced in Sun et al. as a key study on credit data, demonstrates the predictive value of web browsing variables. Using a large sample from a major digital retailer, Rozo et al. show that variables including "Number of website visits, Number of account sessions, Number of terms and conditions views and Number of mobile devices used" enhance the predictive accuracy of probability of default models (Rozo et al. 11). These variables are "readily available to the service provider" and can be collected "without specifically seeking this additional information from customers" (Rozo et al. 1).
Commercial Data Providers: Commercial data providers have emerged to aggregate and standardize alternative credit data. Experian, a major credit bureau, now offers alternative credit data services that incorporate "FCRA-compliant credit data that isn't typically included in traditional credit reports," including "alternative financial services data, rental payment data, full-file public records and account aggregation" (Experian).

## 2. Types of Data

Credit data encompasses multiple types of information that provide complementary perspectives on borrower creditworthiness.
Traditional Credit Data: Includes payment history (records of on-time and missed payments), credit utilization (ratio of credit used to credit available), length of credit history, credit mix (variety of credit types), and new inquiries (recent credit applications).
Transactional and Cash Flow Data: Cash flow data accessed through "permissioned bank APIs and payroll verification provide the strongest combination of predictive accuracy and regulatory defensibility" ("AI Credit Scoring" 1). Key variables include average monthly income and income volatility, recurring payment consistency across rent and utilities, overdraft frequency and time-to-recovery patterns, balance trajectory over 6-12 months, and discretionary spending ratios.
Web Browsing and Digital Behavior Data: Rozo et al. identify specific web browsing variables that enhance credit risk prediction: number of website visits (frequency of customer interaction with the online platform), number of account sessions (session-based engagement metrics), number of terms and conditions views (attention to compliance information), and number of mobile devices used (device diversity as a behavioral signal) (Rozo et al. 11).
Behavioral and Psychometric Data: Behavioral data captures borrower characteristics beyond financial metrics, including Economic Transaction Data (online shopping habits, payment histories, purchase patterns), Social Stability Data (geographic stability, employment consistency, social network connections), and Psychometric Data (personality traits, attitudes toward risk, financial literacy) ("The Role of Alternative Data" 1).

## 3. Quality of Data

Ensuring data quality is critical for reliable credit assessment and regulatory compliance.
Predictive Accuracy and Validation: Credit data quality is evaluated through standard metrics including Area Under the Receiver Operating Characteristics Curve (ROC-AUC), Gini coefficient, and Kolmogorov-Smirnov statistic. Rozo et al. demonstrate that "the inclusion of web browsing variables improves predictive performance over the longer term (a 12 month horizon), but not over a shorter term (a 3 month horizon)" (Rozo et al. 11). Research on micro-enterprise credit assessment shows that "external environmental shocks may reduce the model's precision in detecting potential defaults, the model's overall ranking stability remains intact" ("The Role of Alternative Data" 1). Multi-dimensional alternative data can "mitigate data volatility through feature complementarity, thereby enhancing model robustness" ("The Role of Alternative Data" 2).

Data Representativeness and Bias: Traditional credit data leaves significant gaps. Federal Reserve research cited by LendFoundry indicates that "roughly 32 million American adults are considered 'unscoreable' within traditional credit systems, including nearly 7 million credit-invisible consumers and approximately 25 million individuals with thin credit files" ("AI Credit Scoring" 1). Alternative data sources may suffer from nonrepresentative bias, requiring practitioners to assess whether data reflects the entire borrower population or particular subgroups.
Measurement Accuracy and Consistency: Alternative data may have "definitional differences, coverage deficiencies, and measurement error compared to survey data" (van Delden and Lewis 12). Practitioners should identify "gold standard" sources—typically census or well-established survey data—to serve as ground truth for validation (van Delden and Lewis 13).

Quality Assurance Methods: Cross-validate indicator data across multiple sources through triangulation. Adjust alternative data against traditional sources using post-stratification, multi-level regression, sample matching, propensity score weighting, or calibration methods (van Delden and Lewis 14). As LendFoundry emphasizes, "AI credit scoring models degrade over time. Economic shifts change the relationship between input signals and default behavior. Responsible deployment requires ongoing performance monitoring and scheduled retraining, not a one-time build" ("AI Credit Scoring" 1).

## 4. Ethical Issues

The use of alternative credit data raises significant ethical considerations that practitioners must address to maintain compliance, fairness, and stakeholder trust.

Privacy and Consent: "Any data sourced via open banking APIs requires clear borrower consent, this is both a legal requirement and a trust requirement" ("AI Credit Scoring" 1). Practitioners must ensure explicit, informed consent for data collection and usage. In developed countries, "variables such as phone usage data cannot be accessed without explicit permission due to data protection regulation laws" (Valdrighi et al. 20783). Data must be sufficiently anonymized to prevent identification of individuals.

Fairness and Discrimination: "Alternative data models can produce discriminatory outcomes even without discriminatory intent. Regular fairness audits, testing for disparate impact across protected classes, are a non-negotiable operational requirement" ("AI Credit Scoring" 1). The comprehensive literature review by Bahadori and Hassanzadeh emphasizes that "fairness and regulatory compliance also remain peripheral, with few studies explicitly integrating credit scoring systems with requirements like GDPR or Basel" (Bahadori and Hassanzadeh 3). "Lenders must issue adverse action notices explaining credit denials in plain language. A model that produces scores without reason codes creates FCRA and ECOA exposure" ("AI Credit Scoring" 1).

Regulatory and Legal Considerations: Alternative credit data in the US must comply with the Fair Credit Reporting Act (FCRA). Practitioners must "ensure the data intake architecture is FCRA-compliant by design" with "the permissioning layer, built into the platform, not treated as an afterthought" ("AI Credit Scoring" 1). Firms must undertake appropriate due diligence including Due Diligence Questionnaires, independent verification of vendor responses, and documentation of compliance steps (FISD Alternative Data Council 2).

Responsible Innovation: "There may be customer resistance to the use and/or collection of, for example, mobile usage data" (Valdrighi et al. 20784). "Psychometric and behavioural features may improve prediction" but are "constrained by self-report bias, limited cultural transferability, and privacy considerations" (Bahadori and Hassanzadeh 4). "Showing how data was collected and metrics extracted fosters feedback, research replicability, and further investigation" (van Delden and Lewis 14).

## 5. Literature Search: Papers Citing Research on Credit Data

Foundational Research: Rozo, Betty Johanna Garzon, Jonathan Crook, and Galina Andreeva. "The Role of Web Browsing in Credit Risk Prediction." Decision Support Systems, vol. 164, 2023, p. 113879.

This seminal paper demonstrates the predictive value of web browsing variables in credit risk assessment. Rozo et al. show that "web browsing variables, that can be easily collected by online retailers without specifically seeking this additional information from customers, can be incorporated into models of credit risk to predict PD at account level" (Rozo et al. 11). The study uses a large sample from a major digital retailer and finds that variables including number of website visits, account sessions, terms and conditions views, and mobile devices used enhance predictive accuracy. Key findings: web browsing variables enhance the predictive accuracy of probability of default models at account level; this predictive improvement holds in the absence of credit bureau data facilitating financial inclusion; and inclusion of web browsing variables improves predictive performance over the longer term (a 12 month horizon), but not over a shorter term (a 3 month horizon) (Rozo et al. 11).
Recent Extensions and Applications: Bahadori, Saeid, and Alireza Hassanzadeh. "A Comprehensive Literature Review on AI-Based Credit Assessment in Digital Lending: Exploring Models, Transparency, Alternative Data, and Real-Time Learning." Expert Systems with Applications, 2026.

This systematic literature review of 118 peer-reviewed articles (2018-2025) traces developments across five domains including alternative data. The review finds that "alternative data sources—including psychometrics and transactional streams—show clear benefits for thin-file borrowers, but issues of bias, privacy, and cultural transferability remain unresolved" (Bahadori and Hassanzadeh 3).

"The Role of Alternative Data in Micro-Enterprises' Credit Risk Assessment in China." Journal of Asian Economics, 2026. This empirical study categorizes alternative data into historical credit data and behavioral data, including economic transaction data and social stability data. Using random forest methodology, the study finds that multi-dimensional alternative data holds significant credit value and that behavioral data-based models demonstrate superior risk identification capability (1-2).

Practical Applications and Implementation: "AI Credit Scoring Using Alternative Data: A Complete Lender's Guide." LendFoundry, 2026. This industry guide notes that "roughly 32 million American adults are considered 'unscoreable' within traditional credit systems" and identifies that "cash flow data accessed through permissioned bank APIs and payroll verification provide the strongest combination of predictive accuracy and regulatory defensibility" (1).

## References

"AI Credit Scoring Using Alternative Data: A Complete Lender's Guide." LendFoundry, 2026.
Bahadori, Saeid, and Alireza Hassanzadeh. "A Comprehensive Literature Review on AI-Based Credit Assessment in Digital Lending: Exploring Models, Transparency, Alternative Data, and Real-Time Learning." Expert Systems with Applications, 2026.
Experian. "What is Alternative Data? A Guide for Lenders." Experian Insights, 2026.
FISD Alternative Data Council. "Alternative Data Identification Factors." Version 1.0, Mar. 2023.
"The Role of Alternative Data in Micro-Enterprises' Credit Risk Assessment in China." Journal of Asian Economics, 2026.
Rozo, Betty Johanna Garzon, Jonathan Crook, and Galina Andreeva. "The Role of Web Browsing in Credit Risk Prediction." Decision Support Systems, vol. 164, 2023, p. 113879.
Sun, Yunchuan, et al. "Alternative Data in Finance and Business: Emerging Applications and Theory Analysis (Review)." Financial Innovation, vol. 10, no. 1, 2024, pp. 1-32.
Valdrighi, Giovani, et al. "Best Practices for Responsible Machine Learning in Credit Scoring." Neural Computing and Applications, vol. 37, no. 25, 2025, pp. 20781-20821.
van Delden, Arnout, and David Lewis. "Quality Considerations for Administrative and Commercial Data." The Survey Statistician, no. 85, Jan. 2022, pp. 12-18.

