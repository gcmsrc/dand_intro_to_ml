# Udacity DAND - Intro to Machine Learning
Submission by Giacomo Sarchioni

## Introduction
This README file aims at guiding the reader in the understanding of my 
submission for the **Intro to Machine Learning Project**
of *Udacity Data Analyst Nanodegree*.

In this document I am going to provide:
* instructions on how to navigate through this repository;
* answers to the *free-response* questions asked [here](https://docs.google.com/document/d/1NDgi1PrNJP7WTbfSUuRUnz8yzs5nGVTSzpO7oeNTEWA/pub?embedded=true).

## Instructions
This repository is built as follows:
* *data*, i.e. a folder with original dataset, dumped dataset and classifier;
* *eda*, i.e. a folder with the EDA analysis I have performed on the dataset;
* *notebooks*, i.e. a folder with some Jupyter notebooks. The *Classification Full* notebook is very comprehnsive and covers all the steps I took while building the classifier;
* *scripts*, i.e. main scripts;
* *tools*, i.e. supporting scripts.


## Free-response questions
### Summarize for us the goal of this project and how machine learning is useful in trying to accomplish it

The goal of this project is that of building a model that classifies an Enron employee as a *Person of Interest* (`poi`), i.e. a person who was involved in the Enron accounting scandal. The classification is binary,
i.e. a person is either a poi or is not.
<br>In this context, a ML algorithm is a great tool to build a predictive model, withouth being constrained by any parametric model. ML also allows to evaluate the performance/quality of the predictions and, most importanly, to optimise the model on specific performance metrics.
<br>The dataset is a Python dictionary, where for every 
key (i.e. the name of an employee) there is a dictionary of values,
including financial and message-related variables. The original 
dataset is made of 146 observations, of which **only 18** are
actual poi. The dataset, therefore, is very unbalanaced.
<br>There are 14 financial variables (e.g. salary, bonust, etc.), 6 messages variables (e.g. number of emails sent, number of emails received, etc.) and 1 labelling feature (poi or non poi).
<br>
<br>
As I showed in the `Outlier Identification.html` file (available in the folder *notebook*), I have identified six outliers which I have 
completely removed from the final dataset. They are:

| Name                          | Rational for removal                                                        |
|-------------------------------|---------------------------------------------------------------|
| TOTAL                         | This observation is actually the sum of all the previous ones |
| SHAPIRO RICHARD S             | Incredibly high number of messages sent                       |
| KAMINSKI WINCENTY J           | Incredibly high number of messages received                   |
| KEAN STEVEN J                 | Incredibly high number of messages received                   |
| LOCKHART EUGENE E             | Apart from the name, there is no data for this person.        |
| THE TRAVEL AGENCY IN THE PARK | It doesn't seem an Enron employee :)                          |

<br>

### What features did you end up using in your POI identifier, and what selection process did you use to pick them? Did you have to do any scaling? Why or why not?*

The variables I ended up using are:


| Variable                       | New | Definition                                                                                                                                |
|--------------------------------|-----|-------------------------------------------------------------------------------------------------------------------------------------------|
| `poi`                          | No  |  Boolean, 0 if not POI, 1 if POI                                                                                                          |
| `to_messages`                  | No  | Float, number of messages received                                                                                                        |
| `expenses`                    | No  | Float, amount of expenses (USD)                                                                                                           |
| `from_poi_to_this_person`      | No  | Float, number of messages received from POI                                                                                               |
| `shared_with_poi_ratio`        | Yes | Float, number of messages received and shared with at least one POI as a proportion of total messages received                            |
| `shared_receipt_with_poi`      | No  | Float, number of messages received and shared with at least one POI                                                                       |
| `other`                        | No  | Float, other financial variable (USD)                                                                                                     |
| `to_poi_ratio`                 | Yes | Float, number of messages sent to POI as a proportion of total messages sent                                                              |
| `bonus`                        | No  | Float, bonus (USD)                                                                                                                        |
| `total_stock_value`            | No  | Float, total stock value (USD)                                                                                                            |
| `restricted_stock`             | No  | Float, restricted stock value (USD)                                                                                                       |
| `salary`                       | No  | Float, salary (USD)                                                                                                                       |
| `sqrt_wealth`                  | Yes | Float, sqrt transformation of wealth, i.e, the sum of salary, total_payments, bonus, total_stock_value, expenses, other, restricted_stock |
| `total_payments`               | No  | Float, total payments (USD)                                                                                                               |
| `exercised_stock_options`      | No  | Float, exercised stock options (USD)                                                                                                      |
| `sqrt_exercised_stock_options` | Yes | Float, sqrt transformation of exercised stock options                                                                                     |

I started by creating some new variables (see table above and Classification notebook).
<br>One of the variable is what I call `wealth`, which is simply the sum of most financial variables. I noticed, in fact, that financial features are in general quite correlated so I wanted to try a new feature which is just the sum of them.
<br>For messages variables, I created a series of ratios which, in my intention, should normalise these features. For example, the `to_poi_ratio` is the ratio between the absolute number of emails sent to poi and the total number of email sent. In this way, observations become comparable.
<br>
<br>
I only keep variables for which the percentage of missing values (i.e. `NaN`) is below 50% - to do that, I have actually built a function called `extract_fields_for_ml` in `dict_parser.py` module called `extract_fields_for_ml`.
<br>
I did an ANOVA test on all the original features vs the label (i.e. poi or not). I did it using `SelectKBest` (kind of a shortcut) and kept variables whose p-value is below 5% (this is the list at the beginning of this paragraph).
<br>
Since I have used algorithms like SVM, I have scaled all the features using `MinMaxScaler`. Scaling allows me to remove any influence due to values which are represented in different scale (e.g. wealth can reach millions of USD, while a percentage will have a much narrower range of values).

<br>

### What algorithm did you end up using? What other one(s) did you try? How did model performance differ between algorithms?

I tried a series of algorithms, including SVC, Logistic Regression, Decision Tree, K Nearest Neighbors, Ball Tree, Random Forest, etc. (a full list is available in the *Classification Full* notebook). The process I have used is the following:
* **optimise** the algorithms by using GridSearchCV. Since I wanted my algorithm to recall as many POF as possible, I have optimised on the scoring parametere `recall` (for POI true values only). Please note that I was doing optimisation on a 10-fold cross validation Stratified Shuffle Split (I wanted to perform optimisaton not just once).
* **evaluate** the algorithms using a 1,000-fold cross validation Stratified Shuffled Split.

There are two major things I would like to highlight here:
* for all the appropriate algorithms, I have set up the *class_weight* parameter equal to `balance` so that the fact that only 18 observations are true POIs (out of 140 total samples, after having removed outliers) was taken into account.
* my main script is optimising and evaluating the algorithms on `True` values of the label *poi*, i.e. optimisations and metrics are calculated so that the prediction power of true POIs is maximised. In the testing script provided, however, metrics are calculated globally, e.g. precision and accuracy are calculated on all predicted values.
<br>
I still prefer my original optimisation and evaluation process, but for the purpose of this exercise, I was then using global optimisation and evaluation. In my code, this is reflected in the two modules `optimiser.py` and `evaluate.py`. In the first one, I am setting the scoring parameter equal to *recall_micro* (i.e. the global score), while on the second I am setting a custom parameter called *tester* equal to `True` (in this case, the evaluation is the same as the one provided in `tester.py`).

<br>
The table below reports the global metrics for the algorithms I have used (optimisation on *recall*). The algorithm I ended up using is the first one, i.e. 

### What does it mean to tune the parameters of an algorithm, and what can happen if you don’t do this well?  How did you tune the parameters of your particular algorithm?

### What is validation, and what’s a classic mistake you can make if you do it wrong? How did you validate your analysis? 

### Give at least 2 evaluation metrics and your average performance for each of them.  Explain an interpretation of your metrics that says something human-understandable about your algorithm’s performance.

### Links
MD table generator: http://www.tablesgenerator.com/markdown_tables 
