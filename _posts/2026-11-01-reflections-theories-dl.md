---
title: "Reflections - The role of scientific theories in the era of deep learning: Sufficiency and necessity"
date: 2026-11-01 00:00:00 -0300
tags: science philosophy ml learning theory classification prediction
header:
  overlay_image: "/assets/posts/2026-11-01-reflections-theories-dl/header.jpg"
  overlay_filter: "0.6"
toc: true
toc_label: "Table of Contents"
toc_icon: "list"
---

# Changelog
**November 1st, 2026**: Publication of the original article
{:.notice--info }

# Introduction
I have been thinking a lot about the notions of **sufficiency** and **necessity** lately. These are two very important concepts in theory development because they encompass key notions of what scientific theories do when it comes to understanding (physical / biological / cognitive / information / learning) phenomena, but the more I think about it, the more it seems that they also play a fundamental role in (data-driven) predictive models, especially classification ones.

# The Aliens
Suppose our classification problem is to tell 'human' from 'not human'. Well, all humans are great apes but not all great apes are human (*homo sapiens*). In other words, to be a great ape is a necesary condition to be a human, and to be a human is sufficient to be a great ape. Similarly, to be a human is a necessary condition to be a woman, and to be a woman is a sufficient condition to be a human. In terms of abstract levels of categorization (taxonomy), we say that 'great ape' is a superordinate level of human, 'human' is the basic level, and 'woman' is a subordinate level of human.

So, what that has to do with classification? Well, let's say that there is an alien race studying life on earth and they came up with a binary classifier that assigns 'human' if the organism is a 'woman', and 'not human' otherwise. In this case, their model uses a low-level feature to make predictions and precision-wise, it's a very good classifier—i.e., everything it says to be a human is indeed a human. That is because to be a woman is a sufficient condition for being a human. However, the aliens will soon find out that many humans are not women, which means that their model will miss a lot (false negatives). So, they try again and this time around their model assigns 'human' if it has all the characteristic features of a 'great ape' and 'not human' otherwise. Now, their model never misses a human—perfect recall in information retrieval lingo—but they soon find out that many great apes are not human, so their model makes tons of false alarms (false positives).

# Understanding
This might seem a silly example (it is!) but I'm using it to make a parallel with current, widely used data-driven predictive algorithms. Specifically, the alien race here might as well be a deep learning (DL) model or any predictive algorithm in which a prediction cannot be easily understood (black box algorithms). In the case of DL models specifically, low-level learned features (distributed representations of subordinate descriptors) will make the model behave as the 'being a woman' example—that is, prone to false negatives—while high-level representations will make the model behave as the 'great ape' example—prone to false positives. I believe the issue is even worse. Specifically, even if we could tell exactly what features the model is using to make a prediction, the lack of theory (the alien) means that we cannot ever know the relationship between features and entities (predicted classes), just like humans meeting an alien race for the first time. We can use our senses and other tools to measure a whole lot of features but without experimental work and theory building, it is all data without understanding.

I think the role of theories is to guide us (and predictive models, too) towards the contexts in which subsets of features are meaningful and how they relate to the entities of interest. Crucially, such an understanding can allow us to anticipate errors in predictive models—for example, to tell when a classifier is more prone to make false negatives than false positives because it relies mostly on low-level representations—as well as instantiate better models. The use of low- and high-level descriptors is, indeed, complementary. Using the human vs. not human classifier example mentioned berfore, one set of features tries to capture what makes us unique (high-level), while the other is concerned with the diversity of the human experience (low-level).

[top](#){:.btn .btn--light-outline .btn--small}
