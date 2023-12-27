---
title: "Math Notes: The Types and Uses of Variables"
date: 2019-07-04
modified: 2019-07-12
tags: statistics, machine-learning
category: mathematics
published: true
---

If it can take on different values, it's a variable. They come in a variety of flavours:

- [Independent](#independent)
- [Dependent](#dependent)
- [Controlled](#controlled)
- [Confounding](#confounding)

## Independent Variables {#independent}

These are the variables that represent inputs or causes: potential reasons for variation.

In **experimental design** these are the variables that we manipulate such as group membership, intervention/treatment regime, gender, age[^1].

*Example:*

In [Holding the Hunger Games Hostage at the Gym: An Evaluation of Temptation Bundling](http://www.katherinemilkman.com/journal-articles/holding-the-hunger-games-hostage-at-the-gym-an-evaluation-of-temptation-bundling-pdf) Milkman, Minson and Volpp set up an experiment to assess the effect of temptation bundling on gym attendance, here their independent variable was access to audiobooks of the participants choice. Some they only had access to them at the gym, some had access to them at any time and some weren't given audiobooks at all.

In **machine learning** the independent variables are the 'features' of the dataset.

*Example:*

If we were investigating the [California Housing Prices Dataset](http://www.dcc.fc.up.pt/~ltorgo/Regression/cal_housing.html) the following would be our independent variables:

- longitude and latitude
- housing median age
- total rooms
- total bedrooms
- population: total number of people residing in a block
- households: total number of households (a group of people residing in the same unit) in a block
- ocean proximity

[^1]: while these last two aren't directly under the control of the experimenter it is possible to set your groups up in such a way as to assess the impact that gender and age have on the dependent variables.

## Dependent Variables {#dependent}

These are the thing we're actually studying. Their values are, theoretically[^2], dependent on the values of the inputs or independent variables.

*Examples:*

Milkman et al were studying the impact of reward bundling on gym attendance so their dependent variable was average weekly gym attendance frequencies[^3].

When performing regression on the California Housing Prices dataset this would be median housing price.

[^2]: This may not actually be the case, as there may of course be no relation between the two at all.
[^3]: Clearly this doesn't actually tell you anything about the quality of the time put in, but as that wasn't the focus of the study it doesn't much matter.

## Controlled variables {#controlled}

Control or controlled variables are those which are kept the same throughout the whole experiment, they are potential confounding variables which have been identified and neutralised by the experimenter. Any change to these variables invalidates correlation between the independent and dependent variables.

## Confounding (Extraneous) Variables {#confounding}

A confounding variable is one that might influence the dependent variable and whose effects might explain the outcome if not controlled for. There are three types of confounding variable: situational, participant, and experimenter effect[^1].

### Situational

Aspects of the environment such as light levels, noise etc.

For example in Milkman et al the gym itself becomes a variable, and one that is not easily controlled beyond limiting participants to a single gym.

Widlus and Jones' study [Do Exploratory Arm Movements Contribute to Reach-Ability Judgments?](https://journals.sagepub.com/doi/abs/10.1177/1541931213601827) possibly provides better examples of this. In this study they randomised starting distances and angles so that the participants couldn't use aspects of the environment (walls, marks on the floor etc) as aids to judgement of their ability to reach the object at the focus of the experiment.

### Participant

The ways in which participants vary from each other: experience, mood, anxiety etc.

For Milkman et al, this could include determination to get fit, anxiety associated with public exercise, mood on any given day etc.

While for Widlus and Jones it might include aspects of the participants personal life, for example those who climb are generally better at judging whether or not they can reach something as it is a skill they practice on a regular basis.

### Experimenter effect

The experimenter unconsciously conveying to the participants what the outcomes are that they want, and how they want them to behave. The experimenter may be totally unaware that they are doing this.

While neither of these studies provide a good example of this it **is** something which is commonly seen in surveys, where questions are frequently subtly (or occasionally overtly) leading the respondee towards the desired outcome of the study.

## Sources

McLeod, S. A. (2018, Aug 10). Independent, dependent and extraneous variables. Retrieved from <https://www.simplypsychology.org/variables.html>
Singh and Bajapi (2008) Research Methodology: Techniques & Trends. APH Publishing Corporation, New Delhi.

[^4]: Demand characteristics is sometimes cited as a fourth type of confounding variable however, this is just an amalgamation of the above so **I** don't consider it to be a separate thing.
