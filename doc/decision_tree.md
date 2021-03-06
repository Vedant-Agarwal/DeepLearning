<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Gradient Boosting](#gradient-boosting)
  - [Additive model for residual](#additive-model-for-residual)
  - [Gradient decent](#gradient-decent)
  - [Loss functions](#loss-functions)
  - [Final algorithm](#final-algorithm)
  - [Summary](#summary)
- [Decision Tree](#decision-tree)
  - [Choosing the attributes](#choosing-the-attributes)
  - [Problems](#problems)
  - [Basic algorithm for learning decision trees](#basic-algorithm-for-learning-decision-trees)
  - [The ID3 algorithm](#the-id3-algorithm)
- [Gradient Boosted Tree](#gradient-boosted-tree)
  - [Tree ensembles Model](#tree-ensembles-model)
    - [Different ensemble ways: bagging, boosting, stacking](#different-ensemble-ways-bagging-boosting-stacking)
      - [Random Forest vs Boosted Tree](#random-forest-vs-boosted-tree)
  - [Gradient Boosting Tree](#gradient-boosting-tree)
    - [learning model](#learning-model)
    - [Residual learning to generate tree](#residual-learning-to-generate-tree)
    - [mathematic formulation for learning process](#mathematic-formulation-for-learning-process)
- [XGboost](#xgboost)
  - [Improvement on gradient loss function calculation](#improvement-on-gradient-loss-function-calculation)
    - [Loss function and Taylor expansion](#loss-function-and-taylor-expansion)
    - [New objective function](#new-objective-function)
  - [New objective function](#new-objective-function-1)
  - [Learning steps: Additive Training](#learning-steps-additive-training)
  - [Parameter selection](#parameter-selection)
  - [Features](#features)
  - [Code Example:](#code-example)
  - [Reason for Xgboost Widely used](#reason-for-xgboost-widely-used)
  - [Xgboost vs Gradient Boost decision Tree (GBDT)](#xgboost-vs-gradient-boost-decision-tree-gbdt)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Gradient Boosting
referring http://www.ccs.neu.edu/home/vip/teach/MLcourse/4_boosting/slides/gradient_boosting.pdf

## Additive model for residual

* This one uses regression as example

  * You are given (x1, y1), (x2, y2), ..., (xn, yn), and the task is to fit a model F(x) to minimize square loss.

  *  Given model F. the model is good but not perfect. There are some mistakes: F(x1) = 0.8, while y1 = 0.9, and You are not allowed to remove anything from F or change any parameter in F.

  * Solution is to add additional model (regression tree) h to F, so the new prediction will be F (x ) + h(x ).

  * Just fit a regression tree h to data
  (x1,y1 −F(x1)),(x2,y2 −F(x2)),...,(xn,yn −F(xn))

* yi − F (xi ) are called residuals. These are the parts that existing model F cannot do well. The role of h is to compensate the shortcoming of existing model F.
If the new model F + h is still not satisfactory, we can add another regression tree...

## Gradient decent
![gradient_loss](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/gradient_loss.png)
So basically we have:

* residual ⇔ negative gradient
* fit h to residual ⇔ fit h to negative gradient
* update F based on residual ⇔ update F based on negative gradient

So we are actually updating our model using gradient descent!

> Algorithm could be

* start with an initial model
* iterate until converge:
  * calculate negative gradients −g(xi)
  * fit a regression tree h to negative gradients −g(xi)
  * F := F + ρh, where ρ = 1

The benefit of formulating this algorithm using gradients is that it allows us to consider other loss functions and derive the corresponding algorithms in the same way.

## Loss functions

1. squared loss

* Easy to deal with mathematically
* Not robust to outliers. Outliers are heavily punished because the error is squared.

Pay too much attention to outliers. Try hard to incorporate outliers into the model. Degrade the overall performance

2. Absolute loss

(more robust to outliers) L(y , F ) = |y − F |

3. Huber loss (more robust to outliers)
```
(y − F )^2 if |y − F | ≤ δ
δ(|y−F|−δ/2) if |y−F|>δ
```
## Final algorithm
![regression_loss](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/regression_loss.png)

## Summary
Summary of the Section
* Fit an additive model F =  t ρt ht in a forward stage-wise manner.
* In each stage, introduce a new regression tree h to compensate the shortcomings of existing model.
* The“shortcomings” are identified by negative gradients.
* For any loss function, we can derive a gradient boostingalgorithm.
* Absolute loss and Huber loss are more robust to outliers than square loss.

# Decision Tree

* Any boolean function can be represented by a decision tree.
* best when a small number of attributes provide a lot of information • Note: finding optimal tree for arbitrary data is NP-hard.


* simple example of inductive learning
1. learn decision tree from training examples
2. predict classes for novel testing examples
* Generalization is how well we do on the testing examples.
* Only works if we can learn the underlying structure of the data.

## Choosing the attributes
* How do we find a decision tree that agrees with the training data?
* Could just choose a tree that has one path to a leaf for each example
  but this just memorizes the observations (assuming data are consistent) - we want it to generalize to new examples
* Ideally, best attribute would partition the data into positive and negative examples

## Problems
* How do we which attribute or value to split on?
* When should we stop splitting?
* What do we do when we can’t achieve perfect classification?
* What if tree is too large? Can we approximate with a smaller tree?

## Basic algorithm for learning decision trees
1. starting with whole training data
2. select attribute or value along dimension that gives “best” split
3. create child nodes based on split
4. recurse on each child using child data until a stopping criterion is reached
  * all examples have same class • amount of data is too small • tree too large
  * Central problem: How do we choose the “best” attribute?

* Choose the attribute which has most information gain(Cross entropy)

## The ID3 algorithm
The calculation for information gain is the most difficult part of this algorithm. ID3 performs a search whereby the search states are decision trees and the operator involves adding a node to an existing tree. It uses information gain to measure the attribute to put in each node, and performs a greedy search using this measure of worth. The algorithm goes as follows:


Given a set of examples, S, categorised in categories ci, then:

1. Choose the root node to be the attribute, A, which scores the highest for information gain relative to S.

2. For each value v that A can possibly take, draw a branch from the node.

3. For each branch from A corresponding to value v, calculate Sv. Then:

    * If Sv is empty, choose the category cdefault which contains the most examples from S, and put this as the leaf node category which ends that branch.

    * If Sv contains only examples from a category c, then put c as the leaf node category which ends that branch.

    * Otherwise, remove A from the set of attributes which can be put into nodes. Then put a new node in the decision tree, where the new attribute being tested in the node is the one which scores highest for information gain relative to Sv (note: not relative to S). This new node starts the cycle again (from 2), with S replaced by Sv in the calculations and the tree gets built iteratively like this.



# Gradient Boosted Tree

## Tree ensembles Model

> Decision tree is about learning a set of rules:

* Advantages:
  * Interpretable
  * Robust
  * Non linear link

* Drawbacks:
  * Weak Learner
  * High variance

> classification and regression trees __(CART)__

The tree ensemble model is a set of classification and regression trees __(CART)__. A CART is a bit different from decision trees, where the leaf only contains decision values. In CART, a __real score__ is associated with each of the leaves, which gives us richer interpretations that go beyond classification. This also makes the unified optimization step easier. Here is example on how ensemble works

![CART_tree](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/CART_tree.jpg)


Usually, a single tree is not strong enough to be used in practice. What is actually used is the so-called tree ensemble model, which sums the prediction of multiple trees together. The prediction scores of each individual tree are summed up to get the final score.

![xgboost_model](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/xgboost_model.jpg)

![xgboost_parameter](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/xgboost_para.jpg)

### Different ensemble ways: bagging, boosting, stacking
![ensemble_ways](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/ensemble_ways.png)


#### Random Forest vs Boosted Tree

The major reason is in terms of training objective, __Boosted Trees(GBM)__ tries to add new trees that compliments the already built ones.  This normally gives you better accuracy with less trees.

* Random forrest decides to create a large number of them based on bagging. The basic idea is to resample the data over and over and for each sample train a new classifier. Different classifiers overfit the data in a different way, and through voting those differences are averaged out.

* GBM is a boosting method, which builds on weak classifiers. The idea is to add a classifier at a time, so that the next classifier is trained to improve the already trained ensemble. Notice that for RF each iteration the classifier is trained independently from the rest

> Optimizing Goal

* Optimizing training loss encourages __predictive models__.Fitting well in training data at least get you close to training data which is hopefully close to the underlying distribution
* Optimizing regularization encourages __simple models__. Simpler models tends to have smaller variance in future predictions, making prediction stable

The compleixty of model/regularization will be
![xgboost_model_complexity](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/xgboost_model_complexity.jpg)


## Gradient Boosting Tree

After introducing the model, let us begin with the real training part. How should we learn the trees? The answer is, as is always for all supervised learning models: define an objective function, and optimize it!

### learning model

![xgboost_loss](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/xgboost_loss.jpg)

### Residual learning to generate tree

![gradient_boosting](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/gradient_boosting.jpg)


### mathematic formulation for learning process

The idea is to train the additive and residual from last tree, same as gradient boost idea
![tree_additive_training](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/tree_additive_training.png)

![tree_additive_training_2](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/tree_additive_training_2.png)


# XGboost

Xgboots is a better and faster implementation for gradient boost tree

## Improvement on gradient loss function calculation

### Loss function and Taylor expansion

![tree_additive_training_3](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/tree_additive_training_3.png)

### New objective function
![tree_additive_training_4](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/tree_additive_training_4.png)

* it only relies on first and second order gradient values.
* This formulation applies to all general loss function. square loss is a example case.

* In the general case, we take the __Taylor expansion__ of the loss function up to the __second order__

![xgboost_loss_optimization](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/xgboost_loss_optimization.jpg)

One important advantage of this definition is that it only depends on gi and hi. This is how xgboost can support custom loss functions. We can optimize every loss function, including logistic regression and weighted logistic regression, using exactly the same solver that takes hi as input!

> In summary: __*XGBoost is to Gradient Boosting what Newton's Method is to Gradient Descent.*__

Here is a detailed link for comparison: https://stats.stackexchange.com/questions/202858/loss-function-approximation-with-taylor-expansion/204377



## New objective function
![xgboost_structure_score](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/xgboost_structure_score.jpg)

## Learning steps: Additive Training

First thing we want to ask is what are the parameters of trees? You can find that what we need to learn are those functions fi, with each containing the structure of the tree and the leaf scores. This is much harder than traditional optimization problem where you can take the gradient and go. It is not easy to train all the trees at once. Instead, we use an __additive strategy__: fix what we have learned, and add one new tree at a time.
![xgboost_tree_grow](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/xgboost_tree_grow.jpg)

And we can see how __Prune__ and __regularization__ works in learning process

![xgboost_regularization](https://github.com/zhangruiskyline/DeepLearning_Intro/blob/master/img/xgboost_regularization.jpg)


## Parameter selection

For feature importance analysis, in Simplicity Vs Accuracy trade-off, choose the first. Few rule of thumbs (empiric):
* nrounds: number of trees. Keep it low (< 20 trees)
* max.depth: deepness of each tree. Keep it low (< 7)
* Run iteratively the feature importance analysis and remove the most important features until the 3 most important features represent less than 70% of the whole gain.

> Here is link for xgboots paramter:

https://xgboost.readthedocs.io/en/latest/parameter.html#parameters-for-tree-booster

> Some important parameters explained here

https://www.slideshare.net/tkm2261/overview-of-tree-algorithms-from-decision-tree-to-xgboost


## Features

* Model features: Three main forms of gradient boosting are supported:

  * Gradient Boosting algorithm also called gradient boosting machine including the learning rate.
  * Stochastic Gradient Boosting with sub-sampling at the row, column and column per split levels.
  * Regularized Gradient Boosting with both L1 and L2 regularization.

XGBoost dominates structured or tabular datasets on classification and regression predictive modeling problems.

* System Features

  The library provides a system for use in a range of computing environments, not least:

  * Parallelization of tree construction using all of your CPU cores during training.
  * Distributed Computing for training very large models using a cluster of machines.
  * Out-of-Core Computing for very large datasets that don’t fit into memory.
  * Cache Optimization of data structures and algorithm to make best use of hardware.

* Algorithm Features

The implementation of the algorithm was engineered for efficiency of compute time and memory resources. A design goal was to make the best use of available resources to train the model. Some key algorithm implementation features include:

  * Sparse Aware implementation with automatic handling of missing data values.
  * Block Structure to support the parallelization of tree construction.
  * Continued Training so that you can further boost an already fitted model on new data.


## Code Example:

* The demo code from official xgboost: https://github.com/dmlc/xgboost/tree/master/demo

* Kaggle xgboost example https://www.kaggle.com/cbrogan/xgboost-example-python/code


## Reason for Xgboost Widely used

* Xgboost is very good at tabular data, deep learning is more for image, audio, text

* That being said, xgboost is good at situations that we have high level features. While deep learning is more for raw features(low level)

* Xgboost can handle missing data and nosie data well

* In theory, ensembling model has less impact on model's BV dimension, so more robust to overfitting(compared with linear/logistic regression, if we add more high order polynomial, eays to increase VC dimension)

* System implementation, flexible , easy to extend

## Xgboost vs Gradient Boost decision Tree (GBDT)
see [zhihu discussion link](https://www.zhihu.com/question/41354392/answer/98658997)

* GBDT uses CART as classifier while Xgboost can use linear classifer, when doing so it becomes regression or logistic regression regression with L1 and L2 regularization

* GBDT uses first order gradient, xgboost uses first and second order

* Xgboost adds regularization: number of leaves, L2 of leaf scores

* Shrinkage: eta in xgboost, reduce current tree impact so that later learning can have room

* Parallelization: Not in tree learning, but in feature ranking. xgboost save pre ranked feature and used in later training
