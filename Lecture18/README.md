STAT406 - Lecture 18 notes
================
Matias Salibian-Barrera
2018-11-12

LICENSE
-------

These notes are released under the "Creative Commons Attribution-ShareAlike 4.0 International" license. See the **human-readable version** [here](https://creativecommons.org/licenses/by-sa/4.0/) and the **real thing** [here](https://creativecommons.org/licenses/by-sa/4.0/legalcode).

Lecture slides
--------------

Preliminary lecture slides will be here.

What is Adaboost doing, *really*?
---------------------------------

Following the work of Friedman, Hastie, and Tibshirani [here](https://doi.org/10.1214/aos/1016218223) (see also Chapter 10 of \[HTF09\]), we saw in class that Adaboost can be interpreted as fitting an *additive model* in a stepwise (greedy) way, using an exponential loss. It is then easy to prove that Adaboost.M1 is computing an approximation to the *optimal classifier* G( x ) = log\[ P( Y = 1 | X = x ) / P( Y = -1 | X = x ) \] / 2, where *optimal* here is taken with respect to the **exponential loss** function. More specifically, Adaboost.M1 is using an additive model to approximate that function. In other words, Boosting is attempting to find functions *f*<sub>1</sub>, *f*<sub>2</sub>, ..., *f*<sub>*N*</sub> such that *G*(*x*)=∑<sub>*i*</sub>*f*<sub>*i*</sub>(*x*<sup>(*i*)</sup>), where *x*<sup>(*i*)</sup> is a sub-vector of *x* (i.e. the function *f*<sub>*i*</sub> only depends on *some* of the available features, typically a few of them: 1 or 2, say). Note that each *f*<sub>*i*</sub> generally depends on a different subset of features than the other *f*<sub>*j*</sub>'s.

Knowing the function the boosting algorithm is approximating (even if it does it in a greedy and suboptimal way), allows us to understand when the algorithm is expected to work well, and also when it may not work well. In particular, it provides one way to choose the complexity of the *weak lerners* used to construct the ensemble. For an example you can refer to the corresponding lab activity.

### A more challenging example, the `email spam` data

The email spam data set is a relatively classic data set containing 57 features (potentially explanatory variables) measured on 4601 email messages. The goal is to predict whether an email is *spam* or not. The 57 features are a mix of continuous and discrete variables. More information can be found at <https://archive.ics.uci.edu/ml/datasets/spambase>.

We first load the data and randomly separate it into a training and a test set. A more thorough analysis would be to use *full* K-fold cross-validation, but given the computational complexity, I decided to leave the rest of this 3-fold CV exercise to the reader.

``` r
data(spam, package='ElemStatLearn')
n <- nrow(spam)
set.seed(987)
ii <- sample(n, floor(n/3))
spam.te <- spam[ii, ]
spam.tr <- spam[-ii, ]
```

We now use Adaboost with 500 iterations, using *stumps* (1-split trees) as our weak learners / classifiers, and check the performance on the test set:

``` r
library(adabag)
onesplit <- rpart.control(cp=-1, maxdepth=1, minsplit=0, xval=0)
bo1 <- boosting(spam ~ ., data=spam.tr, boos=FALSE, mfinal=500, control=onesplit)
pr1 <- predict(bo1, newdata=spam.te)
table(spam.te$spam, pr1$class) # (pr1$confusion)
```

    ##        
    ##         email spam
    ##   email   879   39
    ##   spam     55  560

The classification error rate on the test set is rather high (0.0613177). We now compare it with that of a Random Forest:

``` r
library(randomForest)
set.seed(123) 
a <- randomForest(spam ~ . , data=spam.tr) # , ntree=500)
```

We look at the output

``` r
a
```

    ## 
    ## Call:
    ##  randomForest(formula = spam ~ ., data = spam.tr) 
    ##                Type of random forest: classification
    ##                      Number of trees: 500
    ## No. of variables tried at each split: 7
    ## 
    ##         OOB estimate of  error rate: 5.02%
    ## Confusion matrix:
    ##       email spam class.error
    ## email  1807   63  0.03368984
    ## spam     91 1107  0.07595993

Note that the OOB estimate of the classification error rate is 0.0501956. The number of trees used seems to be appropriate in terms of the stability of the OOB error rate estimate:

``` r
plot(a)
```

![](README_files/figure-markdown_github/spam.plot.rf-1.png)

Now use the test set to estimate the error rate of the Random Forest (for a fair comparison with the one computed with boosting) and obtain

``` r
pr.rf <- predict(a, newdata=spam.te, type='response')
table(spam.te$spam, pr.rf)
```

    ##        pr.rf
    ##         email spam
    ##   email   886   32
    ##   spam     55  560

The performance of Random Forests on this test set is better than that of boosting (recall that the estimated classification error rate for 1-split trees-based Adaboost was 0.0613177, while for the Random Forest is 0.0567515 on the test set and 0.0501956 using OOB).

Is there *any room for improvement* for Adaboost? As we discussed in class, depending on the interactions that may be present in the *true classification function*, we might be able to improve our boosting classifier by slightly increasing the complexity of our base ensemble members. Here we try to use 3-split classification trees, instead of the 1-split ones used above:

``` r
threesplits <- rpart.control(cp=-1, maxdepth=3, minsplit=0, xval=0)
bo3 <- boosting(spam ~ ., data=spam.tr, boos=FALSE, mfinal=500, control=threesplits)
pr3 <- predict(bo3, newdata=spam.te)
(pr3$confusion)
```

    ##                Observed Class
    ## Predicted Class email spam
    ##           email   886   36
    ##           spam     32  579

The number of element on the boosting ensemble appears to be appropriate:

``` r
plot(errorevol(bo3, newdata=spam.te))
```

![](README_files/figure-markdown_github/spam.5-1.png)

There is, in fact, a noticeable improvement in performance on this test set. The estimated classification error rate of AdaBoost using 3-split trees on this test set is 0.0443575. Recall that the estimated classification error rate for the Random Forest was 0.0567515 (or 0.0501956 using OOB).

As mentioned above you are strongly encouraged to finish this analysis by doing a complete K-fold CV analysis in order to compare boosting with random forests on these data.

Gradient boosting
-----------------

Discussed in class.

Neural Networks
---------------

Discussed in class.

### An example with a simple neural network

This example using the ISOLET data illustrates the use of simple neural networks, and also highlights some issues of which it may be important to be aware.

We will use the ISOLET data again. The data set is available here: <http://archive.ics.uci.edu/ml/datasets/ISOLET>, along with more information about it. It contains data on sound recordings of 150 speakers saying each letter of the alphabet (twice). See the original source for more details. The data set is rather large and available in compressed form. Here we will read it from a private copy in plain text form available on Dropbox.

#### "C" and "Z"

First we look at building a classifier to identify the letters C and Z. This is the simplest scenario and it will help us fix ideas. We now read the full data set, and extract the training and test rows corresponding to those two letters:

``` r
library(nnet)
xx.tr <- read.table('https://www.dropbox.com/s/b0avl1w6pcevfc5/isolet-train.data?dl=1', sep=',')
xx.te <- read.table('https://www.dropbox.com/s/lrdrj6u399to1h6/isolet-test.data?dl=1', sep=',')
lets <- c(3, 26)
LETTERS[lets]
```

    ## [1] "C" "Z"

``` r
# Training set
x.tr <- xx.tr[ xx.tr$V618 %in% lets, ]
x.tr$V618 <- as.factor(x.tr$V618)
# Test set
x.te <- xx.te[ xx.te$V618 %in% lets, ]
truth <- x.te$V618 <- as.factor(x.te$V618)
```

We train a NN with a single hidden layer, and a single unit in the hidden layer.

``` r
set.seed(123)
a1 <- nnet(V618 ~ ., data=x.tr, size=1, decay=0, maxit=1500, MaxNWts=2000)
```

    ## # weights:  620
    ## initial  value 350.425020 
    ## iter  10 value 41.176789
    ## iter  20 value 18.095256
    ## iter  30 value 18.052107
    ## iter  40 value 18.050646
    ## iter  50 value 18.050036
    ## iter  60 value 18.048042
    ## iter  70 value 12.957465
    ## iter  80 value 6.911630
    ## iter  90 value 6.483383
    ## iter 100 value 6.482835
    ## iter 110 value 6.482692
    ## iter 120 value 6.482643
    ## iter 130 value 6.482535
    ## iter 140 value 6.481985
    ## iter 150 value 0.945104
    ## iter 160 value 0.016044
    ## final  value 0.000089 
    ## converged

Note the slow convergence. The final value of the objective value was:

``` r
a1$value
```

    ## [1] 8.868116e-05

The error rate on the training set ("goodness of fit") is

``` r
b1 <- predict(a1, type='class') #, type='raw')
mean(b1 != x.tr$V618)
```

    ## [1] 0

We see that this NN fits the training set perfectly. Is this desirable?

We now run the algorithm again, with a different starting point.

``` r
set.seed(456)
a2 <- nnet(V618 ~ ., data=x.tr, size=1, decay=0, maxit=1500, MaxNWts=2000)
```

    ## # weights:  620
    ## initial  value 336.934868 
    ## iter  10 value 157.630462
    ## iter  20 value 61.525474
    ## iter  30 value 48.367804
    ## iter  40 value 43.517664
    ## iter  50 value 36.639732
    ## iter  60 value 36.478617
    ## iter  70 value 35.625710
    ## iter  80 value 26.860150
    ## iter  90 value 26.859935
    ## iter 100 value 22.644407
    ## iter 110 value 22.622998
    ## iter 120 value 22.622435
    ## iter 130 value 19.205152
    ## iter 140 value 13.031944
    ## iter 150 value 11.588969
    ## iter 160 value 11.583343
    ## final  value 11.583318 
    ## converged

Compare the attained value of the objective and the error rate on the training set with those above (8.910^{-5} and 0, respectively):

``` r
a2$value
```

    ## [1] 11.58332

``` r
b2 <- predict(a2, type='class') #, type='raw')
mean(b2 != x.tr$V618)
```

    ## [1] 0.004166667

So, we see that the second run of NN produces a much worse solution. How are their performances on the test set?

``` r
b1 <- predict(a1, newdata=x.te, type='class') #, type='raw')
mean(b1 != x.te$V618)
```

    ## [1] 0.03333333

``` r
b2 <- predict(a2, newdata=x.te, type='class') #, type='raw')
mean(b2 != x.te$V618)
```

    ## [1] 0.025

The second (worse) solution performs better on the test set.

What if we add more units to the hidden layer?

``` r
set.seed(123)
a1 <- nnet(V618 ~ ., data=x.tr, size=3, decay=0, maxit=1500, MaxNWts=2000, trace=FALSE)
set.seed(456)
a2 <- nnet(V618 ~ ., data=x.tr, size=3, decay=0, maxit=1500, MaxNWts=2000, trace=FALSE)
```

The objective functions are 6.4827381 and 9.052401810^{-5}, and their performance on the training and test sets are:

``` r
b1 <- predict(a1, type='class') #, type='raw')
mean(b1 != x.tr$V618)
```

    ## [1] 0.002083333

``` r
b2 <- predict(a2, type='class') #, type='raw')
mean(b2 != x.tr$V618)
```

    ## [1] 0

``` r
b1 <- predict(a1, newdata=x.te, type='class') #, type='raw')
mean(b1 != x.te$V618)
```

    ## [1] 0.03333333

``` r
b2 <- predict(a2, newdata=x.te, type='class') #, type='raw')
mean(b2 != x.te$V618)
```

    ## [1] 0.04166667

What if we add a decaying factor?

``` r
set.seed(123)
a1 <- nnet(V618 ~ ., data=x.tr, size=3, decay=0.05, maxit=500, MaxNWts=2000, trace=FALSE)
a1$value
```

    ## [1] 5.345279

``` r
set.seed(456)
a2 <- nnet(V618 ~ ., data=x.tr, size=3, decay=0.05, maxit=500, MaxNWts=2000, trace=FALSE)
a2$value
```

    ## [1] 5.345279

The two solutions are the same. How does it do on the training and test sets?

``` r
b1 <- predict(a1, type='class') #, type='raw')
mean(b1 != x.tr$V618)
```

    ## [1] 0

``` r
b1 <- predict(a1, newdata=x.te, type='class') #, type='raw')
mean(b1 != x.te$V618)
```

    ## [1] 0.008333333

Add more weights.

``` r
set.seed(123)
a1 <- nnet(V618 ~ ., data=x.tr, size=6, decay=0.05, maxit=500, MaxNWts=4000, trace=FALSE)
a1$value
```

    ## [1] 4.777806

``` r
set.seed(456)
a2 <- nnet(V618 ~ ., data=x.tr, size=6, decay=0.05, maxit=500, MaxNWts=4000, trace=FALSE)
a2$value
```

    ## [1] 4.172023

``` r
b1 <- predict(a1, type='class') #, type='raw')
mean(b1 != x.tr$V618)
```

    ## [1] 0

``` r
b2 <- predict(a2, type='class') #, type='raw')
mean(b2 != x.tr$V618)
```

    ## [1] 0

``` r
b1 <- predict(a1, newdata=x.te, type='class') #, type='raw')
mean(b1 != x.te$V618)
```

    ## [1] 0.008333333

``` r
b2 <- predict(a2, newdata=x.te, type='class') #, type='raw')
mean(b2 != x.te$V618)
```

    ## [1] 0.008333333

#### More letters

``` r
lets <- c(3, 7, 9, 26)
LETTERS[lets]
```

    ## [1] "C" "G" "I" "Z"

``` r
x.tr <- xx.tr[ xx.tr$V618 %in% lets, ]
x.tr$V618 <- as.factor(x.tr$V618)

# testing set
x.te <- xx.te[ xx.te$V618 %in% lets, ]
truth <- x.te$V618 <- as.factor(x.te$V618)


set.seed(123)
a1 <- nnet(V618 ~ ., data=x.tr, size=1, decay=0, maxit=1500, MaxNWts=2000, trace=FALSE)
a1$value
```

    ## [1] 9.785652e-05

``` r
length(a1$wts)
```

    ## [1] 626

``` r
set.seed(456)
a2 <- nnet(V618 ~ ., data=x.tr, size=1, decay=0, maxit=1500, MaxNWts=2000, trace=FALSE)
a2$value
```

    ## [1] 789.9009

``` r
length(a2$wts)
```

    ## [1] 626

``` r
b1 <- predict(a1, type='class') #, type='raw')
mean(b1 != x.tr$V618)
```

    ## [1] 0

``` r
b2 <- predict(a2, type='class') #, type='raw')
mean(b2 != x.tr$V618)
```

    ## [1] 0.4875

``` r
b1 <- predict(a1, newdata=x.te, type='class') #, type='raw')
mean(b1 != x.te$V618)
```

    ## [1] 0.4583333

``` r
b2 <- predict(a2, newdata=x.te, type='class') #, type='raw')
mean(b2 != x.te$V618)
```

    ## [1] 0.4875

``` r
set.seed(123)
a1 <- nnet(V618 ~ ., data=x.tr, size=3, decay=0, maxit=1500, MaxNWts=2000, trace=FALSE)

set.seed(456)
a2 <- nnet(V618 ~ ., data=x.tr, size=3, decay=0, maxit=1500, MaxNWts=2000, trace=FALSE)

b1 <- predict(a1, type='class') #, type='raw')
mean(b1 != x.tr$V618)
```

    ## [1] 0.005208333

``` r
b2 <- predict(a2, type='class') #, type='raw')
mean(b2 != x.tr$V618)
```

    ## [1] 0.00625

``` r
b1 <- predict(a1, newdata=x.te, type='class') #, type='raw')
mean(b1 != x.te$V618)
```

    ## [1] 0.05416667

``` r
b2 <- predict(a2, newdata=x.te, type='class') #, type='raw')
mean(b2 != x.te$V618)
```

    ## [1] 0.0375

``` r
set.seed(123)
a1 <- nnet(V618 ~ ., data=x.tr, size=3, decay=0.05, maxit=500, MaxNWts=2000, trace=FALSE)

set.seed(456)
a2 <- nnet(V618 ~ ., data=x.tr, size=3, decay=0.05, maxit=500, MaxNWts=2000, trace=FALSE)

b1 <- predict(a1, type='class') #, type='raw')
mean(b1 != x.tr$V618)
```

    ## [1] 0

``` r
b2 <- predict(a2, type='class') #, type='raw')
mean(b2 != x.tr$V618)
```

    ## [1] 0

``` r
b1 <- predict(a1, newdata=x.te, type='class') #, type='raw')
mean(b1 != x.te$V618)
```

    ## [1] 0.01666667

``` r
b2 <- predict(a2, newdata=x.te, type='class') #, type='raw')
mean(b2 != x.te$V618)
```

    ## [1] 0.01666667

``` r
set.seed(123)
a1 <- nnet(V618 ~ ., data=x.tr, size=6, decay=0.05, maxit=500, MaxNWts=4000, trace=FALSE)

set.seed(456)
a2 <- nnet(V618 ~ ., data=x.tr, size=6, decay=0.05, maxit=500, MaxNWts=4000, trace=FALSE)

b1 <- predict(a1, type='class') #, type='raw')
mean(b1 != x.tr$V618)
```

    ## [1] 0

``` r
b2 <- predict(a2, type='class') #, type='raw')
mean(b2 != x.tr$V618)
```

    ## [1] 0

``` r
b1 <- predict(a1, newdata=x.te, type='class') #, type='raw')
mean(b1 != x.te$V618)
```

    ## [1] 0.0125

``` r
b2 <- predict(a2, newdata=x.te, type='class') #, type='raw')
mean(b2 != x.te$V618)
```

    ## [1] 0.0125

#### Even more letters

``` r
lets <- c(3, 5, 7, 9, 12, 13, 26)
LETTERS[lets]
```

    ## [1] "C" "E" "G" "I" "L" "M" "Z"

``` r
x.tr <- xx.tr[ xx.tr$V618 %in% lets, ]
x.tr$V618 <- as.factor(x.tr$V618)

# testing set
x.te <- xx.te[ xx.te$V618 %in% lets, ]
truth <- x.te$V618 <- as.factor(x.te$V618)

set.seed(123)
a1 <- nnet(V618 ~ ., data=x.tr, size=4, decay=0, maxit=1500, MaxNWts=3000, trace=FALSE)

set.seed(456)
a2 <- nnet(V618 ~ ., data=x.tr, size=4, decay=0, maxit=1500, MaxNWts=3000, trace=FALSE)

b1 <- predict(a1, type='class') #, type='raw')
mean(b1 != x.tr$V618)
```

    ## [1] 0.1386905

``` r
b2 <- predict(a2, type='class') #, type='raw')
mean(b2 != x.tr$V618)
```

    ## [1] 0.003571429

``` r
b1 <- predict(a1, newdata=x.te, type='class') #, type='raw')
mean(b1 != x.te$V618)
```

    ## [1] 0.1742243

``` r
b2 <- predict(a2, newdata=x.te, type='class') #, type='raw')
mean(b2 != x.te$V618)
```

    ## [1] 0.05727924

``` r
set.seed(123)
a1 <- nnet(V618 ~ ., data=x.tr, size=3, decay=0.3, maxit=1500, MaxNWts=2000, trace=FALSE)

set.seed(456)
a2 <- nnet(V618 ~ ., data=x.tr, size=3, decay=0.3, maxit=1500, MaxNWts=2000, trace=FALSE)

b1 <- predict(a1, type='class') #, type='raw')
mean(b1 != x.tr$V618)
```

    ## [1] 0

``` r
b2 <- predict(a2, type='class') #, type='raw')
mean(b2 != x.tr$V618)
```

    ## [1] 0

``` r
b1 <- predict(a1, newdata=x.te, type='class') #, type='raw')
mean(b1 != x.te$V618)
```

    ## [1] 0.02625298

``` r
b2 <- predict(a2, newdata=x.te, type='class') #, type='raw')
mean(b2 != x.te$V618)
```

    ## [1] 0.02625298

#### Additional resources for discussion (refer to the lecture for context)

-   <https://arxiv.org/abs/1412.6572>
-   <https://arxiv.org/abs/1312.6199>
-   <https://www.axios.com/ai-pioneer-advocates-starting-over-2485537027.html>
-   <https://medium.com/intuitionmachine/the-deeply-suspicious-nature-of-backpropagation-9bed5e2b085e>