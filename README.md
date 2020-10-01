# A/B Testing Udacity's Free Trial Screener

In this project I consider an actual A/B experiment run by Udacity. I will fully design the experiment, analyze the results, and make data-driven recommendations.

The code is written in Python and contained in a Jupyter notebook.

## Experiment Overview

*[adapted from Udacity]* At the time of the experiment, Udacity courses have two options on the course overview page: "Start free trial", and "Access course materials". If the student clicks "Start free trial", they will be asked to enter their credit card information, and then they will be enrolled in a free trial for the paid version of the course. After 14 days, they will automatically be charged (unless they cancel first). If the student clicks "Access course materials", they will be able to view the videos and take the quizzes for free, but they will not receive coaching support or a verified certificate, and they will not submit their final project for feedback.

In the experiment, Udacity tested a change (called "free trial screener") where if the student clicked "Start free trial", they were asked how much time they had available to devote to the course. If the student indicated 5 or more hours per week, they would be taken through the checkout process as usual. If they indicated fewer than 5 hours per week, a message would appear showing that Udacity courses usually require a greater time commitment for successful completion, and suggesting that the student might like to access the course materials for free. At this point, the student would have the option to continue enrolling in the free trial, or access the course materials for free instead.

The hypothesis is that this might set clearer expectations for students upfront, thus reducing the number of students who enroll in the free trial (and then leave because they realize they don't have enough time) without significantly decreasing the number of students that continue past the free trial and eventually complete the course. If this hypothesis holds true, Udacity can improve the overall student experience and improve coaches' capacity to support students who are likely to complete the course, as fewer coaching resources are "wasted" on students that end up leaving the course during the free-trial phase (and therefore before making at least one payment).

## Experiment Design

We are going to test the introduction of the free trial screener using an A/B test. One subset of students, called "experiment group", will be shown the free trial screener when clicking the "Start free trial" button on the course overview page (they will be asked how much time they can devote to the course etc), while another subset of students, called "control group", will not be shown the free trial screener (after clicking the "Start free trial" button they will directly be prompted to provide credit card information). Let us start by defining the hypothesis we want to test using this A/B experiment.

### Hypothesis

Based on the high-level hypothesis discussed above, the null and alternative hypotheses for the experiment are:

**Null hypothesis (H0):** The free trial screener does not change the number of students who enroll in the free trial (and then leave because of lack of time), but changes the number of students who continue past the free trial.

**Alternative hypothesis (H1):** The free trial screener changes the number of students who enroll in the free trial, but does not change the number of students who continue past the free trial and complete the course.

### Unit of diversion

Since we are tracking whoever clicks the "Start free trial" button, the unit of diversion for the experiment is a cookie, although if the student enrolls in the free trial, they are tracked by the user-id from that point forward. The same user-id cannot enroll in the free trial twice. For users that do not enroll, their user-id is not tracked in the experiment, even if they were signed in when they visited the course overview page.

### Invariant metrics

We need to define what metrics we are going to track as invariant metrics, i.e. metrics that should not change between experiment and control groups and that will be therefore used as sanity checks on the experiment setup. We decide to consider the following metrics:

- **number of cookies:** number of unique cookies to view the course overview page. If the experiment is set up correctly, we should divert an equal number of unique cookies to the experiment and control groups.
- **number of clicks:** number of unique cookies that click the "Start free trial" button. This should also be invariant between experiment and control groups since cookies click this button before the free trial screener is triggered.
- **click-through-probability:** number of unique cookies that click the "Start free trial" button divided by the number of unique cookies that view the course overview page. Since this metric is defined as the ratio of the other two metrics, it should also be invariant between the two groups.

Throughout this experiment, cookie uniqueness is determined by day (same cookie visiting on different days would be counted multiple times). On the other hand, user-ids are automatically unique in time.

### Evaluation metrics

We consider the following metrics as evaluation metrics, i.e. metrics that should change between experiment and control if the alternative hypothesis holds true and that therefore will be used to determine the results of the A/B test:

- **gross conversion:** number of user-ids that enroll in the free trial divided by the number of unique cookies that click the "Start free trial" button. If H1 is true, the free trial screener will screen out students who can't devote enough time to the course. These student will not enroll in the free trial but would rather access materials for free. Under H1, the gross conversion in the experiment group should therefore be *lower* than the gross conversion in the control group.
- **net conversion:** number of user-ids that remain enrolled past the 14-day boundary divided by the number of unique cookies that click the "Start free trial" button. If H1 is true, the free trial screener does not reduce the number of user-ids who ultimately continue past the 14-day free trial. Under H1, the net conversion in the experiment group should *not* be *lower* than the net conversion in the control group. In fact, in order to make the introduction of the free trial screener meaningful from a business standpoint, we would like the net conversion in the experiment group to be slightly *higher* than the net conversion in the control group.
- **retention:** number of user-ids that remain enrolled past the 14-day boundary divided by the number of user-ids that enroll. If H1 is true, retention in the experiment group should be *higher* than retention in the control group, as students who went through the free trial screener and decided to complete the enrollment anyway will be less likely to leave the free trial before the 14-day boundary.

Based on these observations, we explicitly list the statistical and practical significances that we consider for the above metrics. The practical significance (*dmin*) is set by the relevant business stakeholders and represents the minimum difference that would have to be observed in the metric in order to launch the proposed change.

- **gross conversion:** under H0, the difference between the experiment and control gross conversions is zero. Under H1, the difference is non-zero. The practical significance is set at *dmin = -0.013*.
- **net conversion:** under H0, the difference between the experiment and control net conversions is non-zero. Under H1, the difference is zero. The practical significance is set at *dmin = 0.01*.
- **retention:** under H0, the difference between the experiment and control retentions is non-zero. Under H1, the difference is zero. The practical significance is set at *dmin = 0.01*.

In order to launch the proposed change (free trial screener), we decide *a priori* that we need to observe all metrics move in the desired direction and with the desired practical significance.

### Baseline values

We estimate the baseline values for the metrics. These values are derived by analyzing historical data collected before the experiment. In this case, we have:

- Number of unique cookies to view course overview page per day: 40,000
- Number of unique cookies to click "Start free trial" button per day: 3,200
- Enrollments per day: 660
- Probability of payment (i.e. staying enrolled in a course beyond the 14-day boundary), given enroll: 0.53.

From the above values we can derive:

- Click-through-probability on "Start free trial" button: 3,200/40,000 = 0.08
- Probability of enroll, given click on "Start free trial" button: 660/3,200 = 0.20625
- Probability of payment, given click on "Start free trial" button: 0.53 * 0.20625 = 0.109313.

### Variability of the evaluation metrics

Let us revisit each chosen evaluation metric and discuss what distribution they follow:

- **gross conversion:** this metric follows a binomial distribution with *n* the number of unique cookies to click the "Start free trial" button and *p* the "probability of enroll, given click". The expectation value for this metric is *n * p* and the variance is *n * p * (1-p)*. We can rescale this metric by *n* so that it is defined exactly as *p* and its standard deviation is *sqrt(p * (1-p)/n)*.
- **net conversion:** similarly, this metric can be defined as the "probability of payment, given click" (*p*) and *n* is the same as for the gross conversion case.
- **retention:** this metric is defined as the "probability of payment, given enroll" (*p*) with *n* the number of user-ids to enroll.

### Sizing

We need to determine how many pageviews (i.e. total number of unique cookies to view the course overview page) should be collected to adequately power the experiment given the chosen evaluation metrics. We have two groups (experiment and control) and we set *alpha* to 0.05 and *beta* to 0.2. The formula to compute the required sample size *n* (for each group) for each evaluation metric in the case of binomial distributions is given by eq. (2.33) of [this](http://vanbelle.org/chapters%5Cwebchapter2.pdf) and can be also found in the Jupyter notebook. We remind that the formula depends on *alpha*, *beta*, *p*, and *dmin* for each metric. Since we are tracking and testing multiple metrics, we might consider applying the Bonferroni correction to reduce the possibility of observing false positives. However, given that the three evaluation metrics are to some extent correlated, applying the Bonferroni correction might be overly conservative (as it considers multiple metrics as independent).

The explicit calculation of the sample size for each metric is performed in the Jupyter notebook. Here we present the results, which were obtained without applying the Bonferroni correction (although the code in the Jupyter notebook allows the interested user to apply it and obtain the corresponding results).

#### Gross conversion
- Required number of unique cookies (per group) to click the "Start free trial" button: 15,097
- Required number of unique cookies (per group) to visit the course overview page: 188,712.

#### Net conversion
- Required number of unique cookies (per group) to click the "Start free trial" button: 15,464
- Required number of unique cookies (per group) to visit the course overview page: 193,330.

#### Retention
- Required number of user-ids (per group) to enroll: 39,115
- Required number of unique cookies (per group) to visit the course overview page: 2,370,606.

If we want to be able to track all metrics, we need to consider the maximum of the required sizes, which would be 2,370,606 unique cookies per group to visit the course overview page. However, we observe that the sample size needed to test retention is much larger than those needed to test gross and net conversion. We might consider dropping retention if it requires too many pageviews, very high percentage of traffic diverted to the experiment, or a very long time to run the experiment. In that case, the number of pageviews needed per group would decrease to 193,330.

### Duration and Exposure

We know that our traffic consists of approximately 40,000 unique cookies that visit the course overview page per day (from the historical data at our disposal). If we consider all three metrics, with 100% diverted traffic it would take *2 * 2,370,606/40,000 = 118.5* days to run the experiment, which is definitely too long (and does not leave room for any other experiment).

If we drop retention from the set of evaluation metrics and divert 50% of traffic (20,000 unique cookies per day), we would need *2 * 193,300/20,000 = 19.3* days to run the experiment, which is acceptable. In addition, the experiment does not seem to be very risky for the involved users and for the business itself so a 50% traffic diversion seems like a reasonable percentage.

## Analysis

The results of the experiment are collected in csv files included in this repository (in the `data` folder). They show the number of pageviews, clicks, enrollments, and payments for each day in which the experiment was run. The precise definition of these quantities is the following:

- **Pageviews**: number of unique cookies to view the course overview page that day
- **Clicks**: number of unique cookies to click the "Start free trial" button that day
- **Enrollments**: number of user-ids to enroll in the free trial that day
- **Payments**: number of user-ids who enrolled on that day and remained enrolled past the 14-day boundary (thus making a payment).

Because of the 14-day free-trial window, enrollments and payments are tracked for 14 fewer days than the other quantities. In our analysis, we need to consider only the days for which we have enrollment and payment information as well, since the evaluation metrics depend on these. By summing over those days only, we obtain the following totals:

|             | Control | Experiment |
|-------------|---------|------------|
| Pageviews   | 212,163 | 211,362    |
| Clicks      | 17,293  | 17,260     |
| Enrollments | 3,785   | 3,423      |
| Payments    | 2,033   | 1,945      |

We observe that the totals for pageviews and clicks are above the minimum sample sizes required to test gross and net conversions, but below the minimum size required for retention. For the purpose of this analysis, we decide to drop retention from the set of evaluation metrics since we cannot draw statistically-sound conclusions about this metric using the data at our disposal. In a real-world scenario, we would have made sure that the experiment was run long enough to collect the minimum number of pageviews and clicks to test retention as well.

Finally, the sample sizes are large enough that we can approximate the binomial distributions for the evaluation metrics with normal distributions and therefore construct confidence intervals around the metric point estimates (thanks to CLT).

### Sanity Checks










### Results

## Recommendation

## Follow-Up




