---
layout:     post
title:      Information Value and Population Stability Index
date:       2016-07-20
summary:    They're the same thing. Promise.
categories: [credit-risk]
---

The way credit risk models are framed rarely changes. They are commonly used to decide whether to accept or reject a given application for credit (a home loan, car loan, business loan, trade credit, etc.). In this case, the training set is some collection of past applications which are split into two groups: those made by customers that subsequently paid back their debt and those that didn't. (There is the small question of where to put the applications from the same time period that were rejected, but we're going to ignore it for now.) 

This is a binary outcome, and most credit risk models are simply logistic regression models that predict this outcome. 

The setup for credit risk models rarely changes. The  

Credit risk modelling is an interesting corner of statistics that has its own toolbox and language to go with it. For example, the Information Value (or Value of Information) is a statistic that is used to get a rough idea of how predictive a single feature in isolation is. For most credit risk models, the outcome is binary (did this customer pay back their debt, or did they default?) and the features are discrete (continuous features such as age or loan amount are discretised, e.g. instead of measuring loan amount in raw dollars, it might be broken up into the three buckets [0, 2000), [2000, 5000), [5000, ∞) , )

