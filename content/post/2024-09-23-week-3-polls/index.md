---
title: 'Polls'
author: Jay Chooi
date: '2024-09-23'
slug: week-3-polls
categories: []
tags: []
---

Code for this post can be found [here](https://github.com/jeqcho/usa-2024-election-prediction/tree/main/content/post/2024-09-23-week-3-polls).

## Today, we are getting more polls than before

The number of polls conducted nationwide and in each state has increased across election cycles. In the plot below, we tallied the number of polls who were completed in August of each election cycle.





<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />
The number of nationwide polls increased 60\% from 525 in 2016 to 843 in 2024 (and take note that these are the ones ending in August). The same pattern of increase is also observed for each swing state. The question now is then how can we make use of this huge number of polls to predict the election? One way is to sort polls into different grades and use the polls that have higher grades in our predictive models.

## Are some polls better than others?


[538](https://projects.fivethirtyeight.com/polls/) employs a grading scheme to weigh each poll in their models. Here are the distributions of the grades of the polls. In 2024, they changed to using a numeric grade that factors in historical accuracy and methodological transparency.
<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />

According to their grading, not all polls are the same, with a small minority of polls getting the "A" grade under a unimodal distribution. When they switched to numeric grading in 2024, this distribution becomes bimodal, with an additional peak at the high-quality side. But how useful is the grading scheme?





Here we plotted the national poll approval for Trump from 2016 to 2024, from the top 20% polls and the bottom 20% polls. Here the poll approval means the probability that a person chosen at random will vote for Trump.

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />

Surprisingly, it turns out that the results from the bottom 20% polls coincide with the top 20% polls. We can clearly notice the variation across years but not across polls, yielding that perhaps there is no material difference in results on the type of polls used. Next, we will use aggregated state-level polls to predict the 2024 elections.

## Predicting the 2024 election using polls

Here we again used a random forest to predict the vote share of Trump in 2024. We initialized with 500 trees and trained it to predict state-level popular vote based on the following features: year, state, poll approval at 7 weeks before the election, 10 weeks and 15 weeks.

We chose 7 weeks because it is the most current data we have for now as of 23 September. 15 weeks was chosen because Biden droppped out 15 weeks before the election, and it would not be accurate to predict the vote share involving Harris using poll approvals more than 15 weeks ago when she wasn't running. Here are the results.












<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-12-1.png" width="672" />

Surprisingly, the random forest predicts that Democrats will win Georgia, Pennsylvania and Arizona, while Republicans will win Wisconsin, Michigan, North Carolina and Nevada. This is in contrast with betting markets, which forecasts the exact opposite except for Pennsylvania and Nevada.

Let's take a closer look at the model through the RMSE of Leave-One-Out Cross-Validation.


```
## [1] "LOO-CV Mean Squared Error:  10.9223005760208"
```

```
## [1] "LOO-CV Root Mean Squared Error:  3.30489040302712"
```

The RMSE is 3.3. Using this margin of error, our forecasts on the state winers is not certain to hold true, since all our forecasts gave vote shares where the 50\% mark is within the margin of error. We simply do not have any certainty on the swing states right now.
