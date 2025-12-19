# Final Project README: Sentiment Signals and Market Reaction (CAR5)

Repository link: <PASTE YOUR GITHUB REPO LINK HERE>

## Introduction:

This project is an end to end predictive modeling study that tests whether sentiment extracted from financial text can predict short horizon market reactions around company events. Our dataset is organized at the event level, where each observation links a company event date and cleaned text content to a realized post event market outcome. The predictive task is to classify whether the abnormal five day reaction is positive, using CAR5 as the market outcome and CAR5_pos as the binary target. We build sentiment features from the text using VADER and FinBERT, train both a baseline model and a neural network model used in the notebook, and then benchmark predictions against realized outcomes using classification diagnostics and return based checks. Overall, the results suggest that predicting CAR5 direction from text based sentiment alone is difficult in this dataset. The models show weak separation between up and down outcomes, predicted probabilities are not consistently reliable as odds, and return sorting does not show a stable monotonic improvement in realized CAR5.

## Data Description:

The dataset is structured at the event level. Each observation corresponds to a company event and includes a ticker symbol, an event date, cleaned text content, and a sector label, along with the realized market outcome over the event window. The text is cleaned and standardized before feature extraction, and the outcome variables are computed so each row links language and market behavior. This event level structure makes it possible to train models on historical events and evaluate them on later events.

The project matters because markets respond quickly to new information, and financial language may contain signals about expectations, uncertainty, and surprise. If we can quantify that language using NLP tools, we can test whether these signals line up with abnormal price moves. Even when prediction is difficult, building a clean evaluation framework is useful because it tells us whether the signal is real, how stable it is out of sample, and what kinds of improvements would be needed to make the analysis stronger.

Our predictive task is to forecast whether the market reaction is positive over a short window after each event. We focus on CAR5, which is the cumulative abnormal return over a five day window around the event. Abnormal means we adjust for broad market movement so the outcome reflects company specific performance relative to a benchmark during the window. We create a binary target called CAR5_pos that equals 1 when CAR5 is positive and 0 when CAR5 is negative. This allows us to treat the problem as a classification task, where the model outputs a probability of an up move and a predicted label.

## Models and Methods: Overview of models and implementation

Feature engineering transforms raw text into numeric predictors. We use two sentiment tools. VADER provides fast rule based polarity scores that are easy to interpret, and we aggregate them into event level features such as average sentiment. FinBERT is a transformer model trained on financial language that can capture context beyond dictionary rules, and we convert its outputs into event level features such as an average sentiment score and uncertainty style measures like confidence and entropy. The notebook is designed to compare simpler feature sets against richer feature sets to test whether extra text information improves predictive performance.

We evaluate two models. Logistic regression serves as a baseline because it is simple, fast, and interpretable, and it helps us understand whether any measurable signal exists without adding complexity. The neural network in the notebook is implemented as an MLPClassifier pipeline that includes missing value imputation and feature scaling followed by a multilayer perceptron classifier. The motivation for the neural network is that it can learn nonlinear relationships and interactions among features that a linear model might miss. We evaluate both models out of sample using an existing train test split when available or a time based split using the event date, with the goal of measuring performance on unseen events rather than in sample fit.

## Results and Interpretation: Review of modeling results and interpretation of performance

To benchmark predictions against the market, we use both classification diagnostics and market outcome checks. The first key graph is the ROC curve, which measures how well the model ranks up events above down events across all probability thresholds. The summary statistic ROC AUC is the probability that a randomly chosen up event receives a higher predicted score than a randomly chosen down event, where 0.50 corresponds to random ranking. In our results, the ROC curves are close to the diagonal, which indicates weak separation between positive and negative CAR5 events out of sample. 

<img width="975" height="384" alt="download (5)" src="https://github.com/user-attachments/assets/d31812d1-8195-446c-b1b6-fd806fd9f866" />

The confusion matrix complements the ROC curve by showing how the model behaves at a specific threshold, which we set to 0.5. It breaks outcomes into true positives, false positives, true negatives, and false negatives, and it highlights whether the model is biased toward predicting one class. Logistic regression tended to predict up more often, which can inflate the number of predicted ups while reducing the ability to correctly identify down events. The neural network was less biased toward predicting up, but in our test run it did not translate into stronger direction accuracy. 

<img width="755" height="384" alt="Confusion matrix" src="https://github.com/user-attachments/assets/e7846e01-716b-4660-a449-acaafe4d0c4b" />


The predicted probability histogram checks whether the model produces meaningfully different probability distributions for events that truly ended up up versus events that truly ended up down. A strong model would place most up events at high probabilities and most down events at low probabilities, creating visible separation. In our results, the distributions overlap heavily and many predictions cluster near 0.5, which suggests the model is not confident and not strongly discriminative for CAR5 direction. Place the probability histogram figure here.

<img width="606" height="384" alt="download (1)" src="https://github.com/user-attachments/assets/38598061-5e19-4da9-8052-e3886bbcecc7" />

We then connect predictions to realized returns more directly using a decile sort. We group events into bins based on predicted probability of an up move and compute the average realized CAR5 in each bin. If the model provides a stable ranking signal, higher probability bins should show higher average CAR5. In our results, the pattern is not monotonic and the average CAR5 can swing across bins, which implies the ranking signal is unstable and that high predicted confidence does not reliably correspond to better realized abnormal returns. 

To provide an intuitive market facing summary, we also compute cumulative event level performance under simple rules. The baseline takes every event CAR5, the long only rule takes CAR5 only when the model predicts up, and the long short rule goes long when the model predicts up and short when the model predicts down. This is not a full trading backtest, but it shows whether the model would have improved outcomes relative to a simple baseline when returns are compounded across events. In our results, the model based curves do not consistently outperform the baseline, and the long short approach can be unstable when direction accuracy is weak. 

Finally, we examine whether performance differs by sector by computing accuracy within each sector group. This helps interpret where the sentiment to market relationship might be more consistent and where it might be weaker. Sector results should be interpreted cautiously, especially when sector counts are small, because small samples can make accuracy appear artificially high or low. Place the sector accuracy figure here.

<img width="984" height="384" alt="download (2)" src="https://github.com/user-attachments/assets/21fe64cc-2e78-4ec7-94f4-10c5cdf0f5e4" />

<img width="993" height="384" alt="download (3)" src="https://github.com/user-attachments/assets/fa679ad0-e7f0-41b6-b2ca-a6eb01f52e97" />

## Conclusion and Next Steps:

Overall, the project delivers a complete workflow for sentiment to market analysis, including data preparation, NLP feature extraction using VADER and FinBERT, modeling with both a baseline and a neural network, and benchmarking against realized abnormal returns with clear diagnostic graphs. The main finding is that predicting CAR5 direction from text based sentiment alone is difficult in this dataset. The diagnostic plots indicate limited separation, inconsistent probability calibration, and unstable return sorting, which means that neither the baseline nor the neural network provides a strong, consistent out of sample edge on CAR5. Even with these limitations, the repository provides a reproducible template for evaluating whether language contains measurable information about short horizon abnormal returns, and it clearly shows what was tested, how it was tested, and what improvements are most likely to matter next.

A natural next step is to expand the number of events to reduce noise and improve statistical power, because short horizon abnormal returns can be dominated by idiosyncratic moves. Another next step is to add stronger control variables and context features, such as recent volatility, prior price trend, firm size, and simple event surprise proxies, so the model is not relying on text alone. It would also be useful to test alternative horizons and rolling window validation so performance is measured under more realistic time based conditions. On the NLP side, improvements could include better text segmentation, consistent handling of long documents, and experimenting with richer text representations while keeping evaluation strictly out of sample.

