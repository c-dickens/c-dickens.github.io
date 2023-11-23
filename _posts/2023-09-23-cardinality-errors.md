---
layout: distill
title:  Understanding Errors of Cardinality Estimators
date:   2023-09-22 11:03:16
description: Converting from relative error to RSE bounds
tags: maths
categories: sketches

authors:
  - name: Charlie Dickens

bibliography: 2023-09-23-cardinality.bib


# Optionally, you can add a table of contents to your post.
# NOTES:
#   - make sure that TOC names match the actual section names
#     for hyperlinks within the post to work correctly.
#   - we may want to automate TOC generation in the future using
#     jekyll-toc plugin (https://github.com/toshimaru/jekyll-toc).
toc:
  - name: Error Guarantees for Cardinality Estimators
    # if a section has subsections, you can add them as follows:
    # subsections:
    #   - name: Example Child Subsection 1
    #   - name: Example Child Subsection 2
  - name: From Relative Errors to Error Distributions

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .fake-img {
    background: #bbb;
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 0px 4px rgba(0, 0, 0, 0.1);
    margin-bottom: 12px;
  }
  .fake-img p {
    font-family: monospace;
    color: white;
    text-align: left;
    margin: 12px 0;
    text-align: center;
    font-size: 16px;
  }

---

# Error Guarantees for Cardinality Estimators

_A note to on how to derive the error rates for common 
cardinality estimation algorithms when the number of buckets is large
enough to assume Gaussian error_.


Probabilistic approximation algorithms often come with what is called
$$(\epsilon, \delta)$$ guarantees.  
Such a guarantee means that with probability at least 
$$1-\delta$$, the algorithm returns an estimate that is 
accurate within some bounds that are defined by the 
parameter $$\epsilon$$.
The specific type of guarantee may vary from algorithm 
to algorithm; for example, if the quantity we wish to
estimate is $$x$$ and the returned value is $$\hat{x}$$,
then we may seek estimates of the form 
$$
(1-\epsilon) x \le \hat{x} \le (1+\epsilon)x.
$$
Alternatively, other types of algorithm (e.g. a 
CountMin sketch) returns estimates of the form:
$$
x \le \hat{x} \le (1+\epsilon)x.
$$

However, _cardinality estimation_ algorithms
typically come with a 
guarantee on the _standard error_ or the _variance_ of 
the estimator.
These guarantees are related and it often takes me an 
annoying amount of time to translate between the two.
This is especially frustrating because when I think of 
common unique counting algorithms 
(such as the $$k$$-minimum values) algorithm, there is an
associated guarantee of correctness for that algorithm.
However, that guarantee is not strictly the same as the 
one that is given in common libraries such as [Apache 
DataSketches error tables](https://datasketches.apache.org/docs/Theta/ThetaErrorTable.html).
So how to map between the two?
The answer is failry simple when the number of buckets in the sketch is large 
(nb. if this is not the case then the error distribution gets 
[truncated at zero and is not Gaussian](https://datasketches.apache.org/docs/HLL/HLL.html)).

## From Relative Errors to Error Distributions
For an input dataset, the cardinality is the number of distinct items in the dataset without multiplicities.
Suppose that a cardinality estimation algorithm returns 
an estimate $$\hat{n}$$ for a (multi)set of cardinality $$n$$
with standard error $$\sqrt{c} n / \sqrt{k}$$.
The constant $$c$$ is algorithm dependent and the 
parameter $$k$$ is the number of retained samples 
(or buckets in the sketch).
This means that the variance (another way of writing 
the guarantees) of the estimator is $$c n^2 / k$$ 
and the relative standard error is $$\sqrt{c} / \sqrt{k}$$.
The **relative standard error** corresponds to the **relative error** that 
we typically see in algorithms papers; hence we can
set $$\epsilon = \sqrt{c} / \sqrt{k}$$ so that the number
of samples or buckets is related to error as 
$$k = c / \epsilon^2.$$
This format is much closer to what you'll see in an
algorithms paper rather than a statistical paper.
It is also easy to get a simple bound on the total space 
usage of the algorithm by multiplying through the size 
of the hash domain that is used in the algorithm to 
get $$\textsf{space} = c \log(U) \epsilon^{-2}.$$


Now, how do we use this? 
Practitioners often want to know how to set the size of 
the cardinality estimator so that they have confidence
interval for the parameter being estimated.
To do so, set the failure probability of the algorithm to
be $$\delta = 0.05$$ which means that we will roughly 
obtain a $$95\%$$ confidence interval.
When the number of samples is large enough, the error 
distribution is (approximately) Gaussian so the associated $$z$$-score is 
$$1.96$$ which we use to adjust the number of buckets as
$$
k = \frac{c \cdot 1.96^2}{\epsilon^2}.
$$ 
If you want different confidence intervals, then you 
adjust the rescaling factor from the $$z$$-score accordingly.

This expression will give the commonly used 
error bounds for standard sketches.
Improved bounds can be given with more work, but once you
begin to merge sketches, things tend to revert to the 
setting given above.
All of this is done assuming that the number of buckets in the sketch
is sufficiently large to assume Gaussian errors.
If the number of buckets is small, then the distribution gets truncated
at zero and this squashes the distribution so the analysis above does 
not apply.

