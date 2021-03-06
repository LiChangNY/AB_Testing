Experiment Design by Lily Chang
========================================================

Click [here](https://docs.google.com/document/u/1/d/1aCquhIqsUApgsxQ8-SQBAigFDcfWVVohLEXcV6jWbdI/pub?embedded=Tru) to view information on the Free Trial Screener experiment. 

##Metric Choice
**For each metric, explain both why you did or did not use it as an invariant metric and why you did or did not use it as an evaluation metric.**

For invariant metrics, the selection process was no-brainer: Number of cookies, number of clicks and click-through-probability are my choices. However, choosing evaluation metrics was a bit complicated. My initial set of choice includes gross conversion, retention, and net conversion. However, this would require at least 4,739,879 pageviews in total for control and experiment groups, that is, about 118 days at 100% of daily traffic diverted to this experiment, which is too long for Udacity.

So I came to a second set of metrics which include gross conversion and net conversion only because retention requires a lot more pageviews. The minimum pageviews required is cut to only 679,300 (detailed calculations provided below.)

The answers below correspond to the second set of metrics choices. Both gross conversion and net conversion are required to be valid to launch the experiment.
* Number of cookies: Invariant metric. Since a cookie is the initial unit of diversion, number of cookies should be the same in both control and experiment group. 

* Number of user-ids: It's not invariant because user ids aren't tracked unless a student enrolls in the free trial, thus number of enrollment differ between the control and experiment group. Therefore, number of user-ids could be an evalution metric, but not ideal because it's not normalized.

* Number of clicks: Invariant Metrics. Clicking on “Start Free Trial” happens before the screener, so you would actually want both control and experiment groups to have the same number of clicks to start with. 

* Click-through-probability (#clicks/#cookies): Invariant Metrics. Both number of cookies and number of clicks are invariant metrics, so is click-through-probability. 

* Gross conversion(#enrollment/#clicks): Evaluation metrics. If the hypothesis holds true, the free-trial screener would turn away some students that are unable to commit more than 5 hours a week. Therefore, I might expect a lower gross conversion rate in the experiment group than the control group. A 1% reduction of gross conversion will be considered as practically significant and valid for launching the experiment.

* Retention (#payment/#enrollment): It’s a potential evaluation metrics but I ended up not using it because of unrealistic requirement for minimum pageviews. Because of it, I don't really need this metric to be valid in order to launch the experiment. But had I used it as an evaluation metric, I would expect this metric to be higher in the experiment group because, though both number of payments and enrollments are reduced in the experiment group than control group, the magnitude of decrease in payment should be smaller than that in enrollments. 

* Net conversion(#payment/#clicks): Although the free-trial screener results in fewer enrollments (because less-committing students are not enrolling), I would anticipate a higher share of users who clicked on the free trial to continue past the free-trial, to justify the launch of this experiment. This would align with the second half of the goal"—without significantly reducing the number of students to continue past the free trial and eventually complete the course." 

```{r echo=TRUE}
get_z_star = function(alpha) {
    return(-qnorm(alpha / 2))
}

get_beta = function(z_star, s, d_min, N) {
    SE = s /  sqrt(N)
    return(pnorm(z_star * SE, mean=d_min, sd=SE))
}

required_size = function(s, d_min, Ns=1:100000, alpha=0.05, beta=0.2) {
    for (N in Ns) {
        if (get_beta(get_z_star(alpha), s, d_min, N) <= beta) {
            return(N)
        }
    }
    
    return(-1)
}
```

##Measuring Standard Deviation
**List the standard deviation of each of your evaluation metrics. (These should be the answers from the "Calculating standard deviation" quiz.) **

**For each of your evaluation metrics, indicate whether you think the analytic estimate would be comparable to the the empirical variability, or whether you expect them to be different (in which case it might be worth doing an empirical estimate if there is time). Briefly give your reasoning in each case.**

Both of my evaluation metrics, gross conversion and net conversions, use number of cookies as the denomitor and unit of analysis. Since cookies is also the unit of diversion of this experiment, I can presume that an analytical estimate will be sufficient. 

```{r echo=TRUE}
##analytical estimate of standard deviation
print("Gross Converstion SD: ")
sqrt(0.20625*(1-0.20625) / 400)
#Uncomment below if you want to see the standard deviation of retention.
#print("Retention SD:")
#sqrt(0.53*(1-0.53) / (660/3200 * 400))
print("Net Conversion SD:")
sqrt(0.1093125*(1-0.1093125) /400)
```

##Sizing
###Number of Samples vs. Power
**Indicate whether you will use the Bonferroni correction during your analysis phase, and give the number of pageviews you will need to power you experiment appropriately. (These should be the answers from the "Calculating Number of Pageviews" quiz.)**

I tested the sample requirements for two designs. The first one uses gross conversion, retention and net conversion with no Bonferroni. However, this design requires an exorbitant number of pageviews(4,739,879), which means that even if I divert 100% of traffic to this experiment, it would take me about 118 days to get the results. This would be very risky for the company. 

Therefore, I dropped retention from the design and now it would only require 679,300 pageviews. 

```{r echo=TRUE}
##Calculate Number of Pageviews
sample_size_gross <- required_size(s= sqrt(0.20625*(1-0.20625) * 2), d_min = 0.01, alpha = 0.05, beta = 0.2 )
CTP <- 0.08
print("Total pageviews (Gross Conversion): ")
sample_size_gross / CTP *2

sample_size_retention <- required_size(s=sqrt(0.53*(1-0.53) * 2), d_min = 0.01, alpha = 0.05, beta = 0.2 )
retention_baseline <- 660/40000
print("Total pageviews (Retention): ")
sample_size_retention / retention_baseline *2

sample_size_net <- required_size(s= sqrt(0.1093125*(1-0.1093125) * 2), d_min = 0.0075, alpha = 0.05, beta = 0.2 )
print("Total pageviews (Net Conversion): ")
sample_size_net / CTP * 2
```

###Duration vs. Exposure
**Indicate what fraction of traffic you would divert to this experiment and, given this, how many days you would need to run the experiment. (These should be the answers from the "Choosing Duration and Exposure" quiz.)**

**Give your reasoning for the fraction you chose to divert. How risky do you think this experiment would be for Udacity?**

One rule of thumb I learned about duration from Udacity videos is that you might want to run your experiment long enough to cover a mix of weekends and weekdays. Further, the first payment incurs after 14 days of enrollment. In order to observe if the earliest enrollees have made the payments, we should let the experiment run for 2 weeks at least.

In terms of exposure, I think it's low-risk because it’s just a pop-up screener that does not have significant impact on the database or any backend systems. Even if students are suggested not to enroll in the free trial, they are not obligated to abide by the rule. They could still enroll in the free trial or access free materials. Therefore, I think I can expose 80% of the traffic to this experiment, which will give me a total of 22 days, enough to cover three weeks.

##Experiment Analysis
###Sanity Checks
**For each of your invariant metrics, give the 95% confidence interval for the value you expect to observe, the actual observed value, and whether the metric passes your sanity check. (These should be the answers from the "Sanity Checks" quiz.)**

**For any sanity check that did not pass, explain your best guess as to what went wrong based on the day-by-day data. Do not proceed to the rest of the analysis unless all sanity checks pass.**

From this point on, I will check against the data from the experiment: https://docs.google.com/spreadsheets/d/1Mu5u9GrybDdska-ljPXyBjTpdZIUev_6i7t4LRDfXM8/edit#gid=0

The calculations are provided below. Of all three invariant metrics, that is, number of cookies, number of clicks, and CTR, the observed values are included in confidence interval, meaning that the difference between control and experiment group is not statistically significant. So they passed sanity check. 

```{r echo=TRUE}
##Sanity Checks
###Number of cookies
print("Number of cookies")
prob <- 0.5
control <- 345543 
experiment <- 344660
z <- 1.96
se <- sqrt( prob * (1-prob) / (control + experiment))
m <- z * se
lower_ci <- prob - m; print("Lower bound"); print(lower_ci)
upper_ci <- prob + m; print("Upper bound"); print(upper_ci)
observed <- control/(control + experiment); print("Observed"); print(observed)

###Number of clicks
print("Number of clicks")
prob <- 0.5
control <- 28378 
experiment <- 28325
z <- 1.96
se <- sqrt( prob * (1-prob) / (control + experiment))
m <- z * se
lower_ci <- prob - m; print("Lower bound");print(lower_ci)
upper_ci <- prob + m; print("Upper bound"); print(upper_ci)
observed <- control/(control + experiment); print("Observed"); print(observed)

###CTR
print("CTR")
control <- 345543 
experiment <- 344660
control_click <- 28378  
experiment_click <-  28325
prob <- (control_click / control)
z <- 1.96
se <- sqrt( prob * (1-prob) / control)
m <- z * se
lower_ci <- prob - m; print("Lower bound"); print(lower_ci)
upper_ci <- prob + m; print("Upper bound"); print(upper_ci)
observed <- (experiment_click/experiment) ; print("Observed");print(observed)
```

##Result Analysis
###Effect Size Tests
**For each of your evaluation metrics, give a 95% confidence interval around the difference between the experiment and control groups. Indicate whether each metric is statistically and practically significant. (These should be the answers from the "Effect Size Tests" quiz.)**

At the 95% significance level, the confidence interval [-0.0291, -0.0120] of gross conversion does not include either 0 or practical significance (dmin= 0.01), so it's statically and practically significant. 

The confidence interval for net conversion [-0.0116, 0.0019] includes both 0 and the negative of the practical significance boundary (dmin = 0.0075). Hence, the difference on net conversion is neither statistically nor practically significant. 


```{r echo=TRUE}
##Effect Sizes Tests: No Bonferroni
####Gross conversion = #enrollment / #clicks. pooled p
control <- 17293
control_event <- 3785
experiment <- 17260
experiment_event <- 3423
control_prob <- control_event/control; print(control_prob)
experiment_prob <- experiment_event/experiment; print(experiment_prob)
pooled_prob <- (control_event + experiment_event) / (control + experiment); print(pooled_prob)
pooled_se <- sqrt((pooled_prob)*(1-pooled_prob)* (1/control + 1/experiment));print(pooled_se)
d = experiment_prob - control_prob; print(d)
dmin = 0.01
z <- 1.96
m <- z * pooled_se
lower_ci <- d - m; print(lower_ci)
upper_ci <- d + m; print(upper_ci)

####Net conversion = #payment / #clicks. pooled p
control <- 17293
control_event <- 2033
experiment <- 17260
experiment_event <- 1945
control_prob <- control_event/control; print(control_prob)
experiment_prob <- experiment_event/experiment; print(experiment_prob)
pooled_prob <- (control_event + experiment_event) / (control + experiment); print(pooled_prob)
pooled_se <- sqrt((pooled_prob)*(1-pooled_prob)* (1/control + 1/experiment));print(pooled_se)
d = experiment_prob - control_prob; print(d)
dmin = 0.0075
z <- 1.96
m <- z * pooled_se
lower_ci <- d - m; print(lower_ci)
upper_ci <- d + m; print(upper_ci)
```


###Sign Tests
**For each of your evaluation metrics, do a sign test using the day-by-day data, and report the p-value of the sign test and whether the result is statistically significant. (These should be the answers from the "Sign Tests" quiz.)**

The final project results show that the experiment group underperforms the control group in gross conversion in 19 out of 23 days, while net conversion in 13 out of 23 days.  

Using http://graphpad.com/quickcalcs/binomial1.cfm, it shows that the two-tail p value for gross conversion is 0.0026, which is smaller than alpha (0.05), indicating the result is statistically significant. 
But the two-tail p value for net conversion is 0.6776, so it didn't pass the sign test.

##Summary
**State whether you used the Bonferroni correction, and explain why or why not. If there are any discrepancies between the effect size hypothesis tests and the sign tests, describe the discrepancy and why you think it arose.**

The Bonferroni correction is only used to reduce the chances of identifying at least one significant result due to chance at the presence of multiple metrics. Because my experiments require both of two evaluation metrics, gross conversion and net conversion, to be valid for launching, no Bonferroni correction was used.

The results under the sign test and effect size test are consistent with each other, both showing that gross conversion is statistically significant while net conversion not. 


##Recommendation
**Make a recommendation and briefly describe your reasoning.**
     
The experiment hypothesis states that the goal of the screener is, first, to "set clearer expectations for students upfront, thus reducing the number of frustrated students who left the free trial because they didn't have enough time." The experiment result shows that, indeed, gross conversion is lower in the experiment group, meaning that fewer people are signing up after clicking on the free-trial button, so Udacity was able to lower the cost. But as to the second half of goal"—without significantly reducing the number of students to continue past the free trial and eventually complete the course", our results show that the net conversion is not statistically significant, and the confidence interval of net conversion does include the negative of the practial significance boundary, meaning that I can't be confident about the size of this effect at the 95% confidence level. Hence, it would be too risky to recommend for launching this experiment.

##Follow-Up Experiment
**Give a high-level description of the follow up experiment you would run, what your hypothesis would be, what metrics you would want to measure, what your unit of diversion would be, and your reasoning for these choices.**

After a student completes the check-out process and lands in the “You are enrolled in the free-trial” confirmation page, a screen will pop up to invite him to join an online study group. The study group is led by a veteran Udacity student and ranges from 15-20 people (a class size) to keep students close. The student can either sign up now or join the general discussion forum later. To ensure that the pop-ups are shown properly, the student will have to press "Yes to join" or "Not now" to proceed. 
The message will look something like below. 
“Hi, XXX. Welcome to Udacity XXXX course. I am XXX, the tutor of study group XXX and a Udacity alum as well. This Udacity study group is a small(10-15 people), interactive online community where you will be able to share experiences with your peer students who start around the same time as you, post questions, attend office hours, and discuss your course progress. If interested, please click ‘Join’ button below. If not, you can always interact with students and coaches later on the general discussion forum. “

The hypothesis is that by actively engaging new students in such a small study group, it will make them feel more attached to the community, confident and informed of what kind of support they can get from the Udacity community, therefore increasing the likelihood of continuing past the free-trial period. 

The initial unit of diversion is a user-id on the enrollment confirmation page and remain so throughout the experiment. Here are the metrics I will be looking at. 

* Number of user-ids: Number of users who complete the checkout and enroll in the free trial. This is an invariant metric as we want the experiment and control group to have the same number of new students. 

* Retention: Number of user-ids to remain enrolled past the 14-day boundary (and thus make at least one payment) divided by number of user-ids that enroll in the free trial. This is an evaluation metric. If the hypothesis holds true, we should expect a higher retention rate in the experiment group. 

##Resources
* http://www.aaos.org/news/aaosnow/apr12/research7.asp
* http://graphpad.com/quickcalcs/binomial1.cfm
* http://www.evanmiller.org/ab-testing/
* https://rstudio-pubs-static.s3.amazonaws.com/98017_0cc8015eeef84fbdbe4ebf893e31e1a3.html
