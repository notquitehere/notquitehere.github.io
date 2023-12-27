---
title: "Machine Learning Notes: Model Selection and Evaluation"
date: 2019-06-14
modified: 2019-07-04
tags:
category: machine-learning
published: true
---

In order to effectively evaluate our models we need something to evaluate them with. To achieve this we split our data into three groups: Training, Validation and Evaluation (sometimes known as Test). How we go about splitting our data will depend a lot on how much data we have. There are two techniques used for splitting data [holdout](#holdout) and [cross-validation](#cross-validation). The former is only useful for large datasets as it removes a fairly sizeable chunk of the data. On smaller datasets this results in there being insufficient data in each group resulting in loss of trend and generalisation as well as an increased danger of overfitting. These issues can be counteracted to an extent using cross-validation.

## Principle

We have a set of models $\big\\{ M_i \big\\}^K_{k=1}$ which we wish to evaluate using a loss function $L$.

Where `{}` indicate a set, in this instance the set of models we have available to us. This is a set which will consist of between $1$ and $K$ models (since if we have 0 models we don't have anything to test).

### Data partitioning

Our dataset $D$ shall be assumed to be split into the following subsets:

- Training set $T$
- Validation set $V$
- Test or Evaluation set $E$

### Parameters

There are two sets of parameters we need to consider in relation to our models

- the learnable parameters which we gain from the data itself
- the hyper-parameters such as basis, regularisation scheme etc which we do not learn directly from the data

### Process

- establish the learnable parameters using the training set $T$
- use the validation set to establish the set of hyper-parameters which best allows the model to generalise to $V$
- estimate the best performing model using $E$ to test how well the model performs on unseen data. This guards against overfitting of the hyperparameters to the validation set by providing a test of generalisation against unseen data[^1]

[^1]: You need to be careful with this as it's actually possible to overfit your model to your validation data which stops it being generalisable to a larger problem. This is an issue sometimes encountered by those entering [Kaggle](https://www.kaggle.com/) competitions.

## Holdout {#holdout}

If we have a very large dataset ($D$) we can use a holdout scheme whereby we split the data into three groups:

- Training set $T$ randomly sampled from $D$
- Validation set $V$ randomly sampled from $D - T$
- Evaluation set $E = D - T - V$

### Pseudocode algorithm

**Input:** Models $\big\\{ M_i \big\\}_i$ (`model_set`)<br>
**Input:** Dataset $D$ split into training ($T$), evaluation ($E$) and validation ($V$) sets <br>
**Result:** Identification of model $M*$ with the best predictive power

```
for each model in model_set:
    train m using T
    compute model loss on training set
    compute model loss on validation set

select model M* with best overall performance on T and V

Compute loss on E to determine final model performance
```

Prerequisites:

- $D$ must be large enough that splitting it doesn't reduce the training set $T$ to such a point that it is no longer representative of the data's structure.
- The partitioning must be performed such that each of the subsets $\big\\{ T, V, E \big\\}$ is representative of the full dataset.

## Cross-Validation {#cross-validation}

For this method we're going to initially split the data into two sets:

- cross-validation $V$ drawn randomly from $D$
- test/evaluation $E = D - V$

Once this has been done we need to sub-divide $V$ into $K$ 'folds' (groups) $\big\\{v_k\big\\}^K_{k=1}$[^3] such that $V = \cup^K_{k=1}v_k$[^4].

Now we are in a position to train and validate our models.

For each fold we train on $V - v_k$ and validate using $v_k$. So if $K=3$:

Train: $V-v_1$<br>
Validate: $v_1$

Train: $V-v_2$<br>
Validate: $v_2$

Train: $V-v_3$<br>
Validate: $v_3$

After this iteration we pick the model $M*$ with the best overall performance and test it on $E$.

### Pseudocode algorithm

**Input:** Set of models $\big\\{M_i\big\\}_i$ (`model_set`) <br>
**Input:** Dataset $D$ split into cross-validation $V$ and evaluation $E$ sets <br>
**Input:** Number of folds $K$ <br>
**Result:** Identification of model $M*$ with the best predictive power

```
divide V into K folds without losing any data in the process

for m in model_set:
    for k in range (1, K):
        train m using training set C - c_k
        compute model loss on training set
        computer model loss on validation set c_k

        select model M* with best overall performance on T and V

        Compute loss on E to determine final model performance
```

Common folding techniques:

- 10 fold cross-validation (train on 90%, validate on 10%)
- hold 1 out which is equivalent to $N$ fold cross-validation. For $N$ datapoints train on $N-1$ validate on $1$. This is a common strategy when the data is **very** small.

[^3]: For each integer between $1$ and $K$ we will have a sub-set of $V$ designated $v_k$. So if $K = 3$ we will get $\big\\{v_1\big\\}$, $\big\\{v_2\big\\}$, $\big\\{v_3\big\\}$. See also: [Notes on Notation](/notation)
[^4]: The original set $V$ is equal to the union $\cup$ of all the subsets between $1$ and $K$: you haven't lost any data in the transition.
