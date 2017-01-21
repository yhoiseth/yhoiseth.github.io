---
layout: post
title: "Notes on Intro to Machine Learning (Udacity)"
categories: machine-learning python scikit-learn probability bayes
---

I'm currently enrolled in Udacity's [Intro to Machine Learning](https://www.udacity.com/course/intro-to-machine-learning--ud120). These are my notes. They are a work in progress, as I have yet to finish the course. The notes are not meant to be a complete summary.

## Naive Bayes

### [Machine Learning for Author ID](https://classroom.udacity.com/courses/ud120/lessons/2254358555/concepts/30109586140923)

> One particular feature of Naive Bayes is that it’s a good algorithm for working with text classification. When dealing with text, it’s very common to treat each unique word as a feature, and since the typical person’s vocabulary is many thousands of words, this makes for a large number of features.

```
no. of Chris training emails: 7936
no. of Sara training emails: 7884
Training time: 2.0 seconds
Prediction time: 0.0 seconds
Accuracy: 0.973265073948
```

Pretty good results there, I think.

## Support Vector Machines

Draws a line to separate two classes of data. The best line is the one with the greatest distance to the nearest observation. This is called the _margin_. The greater the margin, the more robust the algorithm is to classification errors.

A third feature

_z = f(x, y)_ 

can be used to enable linear separation where none was possible. 

For example _z = |x|_.

```
no. of Chris training emails: 7936
no. of Sara training emails: 7884
Training time: 169.0 seconds
Prediction time: 19.0 seconds
Accuracy: 0.984072810011
```

Slightly more accurate, but a lot slower that Naive Bayes.

Sliced out 99 % of the datasets:

```
Training time: 0.0 seconds
Prediction time: 1.0 seconds
Accuracy: 0.884527872582
```

`rbf` kernel instead of `linear`:

```Accuracy: 0.616040955631```

## Decision Trees

- Prone to overfitting

## Choose Your Own Algorithm

### K Nearest Neighbor

This one is quite simple. To predict the label of a feature, it chooses the most frequent of the _k_ nearest training features in the feature space.

With `weights='uniform'` (default):

```
Training time: 0.0 seconds
Prediction time: 0.0 seconds
Accuracy: 0.92
```

With `weights='distance'`:

```
Training time: 0.0 seconds
Prediction time: 0.0 seconds
Accuracy: 0.932
```

### AdaBoost

AdaBoost creates a strong classifier from a number of weak classifiers.

```
Training time: 0.0 seconds
Prediction time: 0.0 seconds
Accuracy: 0.924
```

### Random Forest

For classification, the Random Forest algorithm outputs the mode of several decision trees. Good to prevent overfitting.

```
Training time: 0.0 seconds
Prediction time: 0.0 seconds
Accuracy: 0.932
```






































