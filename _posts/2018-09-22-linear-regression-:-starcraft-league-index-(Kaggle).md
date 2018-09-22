---
layout: post
title:  "Linear Regression : Starcraft League Index (Kaggle Dataset)"
image: ''
date:   2018-09-22 00:12:00
tags:
- Linear Regression
- Python
- Kaggle Dataset
description: 'Reviewing Linear Regression for Practice. Using Kaggle Public Dataset : Skillcraft'
categories:
- Linear Regression
---

I've made a full kernel on [Kaggle](www.kaggle.com). Everything is summarized and explained in the kernel, but I'm just gonna succinctly go over it on this post as well. So if you'd like to see the full report, I recommend you to go to the link below. Additionally, I'm a newbie trying out data analysis, so don't get your expectation too high. '^'

**[Starcraft League Index Linear Regression on Kaggle](https://www.kaggle.com/chungjinwoo5d/linear-reg-w-skillcraft-data-novice-attempt)**

---

## Abstract

The final model (equation) was :

{% highlight text %}
LeagueIndex = 0.0212(Age) + 0.0099(HoursPerWeek) + 31.3489(SelectByHotkeys) + 874.1684(AssignToHotkeys) + 0.0297(UniqueHotkeys) + 1050.7058(MinimapAttacks) + 692.5465(NumberOfPACs) - 0.0104(GapBetweenPACs) + 0.1599(ActionsInPAC) - 0.0069(TotalMapExplored) + 185.1212(WorkersMade)
{% endhighlight %}

While the final model showed decent result in terms of the OLS summary in python, I found a huge problem when I was doing Residual Analysis. **The LeagueIndex (Y ; Dependent Variable) is a discrete/categorical type**. Literal facepalm moment. So basically this means that linear regression isn't the right tool to use! If I wanted to stick with regression, I should do a logistic regression or I could do a more advanced approach and apply machine learning.

<span style="color:red">**Lesson Learnt (the hard way) : ALWAYS understand data before using any analysis tool!**</span>

I'm thinking of actually doing a simple Logistic Regression with R, so will post it up later when I've done it ;)

---

## Final Thoughts

I've learnt linear regression in a very short time (in R) and don't know much about it. I wanted to try regression with Python and overall it wasn't as good as R. So, I think I will stick with R on regressions and simple machine learning. Overall great experience. 
