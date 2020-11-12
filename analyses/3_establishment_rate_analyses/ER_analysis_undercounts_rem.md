Establishment probability in worms
================

  - [Descriptive stats](#descriptive-stats)
  - [Models](#models)
      - [Model structure](#model-structure)
      - [Level of analysis - host or
        condition?](#level-of-analysis---host-or-condition)
      - [Weights](#weights)
      - [Random slopes](#random-slopes)
          - [Time of dissection](#time-of-dissection)
          - [Dose](#dose)
      - [Phylogenetic model](#phylogenetic-model)
  - [Model series for hypothesis
    testing](#model-series-for-hypothesis-testing)
      - [Hypothesis 1: recovery rates are higher later in the life
        cycle](#hypothesis-1-recovery-rates-are-higher-later-in-the-life-cycle)
      - [Hypothesis 2: recovery rates differ in intermediate vs
        definitive
        hosts](#hypothesis-2-recovery-rates-differ-in-intermediate-vs-definitive-hosts)
      - [Hypothesis 3: recovery rates increase with worm
        size](#hypothesis-3-recovery-rates-increase-with-worm-size)
      - [Hypothesis 4: recovery rates depend on host
        mass](#hypothesis-4-recovery-rates-depend-on-host-mass)
      - [Hypothesis 5: recover rate over time depends on step in the
        cycle](#hypothesis-5-recover-rate-over-time-depends-on-step-in-the-cycle)
  - [Conclusions](#conclusions)
  - [Dose as response variable](#dose-as-response-variable)

Parasitic worms often have complex life cycles, where they infect
multiple hosts in succession before reproducing. Each step in the life
cycle involves a risk, as some parasites will fail to infect the next
host in the cycle. But is this risk consistent? Using a dataset of
experimental infections from over a hundred worms, we examine what
impacts how likely it is for parasites to establish infection in their
next hosts.

# Descriptive stats

Number of infections (rows):

    ## [1] 2573

Number of species:

    ## [1] 127

Number of species in each phyla:

<div class="kable-table">

| parasite\_phylum | n\_distinct(Species) |
| :--------------- | -------------------: |
| Acanthocephala   |                   10 |
| Nematoda         |                   88 |
| Platyhelminthes  |                   29 |

</div>

Number of stages:

    ## [1] 157

Number of studies:

    ## [1] 153

Total number of exposed hosts:

    ## [1] 16876

Summary of doses:

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##       1      40     150    5201    2000 1000000

Summary of recoveries:

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##       0       3      15    1189      96  650000

Doses and thus recoveries vary over a large range.

# Models

For statistical modeling, we round the number of established/not
established worms to integer values. In this way, we can fit logistic
regression models, instead of treating recovery rate as a continuous
variable.

## Model structure

First, I compare how different model structures perform. I fit models
with `MCMCglmm` as it can also be used for phylogenetic analyses.

Here are the different tested models: (1) recovery rate as continuous
response, (2) recovery rate as proportion (recovered and not recovered
round to integers), (3) bivariate with log recovered and log dose as
response variables, (4) bivariate with recovered/not recovered as counts
(bivariate poisson). Notably, an error term was included in the GLMM to
account for overdispersion (additive overdispersion - equivalent to
adding an obs-level random effect to `glmer`). The preliminary models
include several presumably important predictors, including time after
infection, parasite stage, parasite size, and target host body mass. The
models include study as a random effect, but not phylogeny (yet).

The most complex model (bivariate poisson) had decent mixing for the
random effects, so the models did not have fitting issues.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-18-1.png)<!-- -->![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-18-2.png)<!-- -->

For each model, I extract their predictions.

Then I compare predictions from different models with observed values.
The bivariate LMM is the worse - it has many predicted recovery rates
above 1.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-20-1.png)<!-- -->

When reduced to only plausible values, the univariate GLMM model looks
best.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->
Residual plots. The bivariate LMM is the worst. The univariate GLMM
looks the best.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-22-1.png)<!-- -->

Another way to check model fit is to compare the distribution of
predictions with that of the observations. Here are density plots for
the predicted values. We can see that some models yield predictions more
closely matching the data than others, but it is a little hard to tell
with the substantial right-skew in the data.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-24-1.png)<!-- -->

Here are the distributions overlapping. The univariate GLMM performs
best, i.e. it comes closest to the observed data distribution.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-25-1.png)<!-- -->

The chains for the variance components in this “best” model mixed fine.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-26-1.png)<!-- -->

In these models, we included several presumably important predictors. We
test these more formally below, but here’s the model summary:

    ## 
    ##  Iterations = 1001:40981
    ##  Thinning interval  = 20
    ##  Sample size  = 2000 
    ## 
    ##  DIC: 5395545 
    ## 
    ##  G-structure:  ~Study
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## Study     2.932    2.119    3.872     2000
    ## 
    ##  R-structure:  ~units
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## units     2.643    2.457    2.837     1495
    ## 
    ##  Location effects: cbind(succeeded, failed) ~ log_dpi + nh_fac * to_int_def + log_ws + log_hm 
    ## 
    ##                              post.mean  l-95% CI  u-95% CI eff.samp  pMCMC    
    ## (Intercept)                   2.534379  1.616581  3.568250     2000 <5e-04 ***
    ## log_dpi                      -0.301680 -0.386426 -0.218348     1911 <5e-04 ***
    ## nh_fac2                      -0.858983 -2.191670  0.327480     2000  0.177    
    ## nh_fac3                      -0.502815 -2.273204  1.281900     2000  0.603    
    ## to_int_defDefinitive         -1.590765 -2.248841 -0.902042     2000 <5e-04 ***
    ## log_ws                        0.360602  0.274827  0.443968     2341 <5e-04 ***
    ## log_hm                       -0.071508 -0.129656 -0.006682     2000  0.021 *  
    ## nh_fac2:to_int_defDefinitive  2.420790  1.040702  3.693205     2000 <5e-04 ***
    ## nh_fac3:to_int_defDefinitive  1.025958 -0.702148  2.871177     2000  0.250    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

The model suggests that (i) recovery goes down with time post infection,
(ii) recovery is higher in second definitive hosts and lower in first
definitive hosts, (iii) that large larvae have higher establishment
rates, (iv) that establishment is unrelated to host mass, and (v)
considerable residual variance is between studies.

## Level of analysis - host or condition?

The data were often collected at the level of individual hosts. For
example, a study may have infected 5 hosts and then dissected them at 5
different time points. Or those 5 hosts may have been given different
doses. We did not pool these hosts to be at the ‘study level’ because we
wanted to account for variation due to e.g. when hosts were dissected.

However, some studies had a single condition, such as 100 hosts each
receiving 2 worm larva. The results of such an experiment may be
reported as a mean abundance (i.e. worms per exposed host). From such
results, we know the number of parasites given and the number recovered,
but not their distribution among hosts. Such results are at the study
level.

Logisitc regression accounts for the number of trials (worms given) and
the number of successes (worms recovered), so it should not matter that
dataset is a mix of results at the host level and the study level - the
trials do not change. Nonetheless, let’s compare models fit at either
level. We convert the dataframe to the ‘condition’ level. Any infections
within a study using e.g. different host species, doses, or dissection
times are kept separate, whereas any infection under the same conditions
are pooled.

Now we re-fit the logistic regression from above, but at the ‘condition’
level. We fit them with `lmer` since this is faster.

The fixed effect parameters are almost identical, even though the model
in which rows were sometimes individual hosts have more “observations”.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-31-1.png)<!-- -->

The estimated SE associated with each term is also the same in the two
models.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-32-1.png)<!-- -->

The random effects are also very similar, but they differ in ways that
we would expect. By pooling, the residual variance goes down because
there are fewer points within studies. The between study variance also
goes down slightly, presumably because pooling makes the study effects
more variable.

    ##  Groups   Name        Std.Dev.
    ##  obs      (Intercept) 1.5944  
    ##  study_rs (Intercept) 1.6006

    ##  Groups   Name        Std.Dev.
    ##  obs      (Intercept) 1.4427  
    ##  study_rs (Intercept) 1.3625

We can also compare the R<sup>2</sup> of the two models.

They are similar, but pooling reduces the variance explained.

<div class="kable-table">

|       VF |       VR |       VD |       VE | marg\_r2 | cond\_r2 | study\_var\_explained |
| -------: | -------: | -------: | -------: | -------: | -------: | --------------------: |
| 2.292428 | 2.561766 | 3.289868 | 2.542223 |    0.215 |    0.454 |                 0.239 |
| 1.291616 | 1.856367 | 3.289868 | 2.081492 |    0.152 |    0.370 |                 0.218 |

</div>

Since we proceed mainly at the condition level, let’s re-calculate some
of the descriptive statistics. Number of recovery rates (rows):

    ## [1] 1629

Number of species:

    ## [1] 127

Number of stages:

    ## [1] 157

Number of studies:

    ## [1] 153

Total number of exposed hosts:

    ## [1] 16876

Proportion without worm size:

    ## [1] 0.1436464

Proportion without host mass:

    ## [1] 0.0006138735

Proportion without host or parasite size

    ## [1] 0.1442603

## Weights

Most experimental infections are based on single individuals, but some
experiments report infection rates from groups of individuals.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-44-1.png)<!-- -->

We would like to give experiments with multiple individuals more weight,
but it is tricky to know how. Should an infection of 10 hosts have a
10-fold higher weight in the analysis than an infection with one animal?
Or 2-fold, if on a log10 scale? Let’s try weighting the analysis on a
log-10 scale, and then we’ll compare a non-phylogenetic model with and
without weighting.

The fixed effects are essentially identical in the models with or
without weighting, either at the host and condition level.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-48-1.png)<!-- -->

Or at the condition level.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-49-1.png)<!-- -->

The estimated SE associated with each term is also the same in the two
models.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-50-1.png)<!-- -->

Maybe weights have little impact because they are unrelated to recovery
rate. Experiments given higher weights are not more likely to have a
high or low recovery rate.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-51-1.png)<!-- -->

Given that (i) it is not clear how to weight by sample size and (ii) it
doesn’t affect the parameters, I proceed without weighting by sample
size.

## Random slopes

### Time of dissection

We did not average at the study level, because there are some important
sources of variation within studies, like different dissection times. In
the previous models, we fit a single time-dependent decrease in
recovery. This may be a little disingenuous because different parasite
species or stages may be lost from hosts at different rates. Here is the
relationship over the full data (infections pooled at condition level):

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-53-1.png)<!-- -->

As expected, recovery rates go down with days post infection, but there
is a lot of variation. Also, it is not clear that the relationship is
linear. For example, the dashed line is the relationship with log time,
which assume that hazards are exponential (i.e. they don’t accumulate
linearly).

Given that hosts were dissected on different time schedules in different
studies, each study could have a different relationship with time. Here
is a plot showing time-dependent recovery in 49 studies. We see that the
relationship is usually linear, though sometimes the log relationship
fits better (dashed lines). We can also see that sometimes there is a
negative relationship, sometimes none, and sometimes a positive
relationship.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-54-1.png)<!-- -->

Thus, let’s compare three models: 1) studies differ but have the same
time effect (random intercepts), 2) study x time (random slopes), and 3)
study x log time.

The random slopes model was a clear improvement, regardless whether time
was untransformed…

<div class="kable-table">

|        | npar |      AIC |      BIC |     logLik | deviance |    Chisq | Df | Pr(\>Chisq) |
| :----- | ---: | -------: | -------: | ---------: | -------: | -------: | -: | ----------: |
| m2\_ri |   11 | 15403.47 | 15461.11 | \-7690.737 | 15381.47 |       NA | NA |          NA |
| m2\_rs |   13 | 15330.37 | 15398.49 | \-7652.184 | 15304.37 | 77.10601 |  2 |           0 |

</div>

…or log transformed.

<div class="kable-table">

|         | npar |      AIC |      BIC |     logLik | deviance |    Chisq | Df | Pr(\>Chisq) |
| :------ | ---: | -------: | -------: | ---------: | -------: | -------: | -: | ----------: |
| m2\_ril |   11 | 15416.30 | 15473.94 | \-7697.148 | 15394.30 |       NA | NA |          NA |
| m2\_rsl |   13 | 15340.83 | 15408.95 | \-7657.416 | 15314.83 | 79.46498 |  2 |           0 |

</div>

The random slopes model with log time was also a better fit than the
model with untransformed time.

<div class="kable-table">

|         | npar |      AIC |      BIC |     logLik | deviance | Chisq | Df | Pr(\>Chisq) |
| :------ | ---: | -------: | -------: | ---------: | -------: | ----: | -: | ----------: |
| m2\_rs  |   13 | 15330.37 | 15398.49 | \-7652.184 | 15304.37 |    NA | NA |          NA |
| m2\_rsl |   13 | 15340.83 | 15408.95 | \-7657.416 | 15314.83 |     0 |  0 |           1 |

</div>

Calculating R<sup>2</sup> values for random slope models is more complex
than for random intercept models, because the variance explained by the
random effects depends on the levels of the random effect (Study) *and*
the covariate values (time). We modified code given
[here](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.12225)
to calculate R<sup>2</sup> for random slope models. We can see how the
conditional R<sup>2</sup> goes up as we include random slopes, though it
does not increase further when we use with log time. The reason is that
the overall error variance goes down when we use log time, but so does
the variance between studies. This suggests that using log time better
accounts for variation within studies, thereby reducing the differences
between studies.

<div class="kable-table">

| step                   | df\_used |       VF |       VR |       VD |       VE | marg\_r2 | cond\_r2 | study\_var\_explained |
| :--------------------- | -------: | -------: | -------: | -------: | -------: | -------: | -------: | --------------------: |
| random int             |       NA | 1.370258 | 1.939059 | 3.289868 | 2.048745 |    0.158 |    0.383 |                 0.225 |
| random slope           |        0 | 1.312760 | 2.804736 | 3.289868 | 1.773254 |    0.143 |    0.448 |                 0.305 |
| random slope, log time |        0 | 1.293801 | 2.866076 | 3.289868 | 1.752202 |    0.141 |    0.452 |                 0.311 |

</div>

Here is the summary of the simplest random slopes model (i.e. just time,
study, and their interaction). Interestingly, the fixed effect of dpi is
only weakly negative. Both the random intercept term (studies differ in
mean recovery) and the random slope term (studies differ in how recovery
changes with time) are positive. The covariance tends to be negative,
indicating that studies with above average recoveries tend to be
associated with stronger decreases over time. Conversely, if recovery is
low to begin with, then it has more of a chance to increase (positive
slopes).

    ## Generalized linear mixed model fit by maximum likelihood (Laplace
    ##   Approximation) [glmerMod]
    ##  Family: binomial  ( logit )
    ## Formula: cbind(succeeded, failed) ~ log_dpi + (1 + log_dpi | study_rs) +  
    ##     (1 | obs)
    ##    Data: filter(datx, !is.na(log_ws), !is.na(log_hm))
    ## 
    ##      AIC      BIC   logLik deviance df.resid 
    ##  15423.4  15454.9  -7705.7  15411.4     1388 
    ## 
    ## Scaled residuals: 
    ##      Min       1Q   Median       3Q      Max 
    ## -1.54996 -0.06337  0.00207  0.04398  1.61575 
    ## 
    ## Random effects:
    ##  Groups   Name        Variance Std.Dev. Corr 
    ##  obs      (Intercept) 1.8660   1.3660        
    ##  study_rs (Intercept) 6.4974   2.5490        
    ##           log_dpi     0.4524   0.6726   -0.77
    ## Number of obs: 1394, groups:  obs, 1394; study_rs, 146
    ## 
    ## Fixed effects:
    ##             Estimate Std. Error z value Pr(>|z|)   
    ## (Intercept) -0.71836    0.30973  -2.319  0.02038 * 
    ## log_dpi     -0.23783    0.08659  -2.747  0.00602 **
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Correlation of Fixed Effects:
    ##         (Intr)
    ## log_dpi -0.867

This plot shows how slopes and intercepts are negatively related at the
level of study. This correlation, though, disappears when the time
variable is centered (not shown).

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-61-1.png)<!-- -->

Since the random slopes model appears to be superior, let’s check the
predictions. Here are the random slope predictions within studies. The
model seems to capture the relationships well.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-62-1.png)<!-- -->

The number of studies where recovery increases looks similar to the
number that decreases. Let’s check each study individually. Here are the
number of studies with at least 4 dissection times

    ## [1] 84

We fit a simple logistic regression to each one to test whether recovery
usually increases or decreases over time.

Here’s the distribution of regression coefficients.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-65-1.png)<!-- -->

There were more studies with negative coefficients than positive
coefficients, but not many. Negative coefficients (decreased recover
over time) were a bit more likely to be statistically significant (P \<
0.001) than positive coefficients.

<div class="kable-table">

| beta\_dir |  n | sigs | perc\_sig |
| :-------- | -: | ---: | --------: |
| neg       | 49 |   38 |     0.776 |
| pos       | 35 |   22 |     0.629 |

</div>

How quickly recovery decreases with time is not dependent on life stage,
so e.g. recovery does not decrease faster for worms in the second host
compared to the first host.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-67-1.png)<!-- -->

One worry is if model fit varies with time post exposure, such as if
there was systemic undercounting in early dissections. The residual
plots across studies do not suggest that recovery rates are over- or
underestimated at different dpi.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-68-1.png)<!-- -->

Let’s make some plots with CIs for the change in recovery with time.
We’ll use `MCMCglmm` to fit the models.

Here is the overall decrease in recovery with time, as predicted by the
random intercepts model.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-70-1.png)<!-- -->

Here is the same plot but the predictions are from the random slopes
model. They are similar, though the CI is wider in the random slopes
model.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-72-1.png)<!-- -->

We can also pick a few studies for a manuscript figure that illustrates
the different time by recovery relationships.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-75-1.png)<!-- -->

### Dose

Another researcher-controlled variable that varied within studies was
dose. Some studies used multiple doses. Recovery rates might increase
with dose (e.g. more worms overwhelm immune responses) or decrease with
dose (competition among parasites, increased immune response). Here’s
how the pattern looks across the studies with multiple doses. Often the
relationship is flat or decreasing. And it is usually linear as
log-transforming dose did not provide a better fit in most cases.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-77-1.png)<!-- -->

However, across the whole data dose varies by several orders of
magnitude, so a linear relationship (solid line) does not fit the data
well. Dose probably needs to be log-transformed for analyses.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-78-1.png)<!-- -->

Here is the same plot, but with the x-axis log transformed. The
relationship between recovery and log dose (dashed line) fits better.
Higher doses are associated with lower recovery. This suggests that
researchers use higher doses when lower recovery rates are expected *OR*
that higher doses cause lower recovery.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-79-1.png)<!-- -->

At least within studies, higher doses seem to cause lower recovery. The
red lines on the next plot are the fits for each study and they are
often negative.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-80-1.png)<!-- -->

Given the broad range of doses, it is probably better to use log dose in
the analyses. For example, when we add untransformed dose as a fixed
effect to the random-intercpt model we fitted above, the effect is weak.

<div class="kable-table">

|         | npar |      AIC |      BIC |     logLik | deviance |    Chisq | Df | Pr(\>Chisq) |
| :------ | ---: | -------: | -------: | ---------: | -------: | -------: | -: | ----------: |
| m2\_ril |   11 | 15416.30 | 15473.94 | \-7697.148 | 15394.30 |       NA | NA |          NA |
| m2\_rid |   12 | 15414.99 | 15477.86 | \-7695.493 | 15390.99 | 3.311728 |  1 |   0.0687871 |

</div>

But it is very clear with log-transformed dose.

<div class="kable-table">

|          | npar |      AIC |      BIC |     logLik | deviance |    Chisq | Df | Pr(\>Chisq) |
| :------- | ---: | -------: | -------: | ---------: | -------: | -------: | -: | ----------: |
| m2\_ril  |   11 | 15416.30 | 15473.94 | \-7697.148 | 15394.30 |       NA | NA |          NA |
| m2\_rild |   12 | 15348.48 | 15411.36 | \-7662.241 | 15324.48 | 69.81548 |  1 |           0 |

</div>

A random slopes model with untransformed dose also has numerical issues
with fitting, so I continue with log dose.

Let’s compare three models: 1) studies differ but have the same dose
relationship (random intercepts), 2) study x dose (random slopes), and
3) with both random slopes terms (dose and dpi).

Adding a dose random slopes term to a random intercept model is an
improvement…

<div class="kable-table">

|          | npar |      AIC |      BIC |     logLik | deviance |   Chisq | Df | Pr(\>Chisq) |
| :------- | ---: | -------: | -------: | ---------: | -------: | ------: | -: | ----------: |
| m2\_rild |   12 | 15348.48 | 15411.36 | \-7662.241 | 15324.48 |      NA | NA |          NA |
| m2\_rsd  |   14 | 15280.91 | 15354.27 | \-7626.454 | 15252.91 | 71.5729 |  2 |           0 |

</div>

…as is adding the dose random slopes term to a model that already
includes a random slopes term for time dpi.

<div class="kable-table">

|                                     | npar |      AIC |      BIC |     logLik | deviance |    Chisq | Df | Pr(\>Chisq) |
| :---------------------------------- | ---: | -------: | -------: | ---------: | -------: | -------: | -: | ----------: |
| update(m2\_rsl, . \~ . + log\_dose) |   14 | 15264.04 | 15337.40 | \-7618.020 | 15236.04 |       NA | NA |          NA |
| m2\_rs2                             |   17 | 15195.35 | 15284.43 | \-7580.675 | 15161.35 | 74.69025 |  3 |           0 |

</div>

The main effect of dose explains about 4% of variation in recovery,
whereas the random slopes explains just 1-2%. By contrast, dissection
time random effect explains more of the variation within studies
(\~10%). This suggests that studies with different recovery rates use
different doses and that dose explains relatively little variation
within studies.

<div class="kable-table">

| step                             | df\_used |       VF |       VR |       VD |       VE | marg\_r2 | cond\_r2 | study\_var\_explained |
| :------------------------------- | -------: | -------: | -------: | -------: | -------: | -------: | -------: | --------------------: |
| random int, without dose         |       NA | 1.292118 | 1.857290 | 3.289868 | 2.081553 |    0.152 |    0.370 |                 0.218 |
| random int, dose main effect     |        1 | 1.614699 | 1.838703 | 3.289868 | 1.954761 |    0.186 |    0.397 |                 0.211 |
| random slope for dose            |        0 | 1.194490 | 2.311703 | 3.289868 | 1.791852 |    0.139 |    0.408 |                 0.269 |
| two random slopes, dose and time |        0 | 1.261558 | 3.603290 | 3.289868 | 1.459763 |    0.131 |    0.506 |                 0.375 |

</div>

Let’s check the predictions. Here are the predicted recoveries given the
study by dose interaction (no fixed effects). They look good, but it is
also clear that these relationships vary less than the study x time
relationships.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-89-1.png)<!-- -->

Like we did for dissection times, let’s fit a logistic regression to
each study with at least 4 different doses. There are only 27 of these
studies.

    ## [1] 27

We fit a simple logistic regression to each one to test whether recovery
usually increases or decreases over time.

Here’s the distribution of regression coefficients. There’s a negative
skew.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-92-1.png)<!-- -->

There were twice as many studies with negative coefficients than
positive coefficients, and they were much more likely to be considered
significant (P \< 0.001).

<div class="kable-table">

| beta\_dir |  n | sigs | perc\_sig |
| :-------- | -: | ---: | --------: |
| neg       | 17 |   14 |     0.824 |
| pos       | 10 |    5 |     0.500 |

</div>

So dose explains some variation within studies, but much less than
dissection time. And dose seems to explain a fair amount of the
differences between studies. However, this is also the main variation we
are trying to parse. I think that doses are chosen in response to
different recovery rates by researchers, not the other way around
(i.e. using high doses is not why recovery rates are lower).

Since dose is incorporated into the response variable and since it
probably does not cause the large variation across studies by itself, I
leave it out of the main models, but consider it again at the end of the
notebook.

## Phylogenetic model

Now let’s add phylogeny into the model. We fit `MCMCglmm` models because
it can incorporate phylogeny as random effects (phylogenetic covariance
matrix). We want to assess whether phylogeny accounts for variation
beyond study, since studies are usually on one species or a few closely
related species.

We’ll add phylogeny three ways: 1) by itself, 2) to a model with just
the study effect, and 3) to a model with study and the main fixed
effects. This tests whether phylogeny explains variation alone, beyond
study, and beyond life cycle characteristics, respectively.

When we compare models with DIC, we find that a model with just a
phylogenetic effect is not clearly better than one with just a study
effect (random intercept).

    ## delta DIC, study-only (random intercept) vs phylogeny-only model: -4.18 (higher is better)

Adding phylogeny to the random slopes model is also not an obvious
improvement

    ## delta DIC, study-only (random slopes) vs phylogeny+random slopes: 0.5 (higher is better)

And here’s what happens when we add phylogeny to the model with several
fixed effects (stage, worm and host size).

    ## delta DIC, without vs with phylogeny when fixed effects in model: 1.75 (higher is better)

In the phylogeny-only model, the phylogeny variance component is very
large relatively to the residual variance, it is almost certainly
inflated. Maybe some branches are associated with complete separation
(100% recovery rates).

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-99-1.png)<!-- -->
When we add phylogeny to the random slopes model, it is much lower and
not clearly different from zero, either without fixed effects…

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-100-1.png)<!-- -->

…or with fixed effects.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-101-1.png)<!-- -->

The phylogenetic effect competes with the study effect - when we add
phylogeny to a model with just study (random intercept) their variance
components are negatively correlated (not shown). This makes sense. A
given study usually focused on a single parasite species, so phylogeny
and study should explain some of the same variation.

Let’s compare R<sup>2</sup> values for the models with and without
phylogeny.

Weirdly, the R<sup>2</sup> table suggests phylogeny alone has a huge
effect compared to study alone (with or without random slopes). This is
due to the very large variance component for the phylogenetic effect.

<div class="kable-table">

| model                   | r2m                  | r2c                   |
| :---------------------- | :------------------- | :-------------------- |
| study only, random int  | 0.018 \[0.01-0.03\]  | 0.333 \[0.277-0.399\] |
| study only, rand slopes | 0.01 \[0.001-0.031\] | 0.421 \[0.36-0.486\]  |
| phylogeny only          | 0 \[0-0\]            | 0.655 \[0.519-0.791\] |

</div>

This seeming importance of phylogeny disappears when we add it to a
model with random slopes. It explains little variation beyond that
explained by the study effect.

<div class="kable-table">

| model                   | r2m                   | r2c                   |
| :---------------------- | :-------------------- | :-------------------- |
| study only, rand slopes | 0.01 \[0.001-0.031\]  | 0.421 \[0.36-0.486\]  |
| rand slopes + phylogeny | 0.009 \[0.001-0.024\] | 0.461 \[0.389-0.563\] |

</div>

Phylogeny might account for some variation beyond that explained by
study in a model with multiple fixed effects. However, this phylogenetic
effect bordered zero.

<div class="kable-table">

| model                                     | r2m                   | r2c                   |
| :---------------------------------------- | :-------------------- | :-------------------- |
| random slopes + fixed effects             | 0.145 \[0.101-0.195\] | 0.482 \[0.419-0.546\] |
| random slopes + fixed effects + phylogeny | 0.177 \[0.118-0.241\] | 0.594 \[0.495-0.7\]   |

</div>

I would not actually predict a strong phylogenetic effect, because
different life stages from closely related species might have very
different infection probabilities. Also, recovery rates are variable and
measured with considerable error, making phylogenetic effects harder to
detect.

Since “study” and “species” overlap substantially, maybe we should just
look at higher taxonomic levels. Since studies usually focus on the same
species, maybe we can detect taxonomic/phyla effects by looking at
whether e.g. parasite families have similar recovery rates across
studies. To test this, we replace parasite phylogeny with taxonomy in
the model, but only the higher taxonomic levels (i.e. phylum, class,
order, and family - presumably those levels won’t overlap much with the
study effect).

When we fit that model, we see that the taxonomic effects are usually
near zero. This suggests species from the same order, family, etc. do
not have more similar recovery rates than we would expect.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-110-1.png)<!-- -->

Here is the change in DIC when adding taxonomy to a model with fixed
effects.

    ## delta DIC, random slopes&fixed effx vs +taxonomy: -0.14 (higher is better)

And the R<sup>2</sup> values goes up when we add taxonomy, though the
conditional R<sup>2</sup> has a wide range because the taxonomic effects
were hard to estimate.

<div class="kable-table">

| model                                    | r2m                   | r2c                   |
| :--------------------------------------- | :-------------------- | :-------------------- |
| random slopes + fixed effects            | 0.145 \[0.101-0.195\] | 0.482 \[0.419-0.546\] |
| random slopes + fixed effects + taxonomy | 0.133 \[0.006-0.213\] | 0.655 \[0.496-0.985\] |

</div>

When we fit the taxonomic model with `lmer` and then perform a
likelihood ratio test to see if adding taxonomy improves the model, it
is not significant.

<div class="kable-table">

|             | npar |      AIC |      BIC |     logLik | deviance |    Chisq | Df | Pr(\>Chisq) |
| :---------- | ---: | -------: | -------: | ---------: | -------: | -------: | -: | ----------: |
| m2\_rsl     |   13 | 15340.83 | 15408.95 | \-7657.416 | 15314.83 |       NA | NA |          NA |
| m2\_rs\_tax |   17 | 15347.15 | 15436.22 | \-7656.573 | 15313.15 | 1.686049 |  4 |   0.7932502 |

</div>

The variance components for study and most higher taxonomic levels are
uncorrelated, with the exception of parasite family (some studies might
be the only ones on a particular worm family). This suggests the
negative correlation between study and phylogenetic effects in the
previous models is due to the same species/genera being studied.

    ##                                  (Intercept):(Intercept).study_rs
    ## (Intercept):(Intercept).study_rs                             1.00
    ## log_dpi:(Intercept).study_rs                                -0.94
    ## (Intercept):log_dpi.study_rs                                -0.94
    ## log_dpi:log_dpi.study_rs                                     0.78
    ## parasite_phylum                                             -0.01
    ## parasite_class                                               0.01
    ## parasite_order                                               0.01
    ## parasite_family                                             -0.21
    ## units                                                       -0.14
    ##                                  log_dpi:(Intercept).study_rs
    ## (Intercept):(Intercept).study_rs                        -0.94
    ## log_dpi:(Intercept).study_rs                             1.00
    ## (Intercept):log_dpi.study_rs                             1.00
    ## log_dpi:log_dpi.study_rs                                -0.92
    ## parasite_phylum                                          0.01
    ## parasite_class                                          -0.01
    ## parasite_order                                          -0.02
    ## parasite_family                                          0.10
    ## units                                                    0.17
    ##                                  (Intercept):log_dpi.study_rs
    ## (Intercept):(Intercept).study_rs                        -0.94
    ## log_dpi:(Intercept).study_rs                             1.00
    ## (Intercept):log_dpi.study_rs                             1.00
    ## log_dpi:log_dpi.study_rs                                -0.92
    ## parasite_phylum                                          0.01
    ## parasite_class                                          -0.01
    ## parasite_order                                          -0.02
    ## parasite_family                                          0.10
    ## units                                                    0.17
    ##                                  log_dpi:log_dpi.study_rs parasite_phylum
    ## (Intercept):(Intercept).study_rs                     0.78           -0.01
    ## log_dpi:(Intercept).study_rs                        -0.92            0.01
    ## (Intercept):log_dpi.study_rs                        -0.92            0.01
    ## log_dpi:log_dpi.study_rs                             1.00           -0.01
    ## parasite_phylum                                     -0.01            1.00
    ## parasite_class                                       0.01           -0.01
    ## parasite_order                                       0.01            0.03
    ## parasite_family                                     -0.03           -0.05
    ## units                                               -0.17           -0.02
    ##                                  parasite_class parasite_order parasite_family
    ## (Intercept):(Intercept).study_rs           0.01           0.01           -0.21
    ## log_dpi:(Intercept).study_rs              -0.01          -0.02            0.10
    ## (Intercept):log_dpi.study_rs              -0.01          -0.02            0.10
    ## log_dpi:log_dpi.study_rs                   0.01           0.01           -0.03
    ## parasite_phylum                           -0.01           0.03           -0.05
    ## parasite_class                             1.00           0.01           -0.01
    ## parasite_order                             0.01           1.00            0.16
    ## parasite_family                           -0.01           0.16            1.00
    ## units                                      0.05          -0.02           -0.07
    ##                                  units
    ## (Intercept):(Intercept).study_rs -0.14
    ## log_dpi:(Intercept).study_rs      0.17
    ## (Intercept):log_dpi.study_rs      0.17
    ## log_dpi:log_dpi.study_rs         -0.17
    ## parasite_phylum                  -0.02
    ## parasite_class                    0.05
    ## parasite_order                   -0.02
    ## parasite_family                  -0.07
    ## units                             1.00

Here’s a plot showing how the VC estimates for parasite order are
unrelated to those for study.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-116-1.png)<!-- -->

We can also make a plot to confirm that recovery rates do not vary with
taxonomy. Let’s look at parasite families. The black points are observed
recovery rates, while the red points are the medians for the family. It
looks like some families might have higher infection rates than others,
but recall that these may be single studies.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-117-1.png)<!-- -->

When we take the average recovery for a study, then the differences
among parasite families are much less conspicuous.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-118-1.png)<!-- -->

When we make the same plot for parasite orders, which is less confounded
with “study”, we see few compelling differences.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-119-1.png)<!-- -->

What about the fixed effects? Are they the same with or without a
phylogenetic random effect? They are rather tightly correlated,
suggesting that a model with or without phylogeny will not impact the
conclusions.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-120-1.png)<!-- -->

So, phylogeny does not have a clear effect on recovery, at least
independent from study effects. Since phylogeny (1) does not affect the
fixed effects, (2) is hard to estimate, and (3) intuitively should be
low in this dataset, we leave it out of the main analysis.

# Model series for hypothesis testing

We now want to build a series of models. We’ll use our random slopes
model as the “base” model: it includes just study and days until
dissection. We then add terms to this model to test explicit hypotheses.

Here is the base model summary:

    ## 
    ##  Iterations = 1001:40981
    ##  Thinning interval  = 20
    ##  Sample size  = 2000 
    ## 
    ##  DIC: 5535917 
    ## 
    ##  G-structure:  ~us(1 + log_dpi):study_rs
    ## 
    ##                                  post.mean l-95% CI u-95% CI eff.samp
    ## (Intercept):(Intercept).study_rs     6.464   3.4688   9.6873     1642
    ## log_dpi:(Intercept).study_rs        -1.340  -2.1846  -0.6306     1750
    ## (Intercept):log_dpi.study_rs        -1.340  -2.1846  -0.6306     1750
    ## log_dpi:log_dpi.study_rs             0.494   0.2714   0.7268     1806
    ## 
    ##  R-structure:  ~units
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## units     1.884     1.71    2.057     1696
    ## 
    ##  Location effects: cbind(succeeded, failed) ~ log_dpi 
    ## 
    ##             post.mean l-95% CI u-95% CI eff.samp pMCMC   
    ## (Intercept)  -0.72677 -1.33979 -0.14426     1605 0.018 * 
    ## log_dpi      -0.23555 -0.40921 -0.06856     1703 0.010 **
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Now let’s get to hypothesis testing. Throughout we use the random slopes
model with log-transformed time.

### Hypothesis 1: recovery rates are higher later in the life cycle

First, we test whether parasite life stage impacts establishment,
specifically whether the parasite is infecting the first, second, third
host, etc. To test this, we add ‘step in cycle’ to model.

When we look at the parameter estimates, we see that recovery is
significantly higher in second hosts than first hosts. The difference
between second and third hosts is not significant.

    ## 
    ##  Iterations = 1001:40981
    ##  Thinning interval  = 20
    ##  Sample size  = 2000 
    ## 
    ##  DIC: 5535917 
    ## 
    ##  G-structure:  ~us(1 + log_dpi):study_rs
    ## 
    ##                                  post.mean l-95% CI u-95% CI eff.samp
    ## (Intercept):(Intercept).study_rs    5.7528   3.0440   8.7994     1728
    ## log_dpi:(Intercept).study_rs       -1.2142  -1.9263  -0.5000     1861
    ## (Intercept):log_dpi.study_rs       -1.2142  -1.9263  -0.5000     1861
    ## log_dpi:log_dpi.study_rs            0.4453   0.2403   0.6608     1823
    ## 
    ##  R-structure:  ~units
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## units     1.888    1.722    2.057     1559
    ## 
    ##  Location effects: cbind(succeeded, failed) ~ log_dpi + nh_fac 
    ## 
    ##             post.mean l-95% CI u-95% CI eff.samp  pMCMC    
    ## (Intercept)  -1.42461 -2.10088 -0.79764     2000 <5e-04 ***
    ## log_dpi      -0.24448 -0.40622 -0.07784     2000  0.008 ** 
    ## nh_fac2       1.23316  0.59128  1.82068     2330  0.001 ***
    ## nh_fac3       2.00993  0.97239  3.14440     2000  0.001 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Model DIC is not much better with parasite stage, although the term was
significant.

    ## delta DIC, with and without 'next host': -0.78 (higher is better)

The R<sup>2</sup> table elucidates this contradiction. The overall model
fit is not better. Instead, the fixed effect of stage now explains about
5% of the variation, which was subtracted from the “study” variation. In
essence, different studies usually focus on different life stages, which
in turn differ in their infection rates.

<div class="kable-table">

| model                        | r2m                   | r2c                   |
| :--------------------------- | :-------------------- | :-------------------- |
| base random slopes, log time | 0.01 \[0.001-0.031\]  | 0.421 \[0.36-0.486\]  |
| parasite stage               | 0.066 \[0.029-0.113\] | 0.419 \[0.362-0.484\] |

</div>

So life cycle step is important in determining infection rates. Let’s
plot the predicted means for different life stages at day one post
infection (the intercept). Recovery rates increase with life cycle
length, but the difference between 2nd and 3rd stage larva is not clear,
since CIs overlap. For all stages, predicted recoveries are higher than
observed ones. This is because most hosts were dissected after several
days or even weeks.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-127-1.png)<!-- -->

When we plot the predictions at the median time of dissection (18 days),
then predictions better match observed medians.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-129-1.png)<!-- -->

Here are those predicted means:

<div class="kable-table">

| nh\_fac |       fit |       lwr |       upr |
| :------ | --------: | --------: | --------: |
| 1       | 0.1036774 | 0.0680465 | 0.1471915 |
| 2       | 0.2841728 | 0.2072505 | 0.3788944 |
| 3       | 0.4603294 | 0.2302976 | 0.6961032 |

</div>

The differences among life stages were similar when estimated 1 or 18
dpi (though the CIs are obviously lower at the median of 18 dpi) because
dissection times did not differ much among life stages.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-131-1.png)<!-- -->

### Hypothesis 2: recovery rates differ in intermediate vs definitive hosts

The first host in a cycle can be an intermediate host (in a complex
cycle) or a definitive host (in a direct cycle). Does this matter? To
test this hypothesis, we distinguish between cases where worms infect
intermediate vs definitive hosts. Before entering this term into a
model, let’s make sure that there are enough species and studies at each
combination of life stage and host type. Here are the number of species
in the different combinations. There are fewer intermediate host
infections later in the life cycle, as expected.

    ##       to_int_def
    ## nh_fac Intermediate Definitive
    ##      1           28         29
    ##      2            7         53
    ##      3            5         10

Here are the number of studies:

    ##       to_int_def
    ## nh_fac Intermediate Definitive
    ##      1           33         34
    ##      2            9         57
    ##      3            5         11

Both tables suggest that there are several species and studies in each
combination of life stage and int/def, so it is reasonable to add it to
the mixed model.

The model suggests parasites have lower recovery rates in definitive
hosts than in intermediate hosts.

    ## 
    ##  Iterations = 1001:40981
    ##  Thinning interval  = 20
    ##  Sample size  = 2000 
    ## 
    ##  DIC: 5535918 
    ## 
    ##  G-structure:  ~us(1 + log_dpi):study_rs
    ## 
    ##                                  post.mean l-95% CI u-95% CI eff.samp
    ## (Intercept):(Intercept).study_rs    6.6747   3.5934   9.7863     1685
    ## log_dpi:(Intercept).study_rs       -1.3492  -2.1081  -0.6359     1675
    ## (Intercept):log_dpi.study_rs       -1.3492  -2.1081  -0.6359     1675
    ## log_dpi:log_dpi.study_rs            0.4759   0.2768   0.7092     1713
    ## 
    ##  R-structure:  ~units
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## units     1.805    1.631    1.962     1548
    ## 
    ##  Location effects: cbind(succeeded, failed) ~ log_dpi + nh_fac + to_int_def 
    ## 
    ##                      post.mean l-95% CI u-95% CI eff.samp  pMCMC    
    ## (Intercept)           -0.68860 -1.41746  0.12379     2000  0.081 .  
    ## log_dpi               -0.25516 -0.43968 -0.08857     2000  0.002 ** 
    ## nh_fac2                1.73243  1.10785  2.41794     2000 <5e-04 ***
    ## nh_fac3                2.25507  1.10185  3.38078     2122 <5e-04 ***
    ## to_int_defDefinitive  -1.34096 -1.86547 -0.84942     2000 <5e-04 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Model DIC is not much better though.

    ## delta DIC, with int vs def: -0.51 (higher is better)

Nor is it clear that recovery rates at a given life stage depend on
whether the host is an intermediate or definitive host.

    ## delta DIC, with stage x int/def interaction: 2.34 (higher is better)

Here’s how R<sup>2</sup> changes. Like for parasite stage, the total
variance explained does not increase much by adding the intermediate vs
definitive host distinction, but more variation is attributed to the
fixed effects.

<div class="kable-table">

| model                  | r2m                   | r2c                   |
| :--------------------- | :-------------------- | :-------------------- |
| base + parasite stage  | 0.066 \[0.029-0.113\] | 0.419 \[0.362-0.484\] |
| \+ to int vs def       | 0.088 \[0.05-0.134\]  | 0.464 \[0.402-0.529\] |
| \+ stage by int vs def | 0.102 \[0.061-0.156\] | 0.477 \[0.414-0.541\] |

</div>

And here’s the plot. Predicted means are at the median dissection time
(18 dpi). Infections of first intermediate hosts are more successful
than infection of first definitive hosts. But the opposite is true for
second hosts.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-141-1.png)<!-- -->

The higher recovery in first or second intermediate hosts might be due
to being dissected later, i.e. the model thinks the observed recovery
rates are lower than they would be if they were dissected earlier.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-142-1.png)<!-- -->
Here are the median dissection times.

<div class="kable-table">

| nh\_fac | to\_int\_def | dpi |
| :------ | :----------- | --: |
| 1       | Intermediate |  21 |
| 1       | Definitive   |  15 |
| 2       | Intermediate |  30 |
| 2       | Definitive   |  23 |
| 3       | Intermediate |  14 |
| 3       | Definitive   |  15 |

</div>

But it likely is due to worms having larger infective stages when the
next host is the definitive host.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-144-1.png)<!-- -->

Thus, this term needs to be disentangled from our next model predictor,
worm size. But before moving onto that, let’s make a manuscript figure.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-145-1.png)<!-- -->

### Hypothesis 3: recovery rates increase with worm size

Since later life stages targeting definitive hosts have better
establishment rates, is this because they are larger? Let’s add the size
of infective parasite stages into the model. Moreover, if worm size
drives the pattern, we expect the effect of “step” and the “int vs def”
distinction to decrease once size is added. And there should not be an
interaction between next host and parasite size.

The worm size term is significant. Moreover, the difference among life
stages decreased.

    ## 
    ##  Iterations = 1001:40981
    ##  Thinning interval  = 20
    ##  Sample size  = 2000 
    ## 
    ##  DIC: 5535915 
    ## 
    ##  G-structure:  ~us(1 + log_dpi):study_rs
    ## 
    ##                                  post.mean l-95% CI u-95% CI eff.samp
    ## (Intercept):(Intercept).study_rs    6.5094   3.6586   9.7080     1793
    ## log_dpi:(Intercept).study_rs       -1.3824  -2.1825  -0.6821     1770
    ## (Intercept):log_dpi.study_rs       -1.3824  -2.1825  -0.6821     1770
    ## log_dpi:log_dpi.study_rs            0.4675   0.2675   0.6913     1804
    ## 
    ##  R-structure:  ~units
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## units     1.793    1.634    1.964     2000
    ## 
    ##  Location effects: cbind(succeeded, failed) ~ log_dpi + nh_fac * to_int_def + log_ws 
    ## 
    ##                              post.mean l-95% CI u-95% CI eff.samp  pMCMC    
    ## (Intercept)                    2.23323  0.95921  3.58997     1762  0.002 ** 
    ## log_dpi                       -0.23090 -0.39778 -0.07643     2000  0.006 ** 
    ## nh_fac2                       -0.92891 -2.37184  0.53058     2000  0.228    
    ## nh_fac3                       -1.91745 -4.10911  0.37167     2000  0.079 .  
    ## to_int_defDefinitive          -1.66008 -2.18524 -1.14513     2000 <5e-04 ***
    ## log_ws                         0.31208  0.19262  0.43453     2000 <5e-04 ***
    ## nh_fac2:to_int_defDefinitive   1.65638  0.25089  3.27412     2000  0.036 *  
    ## nh_fac3:to_int_defDefinitive   1.71841 -0.24823  3.67484     2000  0.088 .  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Again, despite being significant, adding worm size is not a huge
improvement, as judged by DIC.

    ## delta DIC, after adding parasite size: 0.37 (higher is better)

Neither is adding a worm size by stage interaction

    ## delta DIC, after adding parasite size x stage: -0.96 (higher is better)

Here is the summary from the more complex model with a worm size by
stage interaction. Most of the interactions with worm size are not
significant.

    ## 
    ##  Iterations = 1001:40981
    ##  Thinning interval  = 20
    ##  Sample size  = 2000 
    ## 
    ##  DIC: 5535916 
    ## 
    ##  G-structure:  ~us(1 + log_dpi):study_rs
    ## 
    ##                                  post.mean l-95% CI u-95% CI eff.samp
    ## (Intercept):(Intercept).study_rs    7.0394   4.0685  10.6804     1607
    ## log_dpi:(Intercept).study_rs       -1.5075  -2.3443  -0.7121     1611
    ## (Intercept):log_dpi.study_rs       -1.5075  -2.3443  -0.7121     1611
    ## log_dpi:log_dpi.study_rs            0.4962   0.2874   0.7530     1595
    ## 
    ##  R-structure:  ~units
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## units     1.783    1.615    1.937     1821
    ## 
    ##  Location effects: cbind(succeeded, failed) ~ log_dpi + nh_fac * to_int_def * log_ws 
    ## 
    ##                                     post.mean  l-95% CI  u-95% CI eff.samp
    ## (Intercept)                           8.31947   3.48587  13.70325     1857
    ## log_dpi                              -0.22773  -0.40486  -0.06465     2000
    ## nh_fac2                              -7.88660 -14.75494  -0.45307     2000
    ## nh_fac3                              -8.35166 -13.90112  -3.32366     1823
    ## to_int_defDefinitive                 -3.41165 -10.96260   4.52332     2000
    ## log_ws                                0.96925   0.44479   1.52825     1849
    ## nh_fac2:to_int_defDefinitive          3.87764  -5.58563  13.06673     2000
    ## nh_fac3:to_int_defDefinitive          3.89574  -4.16323  11.79627     2000
    ## nh_fac2:log_ws                       -0.78567  -1.73871   0.10818     2000
    ## nh_fac3:log_ws                       -0.54550  -1.20080   0.13821     2000
    ## to_int_defDefinitive:log_ws          -0.17328  -1.00656   0.77093     2000
    ## nh_fac2:to_int_defDefinitive:log_ws   0.22007  -0.87715   1.47799     2000
    ## nh_fac3:to_int_defDefinitive:log_ws   0.25936  -0.73998   1.26037     2000
    ##                                      pMCMC    
    ## (Intercept)                          0.003 ** 
    ## log_dpi                              0.011 *  
    ## nh_fac2                              0.032 *  
    ## nh_fac3                              0.002 ** 
    ## to_int_defDefinitive                 0.402    
    ## log_ws                              <5e-04 ***
    ## nh_fac2:to_int_defDefinitive         0.415    
    ## nh_fac3:to_int_defDefinitive         0.362    
    ## nh_fac2:log_ws                       0.096 .  
    ## nh_fac3:log_ws                       0.108    
    ## to_int_defDefinitive:log_ws          0.701    
    ## nh_fac2:to_int_defDefinitive:log_ws  0.704    
    ## nh_fac3:to_int_defDefinitive:log_ws  0.616    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Here’s how R<sup>2</sup> changes. The marginal R<sup>2</sup> again
increased at the expense of study-level variation, suggesting that some
of the differences among studies is related to different sizes of
infective stages. Adding the worm x stage interaction does not explain
much variation either. Interestingly, worm size only explains 1-2% of
the variation beyond that accounted for life stage, but by itself it
explains 7% of the variation, nearly as much as life stage alone. Thus,
worm size may explain the differences among stages.

<div class="kable-table">

| model                       | r2m                   | r2c                   |
| :-------------------------- | :-------------------- | :-------------------- |
| base, stage, and int vs def | 0.102 \[0.061-0.156\] | 0.477 \[0.414-0.541\] |
| \+ worm size                | 0.123 \[0.083-0.168\] | 0.457 \[0.397-0.519\] |
| \+ stage by worm size       | 0.135 \[0.093-0.181\] | 0.471 \[0.409-0.536\] |
| base + worm size            | 0.084 \[0.046-0.127\] | 0.401 \[0.346-0.458\] |

</div>

Now let’s look at the worm size parameter, first without life stage in
the model. Here is the posterior distribution for exp(worm size),
i.e. the odds ratio. It suggests that a 1 unit increase in log worm
size is associated with 1.37 times better odds of infection. Note the
odds are not the same as probability - the change in odds depends on the
baseline infection probability. For example, if the baseline infection
probability is 10%, then a 1 unit increase in log worm size increases
infection probability 13.7% (0.1 x odds ratio). But if the baseline is
50%, then the infection probability increases to 68.5% (0.5 x odds
ratio).

    ## 
    ## Iterations = 1001:40981
    ## Thinning interval = 20 
    ## Number of chains = 1 
    ## Sample size per chain = 2000 
    ## 
    ## 1. Empirical mean and standard deviation for each variable,
    ##    plus standard error of the mean:
    ## 
    ##           Mean             SD       Naive SE Time-series SE 
    ##       1.286391       0.046831       0.001047       0.001139 
    ## 
    ## 2. Quantiles for each variable:
    ## 
    ##  2.5%   25%   50%   75% 97.5% 
    ## 1.197 1.254 1.287 1.316 1.378

Since worm size is log transformed, we need to interpret this
coefficient with regards to proportional change. A change in 1 log unit
corresponds to a exp(1) or 2.72-fold increase in worm size, so the odds
ratio suggests the odds go up 37% with a 2.72-fold increase in worm
size. We can express this with more intuitive percents. Here is the
predicted percent increase in the odds of infection with a 10% increase
in worm size.

    ## 
    ## Iterations = 1001:40981
    ## Thinning interval = 20 
    ## Number of chains = 1 
    ## Sample size per chain = 2000 
    ## 
    ## 1. Empirical mean and standard deviation for each variable,
    ##    plus standard error of the mean:
    ## 
    ##           Mean             SD       Naive SE Time-series SE 
    ##      2.423e-02      3.554e-03      7.948e-05      8.639e-05 
    ## 
    ## 2. Quantiles for each variable:
    ## 
    ##    2.5%     25%     50%     75%   97.5% 
    ## 0.01729 0.02184 0.02431 0.02652 0.03103

And for a 100% increase (2-fold increase):

    ## 
    ## Iterations = 1001:40981
    ## Thinning interval = 20 
    ## Number of chains = 1 
    ## Sample size per chain = 2000 
    ## 
    ## 1. Empirical mean and standard deviation for each variable,
    ##    plus standard error of the mean:
    ## 
    ##           Mean             SD       Naive SE Time-series SE 
    ##      0.1905573      0.0300424      0.0006718      0.0007307 
    ## 
    ## 2. Quantiles for each variable:
    ## 
    ##   2.5%    25%    50%    75%  97.5% 
    ## 0.1328 0.1701 0.1909 0.2097 0.2489

we can double check our calculation by refitting the model, but using
log base 2 instead of ln. In this case the exp coefficient should
correspond to the change in the odds with a doubling of worm size.

    ##   log_ws2 
    ## 0.1938369

The odds ratio for worm size is a bit higher in the model including life
stage.

    ## 
    ## Iterations = 1001:40981
    ## Thinning interval = 20 
    ## Number of chains = 1 
    ## Sample size per chain = 2000 
    ## 
    ## 1. Empirical mean and standard deviation for each variable,
    ##    plus standard error of the mean:
    ## 
    ##           Mean             SD       Naive SE Time-series SE 
    ##       1.368908       0.085133       0.001904       0.001904 
    ## 
    ## 2. Quantiles for each variable:
    ## 
    ##  2.5%   25%   50%   75% 97.5% 
    ## 1.211 1.311 1.367 1.426 1.543

We want to plot predictions for different combinations of worm size and
life stage. Let’s make a new dataframe with the combinations of fixed
effects at which we would like to predict recovery rate. Then we re-fit
the model and extract the predictions and CI for plotting.

We see that much of the variation across life stages can be explained by
worm size. This suggests worm size is an important factor driving the
increase in establishment with life cycle steps.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-160-1.png)<!-- -->

Plot is maybe better in separate panels, though the int vs def
distinction is less clear…

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-161-1.png)<!-- -->

We can also make the same plot, but without the int vs def host
distinction.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-165-1.png)<!-- -->

It certainly looks like this pattern is consistent across the wide span
of larval sizes. But some of the trend could be driven by detection
bias. For example, small worms are harder to find and count. Thus, let’s
check if recovery still increases with worm size when we restrict the
data to the largest third of worm stages.

After restricting the data to large worm stages that are unlikely to be
overlooked, we still see a positive relationship between worm size and
recovery in a model with just random slopes.

    ## 
    ##  Iterations = 1001:40981
    ##  Thinning interval  = 20
    ##  Sample size  = 2000 
    ## 
    ##  DIC: 53308.72 
    ## 
    ##  G-structure:  ~us(1 + log_dpi):study_rs
    ## 
    ##                                  post.mean l-95% CI u-95% CI eff.samp
    ## (Intercept):(Intercept).study_rs    4.5697   0.6637  9.51029     1615
    ## log_dpi:(Intercept).study_rs       -1.0296  -2.3086 -0.07871     1754
    ## (Intercept):log_dpi.study_rs       -1.0296  -2.3086 -0.07871     1754
    ## log_dpi:log_dpi.study_rs            0.4556   0.1491  0.82494     2000
    ## 
    ##  R-structure:  ~units
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## units     1.105   0.8537    1.345     1802
    ## 
    ##  Location effects: cbind(succeeded, failed) ~ log_dpi + log_ws 
    ## 
    ##             post.mean l-95% CI u-95% CI eff.samp pMCMC   
    ## (Intercept)   0.70279 -0.11774  1.68783     2000 0.101   
    ## log_dpi      -0.18968 -0.46999  0.05592     2000 0.146   
    ## log_ws        0.17877  0.04563  0.31399     2000 0.009 **
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

### Hypothesis 4: recovery rates depend on host mass

And what about hosts? Since hosts get larger as life cycles progress,
this could drive the increase in recovery rates with “step”. And might
host mass actually explain recovery better than parasite size? Or do
they interact, with comparably sized worms having a better chance at
infecting a large or small host?

When we added host mass to a model without parasite size, but with stage
in the cycle, recovery rates decreased with host mass. This is not what
we would expect if changes in host mass drove the pattern across life
cycles (i.e. recovery rates were higher in latter stages where hosts
were bigger), but rather suggests that host mass might explain variation
within stages.

    ## 
    ##  Iterations = 1001:40981
    ##  Thinning interval  = 20
    ##  Sample size  = 2000 
    ## 
    ##  DIC: 5535915 
    ## 
    ##  G-structure:  ~us(1 + log_dpi):study_rs
    ## 
    ##                                  post.mean l-95% CI u-95% CI eff.samp
    ## (Intercept):(Intercept).study_rs    7.3378   4.1390  10.8403     1739
    ## log_dpi:(Intercept).study_rs       -1.4550  -2.3415  -0.6958     1795
    ## (Intercept):log_dpi.study_rs       -1.4550  -2.3415  -0.6958     1795
    ## log_dpi:log_dpi.study_rs            0.4975   0.2595   0.7273     1840
    ## 
    ##  R-structure:  ~units
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## units     1.769    1.613     1.94     1578
    ## 
    ##  Location effects: cbind(succeeded, failed) ~ log_dpi + nh_fac * to_int_def + log_hm 
    ## 
    ##                              post.mean l-95% CI u-95% CI eff.samp  pMCMC    
    ## (Intercept)                   -0.52310 -1.27384  0.25156     2000  0.177    
    ## log_dpi                       -0.21776 -0.38868 -0.03894     2000  0.023 *  
    ## nh_fac2                       -0.11418 -1.67789  1.34421     2000  0.878    
    ## nh_fac3                        2.04471  0.36541  3.91338     2000  0.026 *  
    ## to_int_defDefinitive          -1.14119 -1.74358 -0.51287     1870 <5e-04 ***
    ## log_hm                        -0.09746 -0.16341 -0.03411     2000  0.004 ** 
    ## nh_fac2:to_int_defDefinitive   2.23075  0.64996  3.82592     1984  0.012 *  
    ## nh_fac3:to_int_defDefinitive   0.69551 -1.44557  2.68132     2000  0.487    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Host mass alone explained about the same amount of variation as parasite
size did, after accounting for stage. But when we consider host mass in
the absence of stage data, it explains much less variance. This suggests
recovery may vary with host mass within stages.

<div class="kable-table">

| model                          | r2m                   | r2c                   |
| :----------------------------- | :-------------------- | :-------------------- |
| base, stage, and int vs def    | 0.102 \[0.061-0.156\] | 0.477 \[0.414-0.541\] |
| \+ host mass without worm size | 0.124 \[0.079-0.18\]  | 0.502 \[0.432-0.572\] |
| \+ worm size without host mass | 0.123 \[0.083-0.168\] | 0.457 \[0.397-0.519\] |
| base + host mass               | 0.044 \[0.018-0.081\] | 0.477 \[0.409-0.551\] |

</div>

Here’s how adding host mass impacts DIC.

    ## delta DIC, after adding host mass to stage model: 0.52 (higher is better)

Let’s look at the host mass parameter, first without life stage in the
model and then with it. Here is the posterior distribution for the odds
ratio without stage.

    ## 
    ## Iterations = 1001:40981
    ## Thinning interval = 20 
    ## Number of chains = 1 
    ## Sample size per chain = 2000 
    ## 
    ## 1. Empirical mean and standard deviation for each variable,
    ##    plus standard error of the mean:
    ## 
    ##           Mean             SD       Naive SE Time-series SE 
    ##      0.8807918      0.0264778      0.0005921      0.0006603 
    ## 
    ## 2. Quantiles for each variable:
    ## 
    ##   2.5%    25%    50%    75%  97.5% 
    ## 0.8286 0.8635 0.8817 0.8992 0.9314

It suggests the odds decrease 12% for a 1 unit change in host mass. Here
is the predicted percent decrease in the odds of infection with a 10%
increase in host mass…

    ## 
    ## Iterations = 1001:40981
    ## Thinning interval = 20 
    ## Number of chains = 1 
    ## Sample size per chain = 2000 
    ## 
    ## 1. Empirical mean and standard deviation for each variable,
    ##    plus standard error of the mean:
    ## 
    ##           Mean             SD       Naive SE Time-series SE 
    ##      1.206e-02      2.836e-03      6.341e-05      7.070e-05 
    ## 
    ## 2. Quantiles for each variable:
    ## 
    ##    2.5%     25%     50%     75%   97.5% 
    ## 0.00675 0.01008 0.01193 0.01389 0.01776

…and for a doubling of host mass.

    ## 
    ## Iterations = 1001:40981
    ## Thinning interval = 20 
    ## Number of chains = 1 
    ## Sample size per chain = 2000 
    ## 
    ## 1. Empirical mean and standard deviation for each variable,
    ##    plus standard error of the mean:
    ## 
    ##           Mean             SD       Naive SE Time-series SE 
    ##      0.0843125      0.0190910      0.0004269      0.0004761 
    ## 
    ## 2. Quantiles for each variable:
    ## 
    ##    2.5%     25%     50%     75%   97.5% 
    ## 0.04806 0.07100 0.08358 0.09675 0.12219

Surprisingly, the odds ratio for host mass get closer to 1 (smaller
effect) when we control for life stage.

    ## 
    ## Iterations = 1001:40981
    ## Thinning interval = 20 
    ## Number of chains = 1 
    ## Sample size per chain = 2000 
    ## 
    ## 1. Empirical mean and standard deviation for each variable,
    ##    plus standard error of the mean:
    ## 
    ##           Mean             SD       Naive SE Time-series SE 
    ##      0.9076315      0.0298260      0.0006669      0.0006669 
    ## 
    ## 2. Quantiles for each variable:
    ## 
    ##   2.5%    25%    50%    75%  97.5% 
    ## 0.8492 0.8874 0.9072 0.9279 0.9664

But is the effect of host mass independent of parasite size? And is it
consistent across stages? Let’s add host mass to models already
including parasite size and check its interaction with stage.

Adding a host mass main effect to a model with a worm size x stage
interaction is not much of an improvement.

    ## delta DIC, after adding host mass to model with worm size: -2.58 (higher is better)

But there might be an interaction between host mass and life stage.

    ## delta DIC, after adding host mass x stage interaction: 8.12 (higher is better)

The R<sup>2</sup> table suggests that allowing host mass to interact
with stage explains a few percentage points of the variation, and this
does not clearly come at the expense of worm size since overall fit
increased.

<div class="kable-table">

| model                            | r2m                   | r2c                   |
| :------------------------------- | :-------------------- | :-------------------- |
| base, stage x worm size          | 0.135 \[0.093-0.181\] | 0.471 \[0.409-0.536\] |
| \+ host mass                     | 0.174 \[0.123-0.225\] | 0.504 \[0.442-0.568\] |
| \+ host mass x stage interaction | 0.185 \[0.134-0.242\] | 0.514 \[0.45-0.578\]  |

</div>

The step x host mass interaction terms were generally not significant
but they did vary in sign.

    ## 
    ##  Iterations = 1001:40981
    ##  Thinning interval  = 20
    ##  Sample size  = 2000 
    ## 
    ##  DIC: 5535910 
    ## 
    ##  G-structure:  ~us(1 + log_dpi):study_rs
    ## 
    ##                                  post.mean l-95% CI u-95% CI eff.samp
    ## (Intercept):(Intercept).study_rs    7.6819   4.3119  11.5236     1632
    ## log_dpi:(Intercept).study_rs       -1.6239  -2.5155  -0.7579     1826
    ## (Intercept):log_dpi.study_rs       -1.6239  -2.5155  -0.7579     1826
    ## log_dpi:log_dpi.study_rs            0.5191   0.2817   0.7534     1862
    ## 
    ##  R-structure:  ~units
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## units     1.741    1.585      1.9     1738
    ## 
    ##  Location effects: cbind(succeeded, failed) ~ log_dpi + nh_fac * to_int_def * log_ws + nh_fac * to_int_def * log_hm 
    ## 
    ##                                     post.mean  l-95% CI  u-95% CI eff.samp
    ## (Intercept)                          12.27789   6.63089  17.96405     2000
    ## log_dpi                              -0.20141  -0.37024  -0.01372     2000
    ## nh_fac2                             -11.83721 -19.31891  -4.44793     2000
    ## nh_fac3                             -13.06180 -19.93885  -6.78928     2000
    ## to_int_defDefinitive                 -5.02687 -13.93365   3.84989     2000
    ## log_ws                                1.38931   0.80932   2.01853     2000
    ## log_hm                               -0.15283  -0.23396  -0.07212     2209
    ## nh_fac2:to_int_defDefinitive          6.99172  -3.08177  17.20533     2000
    ## nh_fac3:to_int_defDefinitive          5.24086  -4.28954  15.56402     1818
    ## nh_fac2:log_ws                       -1.17116  -2.10878  -0.19037     2013
    ## nh_fac3:log_ws                       -0.91684  -1.68152  -0.20742     2000
    ## to_int_defDefinitive:log_ws          -0.41812  -1.30330   0.59618     2000
    ## nh_fac2:log_hm                        0.34025  -0.25019   0.85648     2000
    ## nh_fac3:log_hm                        0.24050  -0.34052   0.84166     2000
    ## to_int_defDefinitive:log_hm           0.02602  -0.17206   0.21454     2161
    ## nh_fac2:to_int_defDefinitive:log_ws   0.44617  -0.64166   1.78494     2000
    ## nh_fac3:to_int_defDefinitive:log_ws   0.48269  -0.59469   1.67857     1700
    ## nh_fac2:to_int_defDefinitive:log_hm  -0.43374  -1.05240   0.14320     2000
    ## nh_fac3:to_int_defDefinitive:log_hm   0.01963  -0.71739   0.65028     2000
    ##                                      pMCMC    
    ## (Intercept)                         <5e-04 ***
    ## log_dpi                              0.035 *  
    ## nh_fac2                              0.003 ** 
    ## nh_fac3                             <5e-04 ***
    ## to_int_defDefinitive                 0.263    
    ## log_ws                              <5e-04 ***
    ## log_hm                              <5e-04 ***
    ## nh_fac2:to_int_defDefinitive         0.190    
    ## nh_fac3:to_int_defDefinitive         0.281    
    ## nh_fac2:log_ws                       0.015 *  
    ## nh_fac3:log_ws                       0.008 ** 
    ## to_int_defDefinitive:log_ws          0.393    
    ## nh_fac2:log_hm                       0.224    
    ## nh_fac3:log_hm                       0.415    
    ## to_int_defDefinitive:log_hm          0.792    
    ## nh_fac2:to_int_defDefinitive:log_ws  0.477    
    ## nh_fac3:to_int_defDefinitive:log_ws  0.377    
    ## nh_fac2:to_int_defDefinitive:log_hm  0.155    
    ## nh_fac3:to_int_defDefinitive:log_hm  0.951    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Let’s plot these host mass effects.

These are the model predictions without accounting for worm size
(i.e. just host mass in the model). There are not clear, consistent
trends.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-184-1.png)<!-- -->

Let’s look at the same plot, but while controlling for worm size - the
predicted values in the next plot are for the average worm size in each
stage. It does not differ much from the previous plot.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-187-1.png)<!-- -->

Better in two panels?

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-188-1.png)<!-- -->

We can make the same plot, but just distinguishing host in cycle, not
int vs def.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-191-1.png)<!-- -->
Finally, we can check whether worm size and host size interact. In other
words, is being big relatively more important when the next host is big
(or small)?

Let’s first check without stage x covariate interactions. That is, we’ll
add the worm size by host mass interaction to a model with just stage.
The interaction term is not significant.

    ## 
    ##  Iterations = 1001:40981
    ##  Thinning interval  = 20
    ##  Sample size  = 2000 
    ## 
    ##  DIC: 5535914 
    ## 
    ##  G-structure:  ~us(1 + log_dpi):study_rs
    ## 
    ##                                  post.mean l-95% CI u-95% CI eff.samp
    ## (Intercept):(Intercept).study_rs    6.8734   3.7851  10.0650     1364
    ## log_dpi:(Intercept).study_rs       -1.4494  -2.3099  -0.7152     1661
    ## (Intercept):log_dpi.study_rs       -1.4494  -2.3099  -0.7152     1661
    ## log_dpi:log_dpi.study_rs            0.4871   0.2652   0.7186     1736
    ## 
    ##  R-structure:  ~units
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## units     1.767    1.593    1.925     1798
    ## 
    ##  Location effects: cbind(succeeded, failed) ~ log_dpi + nh_fac * to_int_def + log_ws * log_hm 
    ## 
    ##                              post.mean l-95% CI u-95% CI eff.samp  pMCMC    
    ## (Intercept)                    2.83103  0.76670  4.81899     1924  0.006 ** 
    ## log_dpi                       -0.21363 -0.38258 -0.05009     2000  0.017 *  
    ## nh_fac2                       -1.15478 -2.75225  0.30024     2000  0.129    
    ## nh_fac3                       -1.72489 -3.96239  0.45317     2000  0.127    
    ## to_int_defDefinitive          -1.13254 -1.72138 -0.46335     2000  0.001 ***
    ## log_ws                         0.36935  0.15231  0.56073     1930 <5e-04 ***
    ## log_hm                        -0.15059 -0.35194  0.04078     2222  0.130    
    ## nh_fac2:to_int_defDefinitive   1.83967  0.21619  3.34204     1944  0.020 *  
    ## nh_fac3:to_int_defDefinitive   1.44048 -0.45136  3.37484     1924  0.141    
    ## log_ws:log_hm                 -0.00563 -0.02649  0.01748     2110  0.618    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

And the R<sup>2</sup> value does not increase much with this
interaction.

<div class="kable-table">

| model                                | r2m                   | r2c                   |
| :----------------------------------- | :-------------------- | :-------------------- |
| base, stage, and int vs def          | 0.102 \[0.061-0.156\] | 0.477 \[0.414-0.541\] |
| \+ host mass (alone, no worm size)   | 0.124 \[0.079-0.18\]  | 0.502 \[0.432-0.572\] |
| \+ worm size (alone, no host mass)   | 0.123 \[0.083-0.168\] | 0.457 \[0.397-0.519\] |
| \+ worm size x host mass interaction | 0.145 \[0.1-0.196\]   | 0.483 \[0.418-0.547\] |

</div>

Likewise, the host mass by worm size interaction was not significant
when added to a model with stage interactions. This is not surprising,
given that we already allow worm and host size effects to vary across
the steps of the life cycle.

    ## 
    ##  Iterations = 1001:40981
    ##  Thinning interval  = 20
    ##  Sample size  = 2000 
    ## 
    ##  DIC: 5535914 
    ## 
    ##  G-structure:  ~us(1 + log_dpi):study_rs
    ## 
    ##                                  post.mean l-95% CI u-95% CI eff.samp
    ## (Intercept):(Intercept).study_rs    7.6867   4.4099  11.7049     1790
    ## log_dpi:(Intercept).study_rs       -1.6217  -2.5606  -0.7745     1819
    ## (Intercept):log_dpi.study_rs       -1.6217  -2.5606  -0.7745     1819
    ## log_dpi:log_dpi.study_rs            0.5203   0.2746   0.7615     1828
    ## 
    ##  R-structure:  ~units
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## units     1.741    1.583    1.896     1596
    ## 
    ##  Location effects: cbind(succeeded, failed) ~ log_dpi + nh_fac * to_int_def * log_ws + nh_fac * to_int_def * log_hm + log_ws * log_hm 
    ## 
    ##                                      post.mean   l-95% CI   u-95% CI eff.samp
    ## (Intercept)                          12.305185   6.958359  18.056210     1609
    ## log_dpi                              -0.199114  -0.381058  -0.015981     2000
    ## nh_fac2                             -11.871194 -19.694553  -4.559730     2000
    ## nh_fac3                             -13.002873 -20.003745  -6.537428     1775
    ## to_int_defDefinitive                 -5.108299 -13.887112   4.423536     2349
    ## log_ws                                1.392646   0.790722   1.972024     1610
    ## log_hm                               -0.135329  -0.540060   0.293417     2009
    ## nh_fac2:to_int_defDefinitive          7.029571  -2.780704  17.107172     2723
    ## nh_fac3:to_int_defDefinitive          5.222900  -4.727546  14.880848     2204
    ## nh_fac2:log_ws                       -1.175718  -2.115408  -0.169253     2000
    ## nh_fac3:log_ws                       -0.937731  -1.690770  -0.138053     1482
    ## to_int_defDefinitive:log_ws          -0.429737  -1.452122   0.565944     2328
    ## nh_fac2:log_hm                        0.345382  -0.206073   0.859336     2000
    ## nh_fac3:log_hm                        0.214787  -0.571775   0.919715     2035
    ## to_int_defDefinitive:log_hm           0.021637  -0.171777   0.200371     2000
    ## log_ws:log_hm                         0.001899  -0.038989   0.050347     2110
    ## nh_fac2:to_int_defDefinitive:log_ws   0.445545  -0.803652   1.598317     2582
    ## nh_fac3:to_int_defDefinitive:log_ws   0.493577  -0.563904   1.623186     2223
    ## nh_fac2:to_int_defDefinitive:log_hm  -0.444751  -1.039645   0.114651     2000
    ## nh_fac3:to_int_defDefinitive:log_hm   0.031883  -0.647462   0.739244     2000
    ##                                      pMCMC    
    ## (Intercept)                         <5e-04 ***
    ## log_dpi                              0.036 *  
    ## nh_fac2                             <5e-04 ***
    ## nh_fac3                             <5e-04 ***
    ## to_int_defDefinitive                 0.274    
    ## log_ws                              <5e-04 ***
    ## log_hm                               0.525    
    ## nh_fac2:to_int_defDefinitive         0.177    
    ## nh_fac3:to_int_defDefinitive         0.314    
    ## nh_fac2:log_ws                       0.022 *  
    ## nh_fac3:log_ws                       0.020 *  
    ## to_int_defDefinitive:log_ws          0.417    
    ## nh_fac2:log_hm                       0.220    
    ## nh_fac3:log_hm                       0.578    
    ## to_int_defDefinitive:log_hm          0.822    
    ## log_ws:log_hm                        0.918    
    ## nh_fac2:to_int_defDefinitive:log_ws  0.503    
    ## nh_fac3:to_int_defDefinitive:log_ws  0.380    
    ## nh_fac2:to_int_defDefinitive:log_hm  0.139    
    ## nh_fac3:to_int_defDefinitive:log_hm  0.925    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

And it explained essentially no additional variation in recovery rates.

<div class="kable-table">

| model                                      | r2m                   | r2c                   |
| :----------------------------------------- | :-------------------- | :-------------------- |
| base, worm size x stage, host mass x stage | 0.185 \[0.134-0.242\] | 0.514 \[0.45-0.578\]  |
| \+ worm size x host mass interaction       | 0.186 \[0.138-0.243\] | 0.516 \[0.452-0.582\] |

</div>

The model is worse, judged by DIC.

    ## delta DIC, after adding host mass x worm size interaction to model with worm size and host mass: -3.42 (higher is better)

In sum, establishment decreases with host mass, though this varies with
life stage.

## Hypothesis 5: recover rate over time depends on step in the cycle

Are worms lost faster from the second host than the first host? To test
this, let’s examine the interaction between time to dissection and step.
I’ll add this interaction to a model without host mass and worm size and
then to a model with those two variables.

Here is how a time x stage interaction impacts DIC.

    ## delta DIC, added dpi x step interaction to model with just their main effects: -2.17 (higher is better)

But the effect is small, as judged by R<sup>2</sup>

<div class="kable-table">

| model                           | r2m                   | r2c                   |
| :------------------------------ | :-------------------- | :-------------------- |
| base, stage                     | 0.102 \[0.061-0.156\] | 0.477 \[0.414-0.541\] |
| \+ time dpi x stage interaction | 0.114 \[0.072-0.166\] | 0.486 \[0.424-0.547\] |

</div>

Here is the change in model DIC when this time x stage interaction is
added to a model with host and parasite size effects.

    ## delta DIC, added dpi x step interaction to model with just their main effects: -2.63 (higher is better)

<div class="kable-table">

| model                                           | r2m                   | r2c                   |
| :---------------------------------------------- | :-------------------- | :-------------------- |
| with worm and host size, and their interactions | 0.185 \[0.134-0.242\] | 0.514 \[0.45-0.578\]  |
| \+ time dpi x stage interaction                 | 0.19 \[0.139-0.246\]  | 0.516 \[0.451-0.588\] |

</div>

And the terms of this time by stage interaction are not significant.

    ## 
    ##  Iterations = 1001:40981
    ##  Thinning interval  = 20
    ##  Sample size  = 2000 
    ## 
    ##  DIC: 5535913 
    ## 
    ##  G-structure:  ~us(1 + log_dpi):study_rs
    ## 
    ##                                  post.mean l-95% CI u-95% CI eff.samp
    ## (Intercept):(Intercept).study_rs    6.8883   3.5009  10.2459     1486
    ## log_dpi:(Intercept).study_rs       -1.4926  -2.2992  -0.6706     1540
    ## (Intercept):log_dpi.study_rs       -1.4926  -2.2992  -0.6706     1540
    ## log_dpi:log_dpi.study_rs            0.5186   0.2757   0.7536     1819
    ## 
    ##  R-structure:  ~units
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## units     1.735    1.574    1.901     1705
    ## 
    ##  Location effects: cbind(succeeded, failed) ~ log_dpi * nh_fac * to_int_def + nh_fac * to_int_def * log_ws + nh_fac * to_int_def * log_hm 
    ## 
    ##                                       post.mean   l-95% CI   u-95% CI eff.samp
    ## (Intercept)                           11.542139   5.879693  17.059405     2000
    ## log_dpi                                0.069462  -0.239533   0.417042     2000
    ## nh_fac2                              -10.895077 -22.775609   1.570373     2543
    ## nh_fac3                              -11.604892 -20.428883  -2.560820     1845
    ## to_int_defDefinitive                  -3.484030 -11.882391   5.376578     2000
    ## log_ws                                 1.393398   0.824777   2.029008     2000
    ## log_hm                                -0.155181  -0.234167  -0.069230     1653
    ## log_dpi:nh_fac2                       -0.315336  -2.054966   1.547340     2000
    ## log_dpi:nh_fac3                       -0.369509  -1.780262   0.953471     1818
    ## log_dpi:to_int_defDefinitive          -0.589154  -1.011997  -0.226724     2000
    ## nh_fac2:to_int_defDefinitive           5.383268  -8.462701  18.744430     2000
    ## nh_fac3:to_int_defDefinitive           1.501553 -10.169024  12.675731     1857
    ## nh_fac2:log_ws                        -1.168452  -2.299530   0.013546     2512
    ## nh_fac3:log_ws                        -0.971441  -1.711709  -0.223651     2000
    ## to_int_defDefinitive:log_ws           -0.410060  -1.328627   0.560398     2000
    ## nh_fac2:log_hm                         0.351975  -0.205988   0.904237     1814
    ## nh_fac3:log_hm                         0.228275  -0.363599   0.835967     2000
    ## to_int_defDefinitive:log_hm            0.056733  -0.160089   0.239122     2000
    ## log_dpi:nh_fac2:to_int_defDefinitive   0.597796  -1.279688   2.371236     2000
    ## log_dpi:nh_fac3:to_int_defDefinitive   1.201530  -0.235483   2.736447     1856
    ## nh_fac2:to_int_defDefinitive:log_ws    0.428187  -0.947710   1.750973     2000
    ## nh_fac3:to_int_defDefinitive:log_ws    0.636965  -0.396886   1.807764     1624
    ## nh_fac2:to_int_defDefinitive:log_hm   -0.479001  -1.101489   0.149586     2000
    ## nh_fac3:to_int_defDefinitive:log_hm   -0.007254  -0.751632   0.658327     2000
    ##                                       pMCMC    
    ## (Intercept)                           0.002 ** 
    ## log_dpi                               0.690    
    ## nh_fac2                               0.087 .  
    ## nh_fac3                               0.015 *  
    ## to_int_defDefinitive                  0.448    
    ## log_ws                               <5e-04 ***
    ## log_hm                               <5e-04 ***
    ## log_dpi:nh_fac2                       0.732    
    ## log_dpi:nh_fac3                       0.583    
    ## log_dpi:to_int_defDefinitive          0.002 ** 
    ## nh_fac2:to_int_defDefinitive          0.436    
    ## nh_fac3:to_int_defDefinitive          0.784    
    ## nh_fac2:log_ws                        0.040 *  
    ## nh_fac3:log_ws                        0.010 ** 
    ## to_int_defDefinitive:log_ws           0.392    
    ## nh_fac2:log_hm                        0.218    
    ## nh_fac3:log_hm                        0.470    
    ## to_int_defDefinitive:log_hm           0.585    
    ## log_dpi:nh_fac2:to_int_defDefinitive  0.512    
    ## log_dpi:nh_fac3:to_int_defDefinitive  0.122    
    ## nh_fac2:to_int_defDefinitive:log_ws   0.537    
    ## nh_fac3:to_int_defDefinitive:log_ws   0.253    
    ## nh_fac2:to_int_defDefinitive:log_hm   0.133    
    ## nh_fac3:to_int_defDefinitive:log_hm   0.974    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Let’s plot the relationship between time and recovery predicted by the
model.

There are not consistent changes in recovery over time across stages.
The predicted relationships are very uncertain (large CIs), which makes
sense, since different studies yielded different recovery x time
relationships (i.e. the random slopes).

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-207-1.png)<!-- -->
\#\#\# Double check phylogeny again

Finally, let’s check phylogeny again, now that we have a model full of
predictors.

Adding phylogeny to the model is not an improvement by DIC…

    ## delta DIC, saturated model with and without phylogeny: NaN (higher is better)

…or by R<sup>2</sup>.

<div class="kable-table">

| model                                           | r2m                   | r2c                   |
| :---------------------------------------------- | :-------------------- | :-------------------- |
| with worm and host size, and their interactions | 0.19 \[0.139-0.246\]  | 0.516 \[0.451-0.588\] |
| \+ time dpi x stage interaction                 | 0.054 \[0.026-0.107\] | 0.668 \[0.596-0.734\] |

</div>

The lower bound is also near zero when we look at the trace.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-212-1.png)<!-- -->

Thus, phylogeny does not seem important.

# Conclusions

We examined the determinants of establishment rate in worms. It can vary
a lot from one study to the next as well as within studies due to
e.g. dose or time differences (Fig. 1). Establishment rate tends to
increase with life cycle progression, especially when the next host is
the definitive host (Fig. 2). This effect is partly driven by worm size,
with larger worms having higher establishment rates (Fig. 3).
Establishment decreases in big hosts, but the effect is variable across
the life cycle (Fig. 4).

We can quantitatively summarize the results by making an R<sup>2</sup>
table. The table suggests that recovery rates from the same study are
quite similar, especially if we account for time-dependence within
studies (i.e the random-slopes model accounted for about 10% additional
variation). However, the variance explained solely by study (conditional
- marginal R<sup>2</sup>) tends to go down as we add predictors,
indicating that differences from one study to the next can be partly
explained by things like life stage and worm size. Among the predictors,
life stage, worm size, and host mass all had clear effects.

<div class="kable-table">

| model                                            | r2m                   | r2c                   |
| :----------------------------------------------- | :-------------------- | :-------------------- |
| base (time post infection & study random effect) | 0.018 \[0.01-0.03\]   | 0.333 \[0.277-0.399\] |
| \+ time dpi x study (random slopes)              | 0.01 \[0.001-0.031\]  | 0.421 \[0.36-0.486\]  |
| \+ next host in cycle                            | 0.066 \[0.029-0.113\] | 0.419 \[0.362-0.484\] |
| \+ intermediate or definitive host               | 0.088 \[0.05-0.134\]  | 0.464 \[0.402-0.529\] |
| \+ next host x int vs def                        | 0.102 \[0.061-0.156\] | 0.477 \[0.414-0.541\] |
| \+ worm size                                     | 0.123 \[0.083-0.168\] | 0.457 \[0.397-0.519\] |
| \+ worm size x stage                             | 0.135 \[0.093-0.181\] | 0.471 \[0.409-0.536\] |
| \+ host mass                                     | 0.174 \[0.123-0.225\] | 0.504 \[0.442-0.568\] |
| \+ host mass x stage                             | 0.185 \[0.134-0.242\] | 0.514 \[0.45-0.578\]  |
| \+ time dpi x stage                              | 0.19 \[0.139-0.246\]  | 0.516 \[0.451-0.588\] |

</div>

To double check my R<sup>2</sup> calculations, I refit all the models in
the table above with `glmer` and then calculated R<sup>2</sup>. This is
simpler because, unlike with `MCMCglmm` I am not putting CIs on the
variance components. With `glmer` models it is also simple to perform
likelihood ratio tests and have p-values for model comparisons.

The R<sup>2</sup> values from `glmer` models look very comparable for
marginal R<sup>2</sup>, but conditional R<sup>2</sup> tends to be a bit
lower (6% lower in the most complex model). The main reason seems to be
that the random effects VC tends to be slightly high in `MCMCglmm`. I’ve
observed this in previous studies too and I cannot say why exactly
`MCMCglmm` yields slightly higher VCs. I played with priors, but unless
the priors are very strong, this has very little impact on the posterior
parameter estimates. But overall, the similarity is reassuring.

<div class="kable-table">

| step                                             | df\_used | LRT\_pval | marg\_r2 | cond\_r2 | study\_var\_explained |
| :----------------------------------------------- | -------: | --------: | -------: | -------: | --------------------: |
| base (time post infection & study random effect) |       NA |        NA |    0.017 |    0.330 |                 0.313 |
| \+ time dpi x study (random slopes)              |        0 |   0.00000 |    0.010 |    0.401 |                 0.391 |
| \+ next host in cycle                            |        2 |   0.00001 |    0.067 |    0.396 |                 0.329 |
| \+ intermediate or definitive host               |        1 |   0.00000 |    0.085 |    0.440 |                 0.355 |
| \+ next host x int vs def                        |        2 |   0.01680 |    0.099 |    0.451 |                 0.352 |
| \+ worm size                                     |        1 |   0.00000 |    0.120 |    0.430 |                 0.310 |
| \+ worm size x stage                             |        5 |   0.06205 |    0.130 |    0.435 |                 0.305 |
| \+ host mass                                     |        1 |   0.00002 |    0.165 |    0.466 |                 0.301 |
| \+ host mass x stage                             |        5 |   0.38075 |    0.174 |    0.471 |                 0.297 |
| \+ time dpi x stage                              |        5 |   0.05424 |    0.175 |    0.461 |                 0.286 |

</div>

The likelihood ratio tests are also included in the table, and they
suggest that the model improves by adding random slopes, adding life
stage (both next host and the int vs def distinction), adding worm size,
and adding host mass, but not the size x stage interactions.

Let’s also check these terms individually. How much variation do they
explain on their own? Marginal R<sup>2</sup> represents the variance
explained by each term alone. They tend to be consistently positive,
implying each term alone can explain some variance in recovery. The
combinations of worm size and stage and host size and stage explain the
most variation, so the effects of size and stage are not entirely
redundant.

<div class="kable-table">

| model                              | r2m                   | r2c                   |
| :--------------------------------- | :-------------------- | :-------------------- |
| \+ next host in cycle              | 0.066 \[0.029-0.113\] | 0.419 \[0.362-0.484\] |
| \+ intermediate or definitive host | 0.04 \[0.017-0.071\]  | 0.479 \[0.41-0.549\]  |
| \+ next host x int vs def          | 0.102 \[0.061-0.156\] | 0.477 \[0.414-0.541\] |
| \+ worm size                       | 0.084 \[0.046-0.127\] | 0.401 \[0.346-0.458\] |
| \+ worm size x stage               | 0.135 \[0.093-0.181\] | 0.471 \[0.409-0.536\] |
| \+ host mass                       | 0.044 \[0.018-0.081\] | 0.477 \[0.409-0.551\] |
| \+ host mass x stage               | 0.137 \[0.087-0.196\] | 0.513 \[0.445-0.583\] |

</div>

In addition to looking at how “explanatory” terms are alone, we can also
gauge how important they are in the full, final model. Here are the
significant parameters in the final model: stage, host mass, and worm
size by stage.

<div class="kable-table">

| param            |          lwr |          fit |         upr | sig |
| :--------------- | -----------: | -----------: | ----------: | :-- |
| (Intercept)      |    6.6532359 |   12.2898743 |  17.9862207 | sig |
| log\_dpi         |  \-0.3776066 |  \-0.2026914 | \-0.0165445 | sig |
| nh\_fac2         | \-19.3119691 | \-11.7583324 | \-4.4163974 | sig |
| nh\_fac3         | \-19.7195658 | \-13.0890784 | \-6.2937401 | sig |
| log\_ws          |    0.7774543 |    1.3857158 |   1.9952076 | sig |
| log\_hm          |  \-0.2336520 |  \-0.1530771 | \-0.0717097 | sig |
| nh\_fac2:log\_ws |  \-2.1048187 |  \-1.1636208 | \-0.1848662 | sig |
| nh\_fac3:log\_ws |  \-1.6769042 |  \-0.9115978 | \-0.1911521 | sig |

</div>

As we now know what kinds of experiments yield high recovery rates (late
stage in life cycle, large larvae, small host for given stage), let’s
look at whether doses follow these trends. That is, do researchers use
higher doses when they expect lower recoveries?

# Dose as response variable

Here is the distribution of doses - it varies a lot - but it is
reasonably normal on a log scale.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-223-1.png)<!-- -->

Thus, let’s fit the same series of models as for recovery rate, but with
dose as response variable. We can probably also drop the time of
dissection from the model too - it is not obvious why one should use
higher doses when dissection dates are earlier/later. Let’s check
whether dose and dissection time covary. There is a slight tendency to
use higher doses when dissecting later.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-224-1.png)<!-- -->

And here is a likelihood ratio test - adding dissection time weakly
predicts dose.

<div class="kable-table">

|       | npar |      AIC |      BIC |     logLik | deviance |    Chisq | Df | Pr(\>Chisq) |
| :---- | ---: | -------: | -------: | ---------: | -------: | -------: | -: | ----------: |
| lmd0  |    8 | 3785.891 | 3827.811 | \-1884.946 | 3769.891 |       NA | NA |          NA |
| lmd00 |    9 | 3784.536 | 3831.695 | \-1883.268 | 3766.536 | 3.355356 |  1 |   0.0669867 |

</div>

within studies, are higher doses used with later dissection points? Here
are most of the studies with multiple doses and dissection times.
Usually the dose does not vary much with the time of dissection,
suggesting the same random slopes structure is not needed for ‘dose’
models.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-226-1.png)<!-- -->

So let’s model log dose using standard linear mixed models (not random
slopes). As for recovery rates, we add life stage, worm size, and host
mass.

Here’s the R<sup>2</sup> table. Studies vary a lot in the dose used - by
itself the study random effect explains over 90% of the variation in
dose. This suggests that a random effects model might be overkill; given
that there is little variation within studies, we could probably just
model the unique combinations of dose and study. On the other hand, the
marginal R<sup>2</sup> goes up to over 50% in the most complex model,
indicating that differences among studies can be explained by
differences in life stage, worm size, and host mass.

<div class="kable-table">

| step                               | df\_used | LRT\_pval | marg\_r2 | cond\_r2 | study\_var\_explained |
| :--------------------------------- | -------: | --------: | -------: | -------: | --------------------: |
| just study random effect           |       NA |        NA |    0.000 |    0.919 |                 0.919 |
| \+ next host in cycle              |        2 |   0.00000 |    0.284 |    0.918 |                 0.634 |
| \+ intermediate or definitive host |        1 |   0.00000 |    0.321 |    0.911 |                 0.590 |
| \+ next host x int vs def          |        2 |   0.10349 |    0.315 |    0.909 |                 0.594 |
| \+ worm size                       |        1 |   0.60403 |    0.313 |    0.907 |                 0.594 |
| \+ worm size x stage               |        5 |   0.00020 |    0.393 |    0.906 |                 0.513 |
| \+ host mass                       |        1 |   0.00010 |    0.459 |    0.897 |                 0.438 |
| \+ host mass x stage               |        5 |   0.36283 |    0.466 |    0.898 |                 0.432 |

</div>

Let’s plot some of these effects. To get CIs, we’ll fit the model again
with `MCMCglmm`.

Here is the model summary. Notably, the distinction between intermediate
and definitive hosts is not important.

    ## 
    ##  Iterations = 1001:50951
    ##  Thinning interval  = 50
    ##  Sample size  = 1000 
    ## 
    ##  DIC: 3416.063 
    ## 
    ##  G-structure:  ~study_rs
    ## 
    ##          post.mean l-95% CI u-95% CI eff.samp
    ## study_rs     2.658    1.949    3.482     1080
    ## 
    ##  R-structure:  ~units
    ## 
    ##       post.mean l-95% CI u-95% CI eff.samp
    ## units    0.6125   0.5641   0.6609     1000
    ## 
    ##  Location effects: log_dose ~ nh_fac * to_int_def * log_ws + nh_fac * to_int_def * log_hm 
    ## 
    ##                                      post.mean   l-95% CI   u-95% CI eff.samp
    ## (Intercept)                          12.740057   8.369629  17.374599   1000.0
    ## nh_fac2                             -14.321068 -20.065432  -8.029200   1065.2
    ## nh_fac3                             -10.549102 -16.400360  -6.069690    880.9
    ## to_int_defDefinitive                 -2.077307  -8.993402   5.010804   1000.0
    ## log_ws                                0.779738   0.236132   1.241679   1000.0
    ## log_hm                                0.083650   0.027058   0.139688   1000.0
    ## nh_fac2:to_int_defDefinitive          5.755378  -2.685869  13.681703    747.4
    ## nh_fac3:to_int_defDefinitive          1.435905  -6.186517   8.973628   1000.0
    ## nh_fac2:log_ws                       -1.282851  -2.082529  -0.516482    796.2
    ## nh_fac3:log_ws                       -0.981104  -1.566318  -0.388509    898.7
    ## to_int_defDefinitive:log_ws          -0.377937  -1.205389   0.342309   1000.0
    ## nh_fac2:log_hm                       -0.038538  -0.371585   0.289161   1032.7
    ## nh_fac3:log_hm                        0.003671  -0.390736   0.443394    777.2
    ## to_int_defDefinitive:log_hm          -0.072681  -0.198853   0.074759   1000.0
    ## nh_fac2:to_int_defDefinitive:log_ws   0.772190  -0.145298   1.843714    643.6
    ## nh_fac3:to_int_defDefinitive:log_ws   0.169568  -0.677457   1.062156    810.1
    ## nh_fac2:to_int_defDefinitive:log_hm   0.218064  -0.177268   0.561326   1120.6
    ## nh_fac3:to_int_defDefinitive:log_hm   0.087668  -0.383280   0.576981    659.8
    ##                                      pMCMC    
    ## (Intercept)                         <0.001 ***
    ## nh_fac2                             <0.001 ***
    ## nh_fac3                             <0.001 ***
    ## to_int_defDefinitive                 0.498    
    ## log_ws                               0.002 ** 
    ## log_hm                               0.008 ** 
    ## nh_fac2:to_int_defDefinitive         0.130    
    ## nh_fac3:to_int_defDefinitive         0.692    
    ## nh_fac2:log_ws                       0.002 ** 
    ## nh_fac3:log_ws                       0.004 ** 
    ## to_int_defDefinitive:log_ws          0.294    
    ## nh_fac2:log_hm                       0.818    
    ## nh_fac3:log_hm                       0.998    
    ## to_int_defDefinitive:log_hm          0.286    
    ## nh_fac2:to_int_defDefinitive:log_ws  0.120    
    ## nh_fac3:to_int_defDefinitive:log_ws  0.682    
    ## nh_fac2:to_int_defDefinitive:log_hm  0.262    
    ## nh_fac3:to_int_defDefinitive:log_hm  0.742    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

First, here is the change in dose used with life cycle stage. The size
of the points correspond to the number of exposures at that dose for the
study. Lower doses are used with later life stages. The difference
between intermediate and definitive hosts is caused by differences in
host mass.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-236-1.png)<!-- -->

Mean dose decreases with life cycle stage. The CIs seem a little
overconfident, though. Let’s pool all exposures from the same study at
the same dose and refit the model.

The study effect (red) obviously goes down after pooling, as there is
less replication within studies.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-239-1.png)<!-- -->

The means and CIs from this model are not much different, suggesting
pooling within studies does not have a big impact.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-241-1.png)<!-- -->

What about the two covariates parasite size and host size? Does dose
vary with them?

Dose decreases with worm size, mainly across life stages, but also
within them.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-242-1.png)<!-- -->

Dose increases with host mass within and across stages. The different
doses used for intermediate and definitive hosts also looks like it can
be explained by host mass.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-243-1.png)<!-- -->

Since the model suggested that differences between intermediate and
definitive hosts were not important, let’s re-fit the dose model without
this term.

Including the int vs def distinction (red) only marginally lowers the
model deviance.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-247-1.png)<!-- -->

And the DIC is better without this term in the model.

    ## delta DIC, without vs with int/def distinction: 17.35 (higher is better)

Here are the model predictions, after removing the intermediate vs
definitive host distinction.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-249-1.png)<!-- -->

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-250-1.png)<!-- -->

Finally, let’s refit the model, but with just the predicted recovery
rates from the mixed models. The predicted recovery rates combine info
on life stage, worm size, and host mass.

![](ER_analysis_undercounts_rem_files/figure-gfm/unnamed-chunk-256-1.png)<!-- -->

Thus, experimenters usually use higher doses in situations where lower
infection rates are expected.
