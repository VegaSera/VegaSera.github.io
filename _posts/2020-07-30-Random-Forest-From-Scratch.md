---
layout: post
title: Creating a Random Forest model from Scratch
subtitle: 
---


# Overview
## Decision Trees
In order to understand what a Random Forest is, and what it does, one must first understand Decision trees. At its most fundamental level, you can think of a decision tree as a "flowchart for predictions". The tree takes in data in the form of an observation with features, and compares this observation's values to its specified paths. Take a look at this image showing a decision tree fit on the Titanic dataset and we'll walk through the logical steps a decision tree model would take during prediction.

![Decision Tree](https://miro.medium.com/max/781/1*fGX0_gacojVa6-njlCrWZw.png)

Let's first create some data to pass through the model. Let's say we have a 25 year old female who rode the Titanic in 2nd class.
* First, we start at the root node, the top most node. Since sex is encoded as 0 for male, 1 for female, we say that 1 <= 0.5 is False, so we take the right path.
* Secondly, we arrive at a node that asks us about the passenger class. 2 <= 2.5 is True, so we go to the left path.
* This last node doesn't have a comparison, nor does it have any child nodes, so we consider this a **leaf node**. We take the class of this node as our prediction, and we predict our 25 year old woman in 2nd class survives.

So this explains what a decision tree is, and how it predicts. But how does it know where to split the data? What's this "gini" score?

Gini Impurity, along with other impurity metrics like Informational Entropy, categorizes how pure the node is. In the case of Gini Impurity, this value ranges from 0 to 1, with lower values showing higher purity. When we fit the decision tree, it goes through each feature/observation combination and calculates what the impurity of the child nodes would be if it chose to split at that particular point, and it chooses the lowest impurity value to split on.

But as useful as this model is, single decision trees have their flaws. They tend to overfit extremely easily, and as a result (with a couple of exceptions beyond the scope of this post) do not generalize well to unseen data and unseen patterns.



## Random Forests

But all is not lost. We can take things a step further. Instead of one decision tree, we can have many of them that all vote on the final prediction. But you might be wondering, "Hey, that method you described above seems pretty deterministic. Wont a lot of decision trees trained on the same data always return the same result?"

The answer is yes, it would. Let's take a look at what makes Random Forests more than just a bundle of identical trees. Random Forests have the ability to give each tree a somewhat different set of parameters, features, or observations. Each tree having a slightly different set of data makes it able to have slightly different trees.

![Random Forest](https://ucarecdn.com/d1fc3c8a-c1d2-47a2-b078-a7ccad5d9255/)

Now you might be wondering, "If we have a lot of different trees that are giving different results to the same data, isn't that going to introduce a lot of errors into our predictions?"

The answer is yes, absolutely. It's introducing those errors *by design*. The key lies in this statement here: "The average error of a large number of random errors is zero". If you were to take a look at any single decision tree in this forest after it's been given a subset of the data, rather than the whole, you might get some useful prediction out of it, you might not. It would certainly be overfit to the subset of the data it was given. But anything it predicts in error should be drowned out by the rest of the forest. Anything it predicts correctly should add to the chorus of correct predictions and also serve to suppress the errors.


# Constructing this Model


# Implementing this Model yourself

***Before implementing this model yourself, read the Conclusion, Comments, and Considerations section at the end.***

The actual implementation of this model is very similar to SKLearn's own Random Forest Classifier if you have used it. If not, no worries, we'll go through step by step.




# Conclusion, comments, and considerations.

Building this model took a lot more effort than I expected.
