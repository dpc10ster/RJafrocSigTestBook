# Sources of AUC variability {#sources-of-variability}







## TBA How much finished {#sources-of-variability-how-much-finished}
60%



## Introduction {#sources-of-variabilityIntro}
In previous chapters the area AUC under the ROC plot was introduced as the preferred way of summarizing performance in the ROC task, as compared to a pair of sensitivity and specificity values. It can be estimated either non-parametrically, as in `TempComment Chapter \@ref(empirical-auc)`, or parametrically, as in Chapter `TempComment \@ref(binormal-model)`, and even better ways of estimating it are described in TBA Chapter 18 and Chapter 20. 

Irrespective of how it is estimated AUC is a realization of a random variable, and as such, it is subject to sampling variability. Any measurement based on a finite number of samples from a parent population is subject to sampling variability. This is because no finite sample is unique: someone else conducting a similar study would, in general, obtain a different sample. [Case-sampling variability is estimated by the binormal model in the previous chapter. It is related to the sharpness of the peak of the likelihood function, TBA §6.4.4. The sharper that the peak, the smaller the case sampling variability. This chapter focuses on general sources of variability affecting AUC, regardless of how it is estimated, and other (i.e., not binormal model based) ways of estimating it.]

Here is an outline of this chapter. The starting point is the identification of different sources of variability affecting AUC estimates. Considered next is dependence of AUC on the case-set index $\{c\}$, $c = 1,2,...,C$. Considered next is estimating case-sampling variability of the empirical estimate of AUC by an analytic method. This is followed by descriptions of two resampling-based methods, namely the bootstrap and the jackknife, both of which have wide applicability (i.e., they are not restricted to ROC analysis). The methods are demonstrated using `R` code and the implementation of a calibrated simulator is shown and used to demonstrate their validity, i.e., showing that the different methods of estimating variability agree. The dependence of AUC on reader expertise and modality is considered. An important source of variability, namely the radiologist's choice of internal sensory thresholds, is described. A cautionary comment is made regarding indiscriminate usage of empirical AUC as a measure of performance.  

TBA Online Appendix 7.A describes coding of the bootstrap method; Online Appendix 7.B is the corresponding implementation of the jackknife method. Online Appendix 7.C describes implementation of the calibrated simulator for single-modality single-reader ROC datasets. Online Appendix 7.D describes the code that allows comparison of the different methods of estimating case-sampling variability. 
 

## Three sources of variability {#sources-of-variability3sources}
Statistics deals with variability. Understanding sources of variability affecting AUC is critical to an appreciation of ROC analysis. Three sources of variability are identified in [@RN412]: case sampling, between-reader and within-reader variability.

1. Consider a single reader interpreting different case samples. Case-sampling variability arises from the finite number of cases comprising the dataset, compared to the potentially very large population of cases. [If one could sample every case there exists and have them interpreted by the same reader, there would be no case-sampling variability and the poor reader's AUC values (from repeated interpretations of the entire population) would reflect only within reader variability, see #3 below.] Each case-set $\{c\}$, consisting of $K_1$ non-diseased and $K_2$ diseased cases interpreted by the reader, yields an AUC value. The notation $\{c\}$ means different *case sets*. Thus $\{c\}$ = $\{1\}$, $\{2\}$, etc., denote different case sets, each consisting of $K_1$ non-diseased and $K_2$ diseased cases.

There is much "data compression" in going from individual case ratings to AUC. For a single reader and given case-set $\{c\}$, the ratings can be converted to an $A_{z\{c\}}$  estimate, TBA Eqn. (6.49). The notation shows explicitly the dependence of the measure on the case-set $\{c\}$. One can conceptualize the distribution of $A_{z\{c\}}$'s over different case-sets, each of the same size $K_1+K_2$, as a normal distribution, i.e., 

\begin{equation} 
A_{z\{c\}}\sim N(A_{z\{\bullet\}},\sigma_{\text{cs+wr}}^2)
(\#eq:sources-of-variability-cs-wr)
\end{equation}

The dot notation $\{\bullet\}$ denotes an average over all case sets. Thus, $A_{z\{\bullet\}}$ is an estimate of the case-sampling mean of $A_z$ for a single fixed reader and $\sigma_{\text{cs+wr}}^2$ is the *case sampling plus within-reader* variance. The reason for adding the within-reader variance is explained in #3 below. The concept is that a specified reader interpreting different case-sets effectively samples different parts of the population of cases, resulting in variability in measured $A_z$. Sometimes easier cases are sampled, and sometimes more difficult ones. This source of variability is expected to decrease with increasing case-set size, i.e., increasing $K_1+K_2$, which is the reason for seeking large numbers of cases in clinical trials. Case-sampling and within-reader variability also decreases as the cases become more homogenous. An example of a more homogenous case sample would be cases originating from a small geographical region with, for example, limited ethnic variability. This is the reason for seeking multi-institutional clinical trials, because they tend to sample more of the population than patients seen at a single institution.

2. Consider different readers interpreting a fixed case sample. Between-reader variability arises from the finite number of readers compared to the population of readers; the population of readers could be all board certified radiologists interpreting screening mammograms in the US. This time one envisages different readers interpreting a fixed case set $\{1\}$. The different reader's $A_{z;j}$ values ($j$ is the reader index, $j = 1, 2, ..., J$, where $J$ is the total number of readers in the dataset) are distributed:

\begin{equation} 
A_{z;j}\sim N(A_{z;\bullet},\sigma_{\text{br+wr}}^2)
(\#eq:sources-of-variability-br-wr)
\end{equation}

where $A_{z;\bullet}$ is an estimate of the reader population AUC mean (the bullet symbol replacing the reader index averages over a set of readers) for the fixed case-set $\{1\}$ and $\sigma_{\text{br+wr}}^2$ is the *between-reader plus within-reader* variance. The reason for adding the within-reader variance is explained in #3 below. The concept is that different groups of $J$ readers interpret the same case set $\{1\}$, thereby sampling different parts of the reader distribution, causing fluctuations in the measured  $A_{z;j}$ of the readers. Sometimes better readers are sampled and sometimes not so good ones are sampled. This time there is no "data compression" – each reader in the sample has an associated $A_{z;j}$. However, variability of the average $A_{z;\bullet}$ over the $J$ readers is expected to decrease with increasing $J$. This is the reason for seeking large reader-samples.

3. Consider a fixed reader, e.g., $j = 1$, interpreting a fixed case-sample $\{1\}$. Within-reader variability is due to variability of the ratings for the same case: the same reader interpreting the same case on different occasions will give different ratings to it, causing fluctuations in the measured AUC. This assumes that memory effects are minimized, for example, by sufficient time between successive interpretations as otherwise, if a case is shown twice in succession, the reader would give it the same rating each time. Since this is an intrinsic source of variability (analogous to the internal noise of a voltmeter) affecting each reader's interpretations, it cannot be separated from case sampling variability, i.e., it cannot be "turned off". The last sentence needs further explanation. A measurement of case-sampling variability requires a reader, and the reader comes with an intrinsic source of variability that gets added to the case-sampling variance, so what is measured is the sum of case sampling and within-reader variances, denoted $\sigma_{\text{cs+wr}}^2$. Likewise, a measurement of between-reader variability requires a fixed case-set interpreted by different readers, each of whom comes with an intrinsic source of variability that gets added to the between-reader variance, yielding $\sigma_{\text{br+wr}}^2$. To emphasize this point, an estimate of case-sampling variability *always* includes within reader variability. Likewise, an estimate of between-reader variability *always* includes within-reader variability. 

With this background, the purpose of this chapter is to delve into variability in some detail and in particular describe computational methods for estimating them. This chapter introduces the concept of resampling a dataset to estimate variability and the widely used bootstrap and jackknife methods of estimating variance are described. In a later chapter, these are extended to estimating covariance (essentially a scaled version of the correlation) between two random variables.

The starting point is the simplest scenario: a single reader interpreting a case-set.

## Dependence of AUC on the case sample

Suppose a researcher conducts a ROC study with a single reader. The researcher starts by selecting a case-sample, i.e., a set of proven-truth non-diseased and diseased cases. Another researcher conducting another ROC study at the same institution selects a different case-sample, i.e., a different set of proven-truth non-diseased and diseased cases. The two case-sets contain the same numbers $K_1,K_2$ of non-diseased and diseased cases, respectively. Even if the same radiologist interprets the two case-sets, and the reader is perfectly reproducible, the AUC values are expected to be different. Therefore, AUC must depend on a case sample index, which is denoted $\{c\}$, where $c$ is an integer: $c = 1, 2$, as there are two case-sets in the study as envisaged. 

\begin{equation} 
AUC\rightarrow AUC_{\{c\}}
(\#eq:sources-of-variability-case-set)
\end{equation}

Note that $\{c\}$ is not an individual *case* index, rather it is a *case-set* index, i.e., different integer values of $c$ denote different sets, or samples, or groups, or collections of cases. [The dependence of AUC on the case sample index is not explicitly shown in the literature.] 

What does the dependence of AUC on the $c$ index mean? Different case samples differ in their *difficulty* levels. A difficult case set contains a greater fraction of difficult cases than is usual. A difficult diseased case is one where disease is difficult to detect. For example, the lesions could be partly obscured by overlapping normal structures in the patient anatomy; i.e., the lesion does not “stick out”. Alternatively, variants of normal anatomy could mimic a lesion, like a blood vessel viewed end on in a chest radiograph, causing the radiologist to miss the real lesion(s) and mistake these blood vessels for lesions. An easy diseased case is one where the disease is easy to detect. For example, the lesion is projected over smooth background tissue, because of which it “sticks out”, or is more conspicuous2. How does difficulty level affect non-diseased cases? A difficult non-diseased case is one where variants of normal anatomy mimic actual lesions and could cause the radiologist to falsely diagnose the patient as diseased. Conversely, an easy non-diseased case is like a textbook illustration of normal anatomy. Every structure in it is clearly visualized and accounted for by the radiologist’s knowledge of the patient’s non-diseased anatomy, and the radiologist is confident that any abnormal structure, *if present*, would be readily seen. The radiologist is unlikely to falsely diagnose the patient as diseased. Difficult cases tend to be rated in the middle of the rating scale, while easy ones tend to be rated at the ends of the rating scale. 

### Case sampling variability of AUC

An easy case sample will cause AUC to increase over its average value; interpreting many case-sets and averaging the AUCs determines the average value.  Conversely, a difficult case sample will cause AUC to decrease. Case sampling variability causes variability in the measured AUC. How does one estimate this essential source of variability? One method, totally impractical in the clinic but easy with simulations, is to have the same radiologist interpret repeated samples of case-sets from the population of cases (i.e., patients), termed *population sampling*, or more viscerally, as the "brute force" method. 

Even if one could get a radiologist to interpret different case-sets, it is even more impractical to actually acquire the different case samples of truth-proven cases. Patients do not come conveniently labeled as non-diseased or diseased. Rather, one needs to follow-up on the patients, perhaps do other imaging tests, in order to establish true disease status, or ground-truth. In screening mammography, a woman who continues to be diagnosed as non-diseased on successive yearly screening tests in the US, and has no other symptoms of breast disease, is probably disease-free. Likewise, a woman diagnosed as diseased and the diagnosis is confirmed by biopsy (i.e., the biopsy comes back showing a malignancy in the sampled tissues) is known to be diseased. However, not all patients who are diseased are actually diagnosed as diseased: a typical false negative fraction is 20% in screening mammography3. This is where follow-up imaging can help determine true disease status at the initial screen. A false negative mistake is unlikely to be repeated at the next screen. After a year, the tumor may have grown, and is more likely to be detected. Having detected the tumor in the most recent screen, radiologists can go back and retrospectively view it in the initial screen, at which it was missed during the "live" interpretation. If one knows where to look, the cancer is easier to see. The previous screen images would be an example of a difficult diseased case. In unfortunate instances, the patient may die from the previously undetected cancer, which would establish the truth status at the initial screen, too late to do the patient any good. The process of determining actual truth is often referred to as defining the "gold standard", the *ground truth*: or simply *truthing*. 

*One can appreciate from this discussion that acquiring independently proven cases, particularly diseased ones, is one of the most difficult aspects of conducting an observer performance study*.

There has to be a better way of estimating case-sampling variability. With a parametric model, the maximum likelihood procedure provides a means of estimating variability of each of the estimated parameters, which can be used to estimate the variability of $A_z$, as in Chapter `TempComment \@ref(binormal-model)`. The estimate corresponds to case-sampling variability (including an inseparable within-reader variability). If unsure about this point, the reader should run some of the examples in Chapter `TempComment \@ref(binormal-model)` with increased numbers of cases. The variability is seen to decrease.

There are other options available for estimating case-sampling variance of AUC, and this chapter is not intended to be comprehensive. Three commonly used options are described: the DeLong et al method, the bootstrap and the jackknife resampling methods.

## DeLong method {#sources-of-variabilityDeLong}

If the figure-of-merit is the empirical AUC, then a procedure developed by DeLong et al4 (henceforth abbreviated to DeLong) is applicable that is based on earlier work by [@RN2530] and [@bamber1975area]. The author will not go into details of this procedure but limit to showing that it "works". However, before one can show that it "works", one needs to know the true value of the variance of empirical AUC. Even if data were simulated using the binormal model, one cannot use the binormal model based estimate of variance as it is an estimate, not to be confused with a true value. Estimates are realizations of random numbers and are themselves subject to variability, which decreases with increasing case-set size. Instead, a “brute-force” (i.e., simulated population sampling) approach is adopted to determine the true value of the variance of AUC. The simulator provides a means of repeatedly generating case-sets interpreted by the same radiologist, and by sampling it enough time, e.g., $C$ = 10,000 times, each time calculating AUC, one determines the population mean and standard deviation. The standard deviation determined this way is compared to that yielded by the DeLong method to check if the latter actually works.  




```r
bruteForceEstimation <-
  function(seed, mu, sigma, K1, K2) {
    # brute force method to
    # find the population 
    # meanempAuc and stdDevempAuc
    empAuc <- array(dim = 10000)
    for (i in 1:length(empAuc)) {
      zk1 <- rnorm(K1)
      zk2 <- rnorm(K2, mean = mu, sd = sigma)
      empAuc[i] <- Wilcoxon(zk1, zk2)
    }
    stdDevempAuc  <-  sqrt(var(empAuc))
    meanempAuc   <-  mean(empAuc)
    return(list(
      meanempAuc = meanempAuc,
      stdDevempAuc = stdDevempAuc
    ))
  }
```



```r
seed <- 1;set.seed(seed)
mu <- 1.5;sigma <- 1.3;K1 <- 50;K2 <- 52
ret <- bruteForceEstimation(seed, mu, sigma, K1, K2)
# one more trial
zk1 <- rnorm(K1)
zk2 <- rnorm(K2, mean = mu, sd = sigma)
empAuc <- Wilcoxon(zk1, zk2)
ret1  <- DeLongVar(zk1,zk2)
stdDevDeLong <- sqrt(ret1)
cat("brute force estimates:",
    "\nempAuc = ",
    ret$meanempAuc,
    "\npopulation standard deviation =",
    ret$stdDevempAuc, "\n")
#> brute force estimates: 
#> empAuc =  0.819178 
#> population standard deviation = 0.04176683

cat("single sample estimates = ",
    "\nempirical AUC",
    empAuc,
    "\nstandard deviation DeLong = ",
    stdDevDeLong, "\n")
#> single sample estimates =  
#> empirical AUC 0.8626923 
#> standard deviation DeLong =  0.03804135
```

Two functions needed for this code to work are not shown: `Wilcoxon()` calculates the Wilcoxon statistic and the `DeLongVar()` implements the DeLong variance computation method (the DeLong method also calculates co-variances, but these are not needed in the current context). Line 1 sets the `seed` of the random number generator to 1. The `seed` variable is completely analogous to the case-set index `c`. Keeping `seed` fixed realizes the same random numbers each time the program is run. Different values of `seed` result in different, i.e., statistically independent, random samples. Line 2 initialize the values $(\mu, \sigma, K_1, K_2)$ needed by the data simulator: the normal distributions are separated by $\mu = 1.5$, the standard deviation of the diseased distribution is $\sigma = 1.3$,  and there are $K_1 = 50$ non-diseased and $K_2 = 52$ diseased cases. Line 3 calls `bruteForceEstimation`, the “brute force” method for estimating mean and standard deviation of the population distribution of AUC, returned by this function, which are the "correct" value to which the DeLong standard deviation estimate will be compared. Lines 4-9 generates a fresh ROC dataset to which the DeLong method is applied. 

Two runs of this code were made, one with the smaller sample size, and the other with 10 times the sample size (the second run takes much longer). A third run was made with the larger sample size but with a different seed value. The results follow:


```r
seed <- 2;set.seed(seed)
mu <- 1.5;sigma <- 1.3;K1 <- 500;K2 <- 520
ret <- bruteForceEstimation(seed, mu, sigma, K1, K2)
# one more trial
zk1 <- rnorm(K1)
zk2 <- rnorm(K2, mean = mu, sd = sigma)
empAuc <- Wilcoxon(zk1, zk2)
ret1  <- DeLongVar(zk1,zk2)
stdDevDeLong <- sqrt(ret1)
cat("brute force estimates:",
    "\nempAuc = ",
    ret$meanempAuc,
    "\npopulation standard deviation =",
    ret$stdDevempAuc, "\n")
#> brute force estimates: 
#> empAuc =  0.8194988 
#> population standard deviation = 0.01300203

cat("single sample estimates = ",
    "\nempirical AUC",
    empAuc,
    "\nstandard deviation DeLong = ",
    stdDevDeLong, "\n")
#> single sample estimates =  
#> empirical AUC 0.8047269 
#> standard deviation DeLong =  0.01356696
```


1.	An important observation is that as sample-size increases, case-sampling variability decreases: 0.0417 for the smaller sample size vs. 0.01309 for the larger sample size, and the dependence is as the inverse square root of the numbers of cases, as expected from the central limit theorem. 
2.	With the smaller sample size (K1/K2 = 50/52; the back-slash notation, not to be confused with division, is a convenient way of summarizing the case-sample size) the estimated standard deviation (0.038) is within 10% of that estimated by population sampling (0.042). With the larger sample size, (K1/K2 = 500/520) the two are practically identical (0.01300203 vs. 0.01356696 – the latter value is for seed = 2). 
3.	Notice also that the one sample empirical AUC for the smaller case-size is 0.863, which is less than two standard deviations from the population mean 0.819. The "two standard deviations" comes from rounding up 1.96: as in Eqn. `TempComment \@ref(eq:binary-task-model-def-z-alpha2)`, where $z_{\alpha/2}$ was defined as the upper $1-\alpha/2$ quantile of the unit normal distribution and $z_{0.025}=1.96$. 
4.	To reiterate, with clinical data the DeLong procedure estimates case sampling plus within reader variability. With simulated data as in this example, there is no within-reader variability as the simulator yields identical values for fixed seed.

This demonstration should convince the reader that one does have recourse other than the “brute force” method, at least when the figure of merit is the empirical area under the ROC. That should come as a relief, as population sampling is impractical in the clinical context. It should also impress the reader, as the DeLong method is able to use information present in a *single dataset* to tease out its variability. [This is not magic: the MLE estimate is also able to tease out variability based on a parametric fit to a single dataset and examination of the sharpness of the peak of the log-likelihood function, Chapter `TempComment \@ref(binormal-model)`, as are the resampling methods described next.]

Next, two resampling–based methods of estimating case-sampling variance of AUC are introduced. The word “resampling” means that the dataset itself is regarded as containing information regarding its variability, which can be extracted by sampling from the original data (hence the word "resampling"). These are general and powerful techniques, applicable to any scalar statistic, not just the empirical AUC, which one might be able to use in other contexts.

## Bootstrap method {#sources-of-variabilityBootstrap}

The simplest resampling method, at least at the conceptual level, is the bootstrap. *The bootstrap method is based on the assumption that one can regard the observed sample as defining the population from which it was sampled.* Since by definition a population cannot be exhausted, the idea is to resample, *with replacement*, from the observed sample. Each resampling step realizes a particular bootstrap sample set denoted $\{b\}$, where $b = 1, 2, ..., B$. The curly brackets emphasize that different integer values of $b$ denote different *sets of cases*, not individual cases. [In contrast, the notation $(k)$ will be used to denote *removing* a specific case, $k$, as in the jackknife procedure to be described shortly. The index $b$ should not be confused with the index $c$, the case sampling index; the latter denotes repeated sampling from the population, which is impractical in real life; the bootstrap index denotes repeated sampling from the dataset, which is quite feasible.] The procedure is repeated $B$ times, typically $B$ can be as small as 200, but to be safe I generally use about 1000 - 2000 bootstraps. The following example uses Table `TempComment \@ref(tab:ratings-paradigm-example-table)` from Chapter `TempComment \@ref(ratings-paradigm)`. 

For convenience, let us denote cases as follows. The 30 non-diseased cases that received the 1 rating are denoted $k_{1,1},k_{2,1},...,k_{30,1}$. The second index denotes the truth state of the cases. Likewise, the 19 non-diseased cases that received the 2 rating are denoted $k_{31,1},k_{32,1},...,k_{49,1}$ and so on for the remaining non-diseased cases. The 5 diseased cases that received the 1 rating are denoted $k_{1,2},k_{2,2},...,k_{5,2}$, the 6 diseased cases that received the 2 rating are denoted $k_{6,2},k_{7,2},...,k_{11,2}$, and so on. Let us figuratively "put" all non-diseased cases (think of each case as an index card, with the case notation and rating recorded on it) into one hat (the non-diseased hat) and all the diseased cases into another hat (the diseased hat). Next, one randomly picks one case (card) from the non-diseased hat, records it's rating, and puts the case back in the hat, so that it is free to be possibly picked again. This is repeated 60 times for the non-diseased hat resulting in 60 ratings from non-diseased cases. A similar procedure is performed using the diseased hat, resulting in 50 ratings from diseased cases. The author has just described, in painful detail (one might say) the realization of the 1st bootstrap sample, denoted $\{b=1\}$. This is used to construct the 1st bootstrap counts table, Table \@ref(tab:sources-of-variabilitybs1). 


<table>
<caption>(\#tab:sources-of-variabilitybs1)Representative counts table.</caption>
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:right;"> $r = 5$ </th>
   <th style="text-align:right;"> $r = 4$ </th>
   <th style="text-align:right;"> $r = 3$ </th>
   <th style="text-align:right;"> $r = 2$ </th>
   <th style="text-align:right;"> $r = 1$ </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> non-diseased </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 16 </td>
   <td style="text-align:right;"> 35 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> diseased </td>
   <td style="text-align:right;"> 19 </td>
   <td style="text-align:right;"> 8 </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 7 </td>
  </tr>
</tbody>
</table>


So what happened? Consider the 35 non-diseased cases with a 1 rating. If each non-diseased case rated 1 in Table `TempComment \@ref(tab:ratings-paradigm-example-table)` were picked one time, the total would have been 30, but it is 35. Therefore, some of the original non-diseased cases rated 1 must have been picked multiple times, but one must also make allowance as there is no guarantee that a specific case was picked at all. Still focusing on the 35 non-diseased cases with a 1 rating in the first bootstrap sample, the picked labels, reordered after the fact, with respect to the first index, might be: 

\begin{equation} 
k_{2,1},k_{2,1},k_{4,1},k_{4,1},k_{4,1},k_{6,1},k_{7,1},k_{7,1},k_{9,1},...,k_{28,1},k_{28,1},k_{30,1},k_{30,1}
(\#eq:sources-of-variabilitybs1-bs1-non-diseased)
\end{equation}

In this example, case $k_{1,1}$ was not picked, case $k_{2,1}$ was picked twice, case $k_{3,1}$ was not picked, case $k_{4,1}$ was picked three times, case $k_{5,1}$ was not picked, case $k_{6,1}$ was picked once, etc. The total number of cases in Eqn. \@ref(eq:sources-of-variabilitybs1-bs1-non-diseased) is 35, and similarly for the other cells in Table \@ref(tab:sources-of-variabilitybs1). Next, one estimates AUC for this table. Using the Eng website referred to earlier, one gets AUC = 0.843. [It is OK to use a parametric FOM since the bootstrap is a general procedure applicable, in principle, to any FOM, not just the empirical AUC, unlike the DeLong method, which is restricted to empirical AUC.] The corresponding value for the original data, Table `TempComment \@ref(tab:ratings-paradigm-example-table)`, was AUC = 0.870. The first bootstrapped dataset yielded a smaller value than the original dataset because one happened to have picked an unusually difficult bootstrap sample. 

[Notice that in the original data there were 6 + 5 = 11 diseased cases that were rated 1 and 2, but in the bootstrapped dataset there are 7 + 9 = 16 diseased cases that were rated 1 and 2; in other words, the number of incorrect decisions on diseased cases went up, which would tend to lower AUC. Counteracting this effect is the increase in number of correct decisions on diseased cases: 8 + 19 = 27 cases rated 4 and 5, as compared to 12 + 22 = 34 in the original dataset. Reinforcing the effect is that increase in the number of correct decisions on non-diseased cases, albeit minimally: 35 + 16 = 51 rated 1 and 2 vs. 30 + 19 = 49 in the original dataset, and zero counts rated 4 and 5 in the non-diseased vs. 2 + 1 = 3 in the diseased. The complexity of following this *post-facto justification* illustrates the difficulty, in fact the futility, of correctly predicting which way performance will go from comparison of the two ROC counts tables – too many numbers are changing and in the above one did not even consider the change in counts in the bin labeled 4! Hence, the need for an objective figure of merit, such as the binormal model based AUC or the empirical AUC.] 

To complete the description of the bootstrap method, one repeats the procedure described in the preceding paragraphs $B = 200$ times, each time running the website calculator and the final result is $B$ values of AUC, denoted: 

$$AUC_{\{1\}},AUC_{\{2\}},...,AUC_{\{B\}}$$

where $AUC_{\{1\}}=0.843$, etc. The bootstrap estimate of the variance of AUC is defined by [@RN2261]:

\begin{equation} 
\text{Var}\left ( AUC \right )=\frac{1}{B-1}\sum_{b=1}^{B}\left ( AUC_{\{b\}}-AUC_{\{ \bullet\}} \right )^2
(\#eq:sources-of-variabilityVar-bs)
\end{equation}

The right hand side is the traditional definition of (unbiased) variance. The dot represents the average over the *replaced index*. Of course, running the website code 200 times and recording the outputs is not a productive use of time. The following code implements two methods for estimating AUC, the empirical AUC, described in `TempComment Chapter \@ref(empirical-auc)` and the binormal model estimate of AUC, described in Chapter `TempComment \@ref(binormal-model)`.

### Demonstration of the bootstrap method

To minimize clutter, several `R` functions are not shown, but they are compiled. To display them `clone` or 'fork' the book repository and look at the `Rmd` file corresponding to this output and the sourced `R` files listed below:


```r
source(here("R/CH07-Variability/Transforms.R"))
source(here("R/CH07-Variability/LL.R"))
source(here("R/CH07-Variability/RocfitR.R")) 
source(here("R/CH07-Variability/RocOperatingPoints.R"))
source(here("R/CH07-Variability/FixRocCountsTable.R"))
source(here("R/CH07-Variability/WilcoxonCountsTable.R"))
```



```r
doBootstrap <- function(parametricFOM, B, seed, RocTable) {
  # this is the K vector 
  K <- c(sum(RocTable[1,]), sum(RocTable[2,]))
  
  if (parametricFOM) {
    ret <- RocfitR(RocTable)                 
  } else {
    ret <- WilcoxonCountsTable(RocTable) 
  }
  OrigAUC  <- ret$AUC
  
  # ready to bootstrap
  # first put the counts data into a linear array
  # convert counts table to array
  z1  <- rep(1:length(RocTable[1,]),
             RocTable[1,])
  z2  <- rep(1:length(RocTable[2,]),
             RocTable[2,])#do:
  AUC <- array(dim = B)#to save the bs AUC values
  for ( b in 1 : B){
    while (1) {
      RocTable_bs  <- 
        array(dim = c(2,length(RocTable[1,])))
      # bs indices for non-diseased    
      k1_b <- ceiling( runif( K[ 1 ] ) * K[ 1 ] )
      # bs indices for diseased
      k2_b <- ceiling( runif( K[ 2 ] ) * K[ 2 ] )
      bsTable <- table(z1[k1_b])
      #convert array to frequency table
      RocTable_bs[1,as.numeric(names(bsTable))] <- 
        bsTable
      bsTable <- table(z2[k2_b])
      #do:
      RocTable_bs[2,as.numeric(names(bsTable))] <- 
        bsTable
      #replace NAs with zeroes
      RocTable_bs[is.na(RocTable_bs )] <- 0
      if (parametricFOM) {
        temp <- RocfitR(RocTable_bs)
      } else {
        temp <- WilcoxonCountsTable(RocTable_bs) 
      }    
      AUC[b]  <- temp$AUC
      # a return of -1 means AUC did not converge
      if (AUC[b] != -1) break
    }
  }
  meanAUCboot <- mean(AUC)
  Var <- var(AUC)
  stdAUCboot  <- sqrt(Var)
  return(list(
    OrigAUC = OrigAUC,
    meanAUCboot = meanAUCboot,
    stdAUCboot = stdAUCboot
  ))
}
```


Since the bootstrap method is applicable to any scalar figure of merit, two options are provided in the code. If `parametricFOM` is set to `TRUE`, then the binormal model estimate is used, and if set to `FALSE`, the empirical AUC is used. The first set of results are obtained with `parametricFOM` set to `TRUE`.



```r
parametricFOM <- TRUE
B <- 200;seed <- 1;set.seed(seed)
RocTable = array(dim = c(2,5))
RocTable[1,]  <- c(30,19,8,2,1)
RocTable[2,]  <- c(5,6,5,12,22)

ret <- doBootstrap(parametricFOM, B, seed, RocTable)
OrigAUC <- ret$OrigAUC
meanAUCboot <- ret$meanAUCboot
stdAUCboot <- ret$stdAUCboot

cat("Bootstrap variance estimation:",
    "\nparametricFOM = ", parametricFOM,
    "\nseed = ", seed, 
    "\nB = ", B,
    "\nOrigAUC = ", OrigAUC, 
    "\nmeanAUCboot = ", meanAUCboot, 
    "\nstdAUCboot = ", stdAUCboot, "\n")
#> Bootstrap variance estimation: 
#> parametricFOM =  TRUE 
#> seed =  1 
#> B =  200 
#> OrigAUC =  0.8704519 
#> meanAUCboot =  0.8671713 
#> stdAUCboot =  0.04380523
```

This shows that the AUC of the original data (i.e., before performing any bootstrapping) is 0.870, the mean AUC of the B = 200 bootstrapped datasets is 0.867, and the standard deviation of the 200 bootstraps is 0.0438. If one runs the website calculator referenced in the previous chapter on the dataset shown in Table `TempComment \@ref(tab:ratings-paradigm-example-table)`, one finds that the MLE of the standard deviation of the AUC of the fitted ROC curve is 0.0378. The standard deviation is itself a statistic and there is sampling variability associated with it, i.e., there exists such a beast as a standard deviation of a standard deviation; the bootstrap estimate is not too far from the MLE estimate. By setting `seed` to different values, one gets an idea of the variability of the estimate of the standard deviation of AUC. For example, with `seed` = 2, one gets: 



```
#> Bootstrap variance estimation: 
#> parametricFOM =  TRUE 
#> seed =  2 
#> B =  200 
#> OrigAUC =  0.8704519 
#> meanAUCboot =  0.8673155 
#> stdAUCboot =  0.03815402
```

Note that both the mean of the bootstrap samples and the standard deviation have changed, but both are close to the MLE values. Examined next is the dependence of the estimates on `B`, the number of bootstraps. With `seed` = 1 and `B` = 2000 one gets: 



```
#> Bootstrap variance estimation: 
#> parametricFOM =  TRUE 
#> seed =  1 
#> B =  2000 
#> OrigAUC =  0.8704519 
#> meanAUCboot =  0.8674622 
#> stdAUCboot =  0.03833508
```


The estimates are evidently rather insensitive to $B$, but the computation time was longer, ~13 seconds (running MLE 2000 times in 13 seconds is not bad!). It is always a good idea to test the stability of the results to different $B$ and `seed` values. Unlike the DeLong et al method, which is restricted to the Wilcoxon statistic (which equals empirical AUC as per the Bamber theorem), the bootstrap is broadly applicable to other figures of merit, including non-ROC paradigm figures of merit. However, beware that it depends on the assumption that the sample itself is representative of the population. With limited numbers of cases, this could be a bad assumption. [With small numbers of cases it is relatively easy to enumerate the different outcomes of the sampling process and, more importantly, their respective probabilities, leading to what is termed the *exact bootstrap*. It is "exact" in the sense that there is no seed variable or number of bootstrap dependence.] 

Finally, here is the output when using non-parametric AUC, with `seed` = 1.


```
#> Bootstrap variance estimation: 
#> parametricFOM =  FALSE 
#> seed =  1 
#> B =  200 
#> OrigAUC =  0.8606667 
#> meanAUCboot =  0.8604575 
#> stdAUCboot =  0.04125475
```

 
## Jackknife method {#sources-of-variabilityJackknife}

The second resampling method, termed the *jackknife*, is computationally less demanding, but as was seen with the bootstrap, with modern personal computers computational limitations are no longer that important, at least for the types of analyses that this book is concerned with.
 
In this method, the first case is removed, or jackknifed, from the set of cases and the MLE (or empirical estimation) is conducted on the resulting dataset, which has one less case. Let us denote by $AUC_{(1)}$ the resulting value of AUC. The parentheses around the subscript $1$ are meant to emphasize that the AUC value corresponds to that with the first case *removed* from the original dataset. Next, the first case is replaced, and now the second case is removed, the new dataset is analyzed yielding $AUC_{(2)}$, and so on, yielding $K$ ($K$ is the total number of cases; $K=K_1+K_2$) *jackknife AUC values*: 

\begin{equation} 
AUC_{(k)} \qquad k=1,2,...,K
(\#eq:sources-of-variabilityAUC-jackknife-fom)
\end{equation}

The corresponding jackknife pseudovalues $Y_k$ are defined by:

\begin{equation} 
Y_k = K\times AUC - \left ( K-1 \right ) \times AUC_{(k)}
(\#eq:sources-of-variability-pseudovalues-Y)
\end{equation}

Here AUC denotes the estimate using the entire dataset, i.e., not removing any cases. The jackknife pseudovalues will turn out to be of central importance in TBA Chapter 09. The *jackknife AUC values*, defined by Eqn. \@ref(eq:sources-of-variabilityAUC-jackknife-fom), should not be confused with jackknife derived psuedovalues, defined by Eqn. \@ref(eq:sources-of-variability-pseudovalues-Y). 

The jackknife estimate of the variance is defined by [@RN2261]:

\begin{equation} 
\text{Var}_{\text{jack}} = \frac{\left ( K-1 \right )^2}{K} \frac{1}{K-1} \sum_{k=1}^{K} \left ( AUC_{(k)} - AUC_{(\bullet)} \right )^2
(\#eq:sources-of-variability-jackknife-variance-definition)
\end{equation}

Since variance of $K$ scalars is defined by:

\begin{equation} 
\text{Var}\left ( x \right ) = \frac{1}{K-1} \sum_{k=1}^{K} \left ( x_{k} - x_{\bullet} \right )^2
(\#eq:sources-of-variability-variance-definition)
\end{equation}

It follows that:

\begin{equation} 
\text{Var}_{\text{jack}} \left ( \text{AUC} \right ) = \frac{\left ( K-1 \right )^2}{K} \text{Var} \left ( \text{AUC} \right )
(\#eq:sources-of-variability-jackknife-variance-ordinary-variance)
\end{equation}

In Eqn. \@ref(eq:sources-of-variability-jackknife-variance-definition) I have deliberately not simplified the right hand side by canceling out $K-1$. The purpose is to show, Eqn. \@ref(eq:sources-of-variability-jackknife-variance-ordinary-variance), that the usual expression for the variance (of the jackknife FOM values) needs to be multiplied by a **variance inflation factor**  $\frac{\left ( K-1 \right )^2}{K}$, which is approximately equal to $K$, in order to obtain the correct jackknife estimate of variance of AUC. This factor was not necessary when one used the bootstrap method. That is because the bootstrap samples are more representative of the actual spread in the data. The jackknife samples are more restricted than the bootstrap samples, so the spread of the data is smaller; hence the need for the variance inflation factor [@RN2261]. 


```r
doJackknife <- function(parametricFOM, RocTable) {
  # this is the K vector 
  K <- c(sum(RocTable[1,]), sum(RocTable[2,]))
  
  if (parametricFOM) {
    ret <- RocfitR(RocTable)                 
  } else {
    ret <- WilcoxonCountsTable(RocTable) 
  }
  OrigAUC  <- ret$AUC
  
  # first put the counts data into a linear array
  z1  <- rep(1:length(RocTable[1,]),
             RocTable[1,])
  z2  <- rep(1:length(RocTable[1,]),
             RocTable[2,])
  
  AUC_jack  <- array(dim = sum(K))
  Y_k <- array(dim = sum(K))
  z_jk  <- array(dim = sum(K))
  # ready to jackknife
  for ( k in 1 : sum(K)){
    RocTable_jk  <- array(dim = c(2,length(RocTable[1,])))
    if ( k <= K[ 1 ]){
      z1_jk <- z1[ -k ]
      z2_jk <- z2
    }else{
      z1_jk <- z1
      z2_jk <- z2[ -(k - K[ 1 ]) ] 
    }  
    #convert array to frequency table
    RocTable_jk[1,1:length(table(z1_jk))]  <- 
      table(z1_jk)
    RocTable_jk[2,1:length(table(z2_jk))]  <- 
      table(z2_jk)
    #replace NAs with zeroes
    RocTable_jk[is.na(RocTable_jk)] <- 0
    # AUC_jack for observed data
    if (parametricFOM) {
      temp <- RocfitR(RocTable_jk) 
    } else {
      temp <- WilcoxonCountsTable(RocTable_jk) 
    }      
    AUC_jack[k]  <- temp$AUC
    Y_k[k] <- sum(K)*OrigAUC - (sum(K)-1)*AUC_jack[k]
    if (AUC_jack[k] == -1) 
      stop("RocfitR did not converge in jackknife loop") 
  }
  meanAUCjack <- mean(AUC_jack)
  #Efron and Stein's paper, include jackknife inflation factor
  Var_jack <- var(AUC_jack) * ( sum(K) - 1)^2 / sum(K) 
  stdAUCjack  <- sqrt(Var_jack)
  return(list(
    OrigAUC = OrigAUC,
    meanAUCjack = meanAUCjack,
    stdAUCjack = stdAUCjack
  ))
}
```


Since the jackknife method is applicable to any scalar figure of merit, two options are provided in the code. If `parametricFOM` is set to `TRUE`, then the binormal model estimate is used, and if set to `FALSE`, the empirical AUC is used. The first set of results are obtained with `parametricFOM` set to `TRUE`. Notice that the code does not use a `set.seed()` statement, as no random number generator is needed in the jackknife method. Systematically removing and replacing each case in sequence, one at a time, is not random sampling, which should further explain the need for the variance inflation factor in Eqn. \@ref(eq:sources-of-variability-jackknife-variance-ordinary-variance). 



```r
parametricFOM <- TRUE
RocTable = array(dim = c(2,5))
RocTable[1,]  <- c(30,19,8,2,1)
RocTable[2,]  <- c(5,6,5,12,22)

ret <- doJackknife(parametricFOM, RocTable)
OrigAUC <- ret$OrigAUC
meanAUCjack <- ret$meanAUCjack
stdAUCjack <- ret$stdAUCjack

cat("Jackknife variance estimation:",
    "\nparametricFOM = ", parametricFOM,
    "\nOrigAUC = ", OrigAUC, 
    "\nmeanAUCjack = ", meanAUCjack, 
    "\nstdAUCjack = ", stdAUCjack, "\n")
#> Jackknife variance estimation: 
#> parametricFOM =  TRUE 
#> OrigAUC =  0.8704519 
#> meanAUCjack =  0.8704304 
#> stdAUCjack =  0.03861591
```

The next output is with the non-parametric figure of merit:



```
#> Jackknife variance estimation: 
#> parametricFOM =  FALSE 
#> OrigAUC =  0.8606667 
#> meanAUCjack =  0.8606667 
#> stdAUCjack =  0.03689264
```

It may be noticed that the mean of the jackknife figure of merit values, i.e., 0.8606667, exactly equals the original figure of merit 0.8606667 (i.e., that calculated including all cases). This can be shown analytically to be true so long as the figure of merit is the empirical AUC. A similar relation is not true for the bootstrap.

## Calibrated simulator {#sources-of-variabilityCalSimulator}

### The need for a calibrated simulator

The population sampling method used previously, \@ref(sources-of-variabilityDeLong), to compare the DeLong method to a known standard used arbitrarily set simulator values, i.e., $\mu = 1.5$ and $\sigma = 1.3$. One does not know if these values actually represent real clinical data. In this section a simple method of implementing population sampling using a *calibrated simulator* is described. A calibrated simulator is one whose parameters are chosen to match those of an actual clinical dataset. This way one has some assurance that the simulator is realistic and therefore its verdict on a proposed method or analysis (in our case method of estimating AUC variability) is likely to be correct.  

### Implementation of a simple calibrated simulator

The simple simulator described here is limited to a single reader single modality dataset. A more complex simulator describing multiple readers in multiple modalities is described in a later chapter (TBA). Consider a clinical dataset, such as in Table `TempComment \@ref(tab:ratings-paradigm-example-table)`. Analyzed by MLE, this yields binormal model parameters, `a`, `b` and the thresholds $\zeta_1,\zeta_2,\zeta_3,\zeta_4$. After conversion to $\mu=a/b$ and $\sigma= 1/b$ and new zetas $\zeta = \zeta/b$, the values are (in the same order): 2.173597, 1.646099, 0.01263423, 1.475351, 2.494901, 3.945221 (see code output below): 


```r
# mu_sigma is the mu-sigma notation
mu_sigma <- c(2.173597, 1.646099, 0.01263423, 1.475351, 2.494901, 3.945221)
# ab is the a-b notation
ab <- c(1.320453, 0.607497, 0.007675259, 0.8962713, 1.515645, 2.39671)
ab[1]/ab[2] # this is mu
#> [1] 2.173596
1/ab[2]    # this is sigma
#> [1] 1.646099
ab[3:6]/ab[2] # this is zeta in mu-sigma notation
#> [1] 0.01263423 1.47535099 2.49490121 3.94522113
```


[The reason for dividing $\zeta$ by $b$ is that when re-scaling the decision variable axis by $b$ one must also re-scale the cutoffs.] The values $\mu,\sigma,\zeta$ define the calibrated simulator, in the sense that the parameter values are calibrated to match the dataset in Table `TempComment \@ref(tab:ratings-paradigm-example-table)`. 





Here is the function `doCalSimulator()` that will be used to perform the initial calibration followed by population sampling from the calibrated simulator:



```{.r .numberLines}
doCalSimulator <- function(P, parametricFOM, RocCountsTable) {
  K <- c(sum(RocCountsTable[1,]), 
         sum(RocCountsTable[2,]))
  # perform the initial calibration
  ret <- RocfitR(RocCountsTable) # AUC for observed data
  a <- ret$a
  b  <- ret$b
  zetas <- ret$zeta
  mu  <- a/b
  sigma <- 1/b
  zetas <- zetas/b # need to also scale zetas
   # AUC for observed data
  if (parametricFOM) {
    OrigAUC <- RocfitR(RocCountsTable)$AUC                 
  } else {
    OrigAUC <- WilcoxonCountsTable(RocCountsTable)$AUC 
  }
  # perform the population sampling
  AUC <- array(dim = P)
  for ( p in 1 : P){
    while (1) {
      
      RocCountsTableSimPop <- 
        SimulateRocCountsTable(K, mu, sigma, zetas)
      if (parametricFOM) {
        
        # AUC for fitted curve 
        temp <- RocfitR(RocCountsTableSimPop)
        # a return of -1 means RocFitR did not converge
        if (temp[1] != -1) {
          AUC[p]  <- temp$AUC
          break 
        }
      } else {
        AUC[p] <- (WilcoxonCountsTable(RocCountsTableSimPop))$AUC
        break 
      }    
    }
  }
  AUC <- AUC[!is.na(AUC)]
  meanAUC  <- mean(AUC)
  stdAUC  <- sqrt(var(AUC))
  return(list(
    mu = mu, # these define the calibration simulator
    sigma = sigma, #do:
    zetas = zetas, #do:
    OrigAUC = OrigAUC,
    meanAUC = meanAUC,
    stdAUC = stdAUC
  ))
}
```

In the function `doCalSimulator(P, parametricFOM, RocCountsTable)`, `P` is the desired number of population samples, `parametricFOM` is a logical, if set to `TRUE` the binormal model is used to calculate *fitted* AUC and otherwise the Wilcoxon statistic is used to calculate *empirical* AUC, and `RocCountsTable` contains the ROC data, such as Table `TempComment \@ref(tab:ratings-paradigm-example-table)`, to which the simulator is to be calibrated to. Lines 2-3 construct the K-vector, containing $K_1,K_2$. Line 5 performs the maximum likelihood fit, using function `RocfitR(RocCountsTable)`. The returned variable contains $a, b, \zeta$ as a `list`, which are extracted at lines 6-8. Lines 9-11 converts these to the mu-sigma notation. In essence, lines 5 - 11 calibrates the simulator and the calibrated values of the simulator are contained in $\mu,\sigma,\zeta$. Lines 13-17 calculates `OrigAUC`, the AUC of the original data, using parametric `RocfitR` or the Wilcoxon statistic, as appropriate, depending on the value of `parametricFOM`. After defining a length `P` array, at line 19, to hold the sampled `AUC` values, lines 20-39 begins and ends a `for` loop to conduct the `P` population samples. Each pass through the `for` loop yields $K_1$ samples from the non-diseased distribution and $K_2$ samples from the diseased distribution, returned in the variable `RocCountsTableSimPop`, which is similar in structure to a counts table like Table `TempComment \@ref(tab:ratings-paradigm-example-table)`. Within the `for` loop there is an endless `while` loop, needed because `RocfitR` can sometimes fail to converge, signaled by the first member of the returned `list` being minus 1, in which case another iteration of the `while` loop is performed (see line 30) and otherwise the `break` statement (line 32) causes program execution to proceed to the next iteration of the `for` loop. After entering the `while` loop, lines 22-23, a new ROC counts table is generated. The returned `list` is saved to `temp` at line 28, and if `temp[1] != -1` (i.e., `RocfitR` did converge) the AUC value is saved to `AUC[p]`, line 31. Upon exiting the code one has `P` values of AUC in the array `AUC`.

#### Parametric AUC results

The following code uses the function just described and prints out the results.


```r
parametricFOM <- TRUE
seed <- 1
set.seed(seed)
P <- 2000
RocCountsTable = array(dim = c(2,5))
RocCountsTable[1,]  <- c(30,19,8,2,1)
RocCountsTable[2,]  <- c(5,6,5,12,22)
ret <- doCalSimulator(P, parametricFOM, RocCountsTable)
mu <- ret$mu
sigma <- ret$sigma
zetas <- ret$zetas
meanAUC_1_2000 <- ret$meanAUC
stdAUC_1_2000 <- ret$stdAUC
```


After setting `parametricFOM` to `TRUE` (for a parametric fit), `seed` to 1 and `P` to 2000, the ROC counts table is defined and the function `doCalSimulator()` is called. The returned `list` contains the parameter values for the calibrated simulator: $\mu$ = 2.1735969,  $\sigma$ = 1.6460988 and  $\zeta$ = 0.0126342, 1.4753512, 2.4949012, 3.9452209. It also contains `OrigAUC`, the AUC of the original data, calculated by `RocfitR()`, in this case `OrigAUC` = 0.8704519, and the mean and standard deviation of the 2000 AUC values, equal to 0.8676727 and 0.0403331, respectively. 





The simulations were repeated with `seed` = 2. This time the mean and standard deviation of the 2000 AUC values, are equal to 0.8681855 and 0.0405516, respectively. The respective values corresponding to the two `seed` values are quite close to each other (to within a percent). 

More variability is observed, as expected, when the above two simulations are repeated with `P` = 200:



For `seed` = 1 and `P` = 200 the mean and standard deviation of the 200 AUC values, are 0.8727151 and 0.0355281, respectively. 




For `seed` = 2 and `P` = 200 the mean and standard deviation of the 200 AUC values, are 0.8649385 and 0.0450947, respectively. Note the greater variability induced by the change in `seed`, as compared to `P` = 2000.

#### Non-parametric AUC results






The next simulation is with `seed` = 1 and `P` = 2000, but this time `parametricFOM` is set to `FALSE`. The calibration proceeds as before, using `RocfitR` to determine the parameters of the simulation model, calibrating the simulator requires a parametric fit, but this empirical AUC is used to obtained the 2000 AUC samples. The mean and standard deviation of the AUC values, are 0.8497634 and 0.0367476, respectively. Note that these are smaller than the corresponding parametric estimates. The empirical AUC is expected to be smaller than the corresponding parametric AUC as joining adjacent points with straight lines will underestimate the area under the smooth ROC curve. Repeating with `seed` = 2, the mean and standard deviation of the AUC values, are 0.8503732 and 0.0369091, respectively, which are close to the `seed` = 1 values. 


## Discussion{#sources-of-variability-Discussion}

This chapter focused on the factors affecting variability of AUC, namely case-sampling and between-reader variability, each of which contain an inseparable within-reader contribution. The only way to get an estimate of within-reader variability is to have the same reader re-interpret the same case-set on multiple occasions, after a  sufficient time delay to minimize memory effects. This is rarely done and is unnecessary, in the ROC context, to sound experimental design and analysis. Some early publications have suggested that such re-interpretations are needed, but modern methods, described in the next part of the book, does not require re-interpretations. Indeed, it is a waste of precious reader-time resources. Rather than have the same readers re-interpret the same case-set on multiple occasions, it makes much more sense to recruit more readers and/or collect more cases, guided by a systematic sample size estimation method. Another reason I am not in favor of re-interpretations is that the within-reader variance is usually smaller than case-sampling and between-reader variances. Re-interpretations would minimize a quantity that is already small, which is not good practice.

The bootstrap and jackknife methods described in this chapter have wide applicability. Later they will be extended to estimating the covariance (essentially a scaled correlation) between two random variables. Also described was the DeLong method, applicable to the empirical AUC. Using a real dataset and simulators, all methods were shown to agree with each other, especially when the numbers of cases is large, Table 7.3 (row-D). 

The concept of a calibrated simulator was introduced as a way of "anchoring" a simulator to a real dataset. While relatively easy for a single dataset, the concept has yet to be extended to where it would be useful, namely designing a simulator calibrated to a dataset consisting of interpretations by multiple readers in multiple modalities of a common dataset. Just as a calibrated simulator allowed comparison of the different variance estimation methods to a known standard, obtained by population sampling, a more general calibrated simulator would allow better testing the validity of the analysis described in the next few chapters.

This concludes Part A of this book. The next chapter begins Part B, namely the statistical analysis of multiple-reader multiple-case (MRMC) ROC datasets. 

TBA: what to do with removed sections?

## Chapter References {#sources-of-variability-references}

