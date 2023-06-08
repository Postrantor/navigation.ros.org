See discussions, stats, and author profiles for this publication at: [https://www.researchgate.net/publication/319463746](https://www.researchgate.net/publication/319463746_Behavior_Trees_in_Robotics_and_AI_An_Introduction?enrichId=rgreq-02f2f10a6f0883ed7d7ed7c736dac00b-XXX&enrichSource=Y292ZXJQYWdlOzMxOTQ2Mzc0NjtBUzo1ODU5OTMwNDg1NTk2MTZAMTUxNjcyMzAzODE0Mg%3D%3D&el=1_x_2&_esc=publicationCoverPdf)

[Behavior Trees in Robotics and AI: An Introduction](https://www.researchgate.net/publication/319463746_Behavior_Trees_in_Robotics_and_AI_An_Introduction?enrichId=rgreq-02f2f10a6f0883ed7d7ed7c736dac00b-XXX&enrichSource=Y292ZXJQYWdlOzMxOTQ2Mzc0NjtBUzo1ODU5OTMwNDg1NTk2MTZAMTUxNjcyMzAzODE0Mg%3D%3D&el=1_x_3&_esc=publicationCoverPdf)

> 《机器人和人工智能中的行为树：一个介绍》

**Book** · July 2018
DOI: 10.1201/9780429489105

Handling Concurrency in Behavior Trees [View project](https://www.researchgate.net/project/Handling-Concurrency-in-Behavior-Trees?enrichId=rgreq-02f2f10a6f0883ed7d7ed7c736dac00b-XXX&enrichSource=Y292ZXJQYWdlOzMxOTQ2Mzc0NjtBUzo1ODU5OTMwNDg1NTk2MTZAMTUxNjcyMzAzODE0Mg%3D%3D&el=1_x_9&_esc=publicationCoverPdf)

> 处理行为树中的并发性[查看项目](https://www.researchgate.net/project/Handling-Concurrency-in-Behavior-Trees?enrichId=rgreq-02f2f10a6f0883ed7d7ed7c736dac00b-XXX&enrichSource=Y292ZXJQYWdlOzMxOTQ2Mzc0NjtBUzo1ODU5OTMwNDg1NTk2MTZAMTUxNjcyMzAzODE0Mg%3D%3D&el=1_x_9&_esc=publicationCoverPdf)

All content following this page was uploaded by [Michele Colledanchise](https://www.researchgate.net/profile/Michele-Colledanchise?enrichId=rgreq-02f2f10a6f0883ed7d7ed7c736dac00b-XXX&enrichSource=Y292ZXJQYWdlOzMxOTQ2Mzc0NjtBUzo1ODU5OTMwNDg1NTk2MTZAMTUxNjcyMzAzODE0Mg%3D%3D&el=1_x_10&_esc=publicationCoverPdf) on 23 January 2018.

> 所有在此页面后面的内容都是由[Michele Colledanchise](https://www.researchgate.net/profile/Michele-Colledanchise?enrichId=rgreq-02f2f10a6f0883ed7d7ed7c736dac00b-XXX&enrichSource=Y292ZXJQYWdlOzMxOTQ2Mzc0NjtBUzo1ODU5OTMwNDg1NTk2MTZAMTUxNjcyMzAzODE0Mg%3D%3D&el=1_x_10&_esc=publicationCoverPdf)于 2018 年 1 月 23 日上传的。

The user has requested enhancement of the downloaded file.
Michele Colledanchise and Petter O¨ gren
Behavior Trees in Robotics and AI

An Introduction

# Contents

1.  What are Behavior Trees? [3](#_bookmark4)
    1.  A Short History and Motivation of BTs [4](#a-short-history-and-motivation-of-bts)
    2.  What is wrong with FSMs? The Need for Reactiveness and Modularity [5](#what-is-wrong-with-fsms-the-need-for-reactiveness-and-modularity)
2.  Classical Formulation of BTs [6](#classical-formulation-of-bts)
    1.  Execution Example of a BT [9](#execution-example-of-a-bt)
    2.  Control Flow Nodes with Memory [11](#_bookmark22)
3.  Creating a BT for Pac-Man from Scratch [12](#creating-a-bt-for-pac-man-from-scratch)
4.  Creating a BT for a Mobile Manipulator Robot [14](#creating-a-bt-for-a-mobile-manipulator-robot)
5.  Use of BTs in Robotics and AI [15](#use-of-bts-in-robotics-and-ai)
    1.  BTs in autonomous vehicles [16](#bts-in-autonomous-vehicles)
    2.  BTs in industrial robotics [18](#bts-in-industrial-robotics)
    3.  BTs in the Amazon Picking Challenge [20](#bts-in-the-amazon-picking-challenge)
    4.  BTs inside the social robot JIBO [21](#bts-inside-the-social-robot-jibo)

```{=html}
<!-- --
```

2.  How Behavior Trees Generalize and Relate to Earlier Ideas [23](#_bookmark56)
    1.  Finite State Machines [23](#finite-state-machines)
        1.  Advantages and disadvantages [24](#advantages-and-disadvantages)
    2.  Hierarchical Finite State Machines [24](#hierarchical-finite-state-machines)
        1.  Advantages and disadvantages [24](#advantages-and-disadvantages-1)
        2.  Creating a FSM that works like a BTs [29](#creating-a-fsm-that-works-like-a-bts)
        3.  Creating a BT that works like a FSM [32](#creating-a-bt-that-works-like-a-fsm)
    3.  Subsumption Architecture [32](#subsumption-architecture)
        1.  Advantages and disadvantages [33](#advantages-and-disadvantages-2)
        2.  How BTs Generalize the Subsumption Architecture [33](#how-bts-generalize-the-subsumption-architecture)
    4.  Teleo-Reactive programs [33](#teleo-reactive-programs)
        1.  Advantages and disadvantages [34](#advantages-and-disadvantages-3)
        2.  How BTs Generalize Teleo-Reactive Programs [35](#how-bts-generalize-teleo-reactive-programs)
    5.  Decision Trees [35](#decision-trees)
        1.  Advantages and disadvantages [36](#advantages-and-disadvantages-4)
        2.  How BTs Generalize Decision Trees [36](#how-bts-generalize-decision-trees)
    6.  Advantages and Disadvantages of Behavior Trees [37](#advantages-and-disadvantages-of-behavior-trees)
        1.  Advantages [37](#advantages)
        2.  Disadvantages [42](#disadvantages)

```{=html}
<!-- --
```

3.  Design principles [45](#_bookmark99)
    1.  Improving Readability using Explicit Success Conditions [45](#improving-readability-using-explicit-success-conditions)
    2.  Improving Reactivity using Implicit Sequences [46](#improving-reactivity-using-implicit-sequences)
    3.  Handling Different Cases using a Decision Tree Structure [47](#handling-different-cases-using-a-decision-tree-structure)
    4.  Improving Safety using Sequences [47](#improving-safety-using-sequences)
    5.  Creating Deliberative BTs using Backchaining [49](#creating-deliberative-bts-using-backchaining)
    6.  Creating Un-Reactive BTs using Memory Nodes [51](#creating-un-reactive-bts-using-memory-nodes)
    7.  Choosing the Proper Granularity of a BT [52](#choosing-the-proper-granularity-of-a-bt)
    8.  Putting it all together [53](#putting-it-all-together)
4.  Extensions of Behavior Trees [57](#_bookmark127)
    1.  Utility BTs [58](#utility-bts)
    2.  Stochastic BTs [58](#stochastic-bts)
    3.  Temporary Modification of BTs [59](#temporary-modification-of-bts)
    4.  Other extensions of BTs [61](#other-extensions-of-bts)
        1.  Dynamic Expansion of BTs [61](#dynamic-expansion-of-bts)
5.  Analysis of Efficiency, Safety, and Robustness [63](#_bookmark137)
    1.  Statespace Formulation of BTs [63](#statespace-formulation-of-bts)
    2.  Efficiency and Robustness [66](#efficiency-and-robustness)
    3.  Safety [70](#safety)
    4.  Examples [73](#examples)
        1.  Robustness and Efficiency [74](#robustness-and-efficiency)
        2.  Safety [77](#safety-1)
        3.  A More Complex BT [81](#a-more-complex-bt)
6.  Formal Analysis of How Behavior Trees Generalize Earlier Ideas [83](#_bookmark185)
    1.  How BTs Generalize Decision Trees [83](#how-bts-generalize-decision-trees-1)
    2.  How BTs Generalize the Subsumption Architecture [86](#how-bts-generalize-the-subsumption-architecture-1)
    3.  How BTs Generalize Sequential Behavior Compositions [88](#how-bts-generalize-sequential-behavior-compositions)
    4.  How BTs Generalize the Teleo-Reactive approach [89](#how-bts-generalize-the-teleo-reactive-approach)
        1.  Universal Teleo-Reactive programs and FTS BTs [91](#universal-teleo-reactive-programs-and-fts-bts)
7.  Behavior Trees and Automated Planning [93](#_bookmark208)
    1.  The Planning and Acting (PA-BT) approach [94](#the-planning-and-acting-pa-bt-approach)
        1.  Algorithm Overview [98](#algorithm-overview)
        2.  The Algorithm Steps in Detail [98](#the-algorithm-steps-in-detail)
        3.  Comments on the Algorithm [101](#comments-on-the-algorithm)
        4.  Algorithm Execution on Graphs [103](#algorithm-execution-on-graphs)
        5.  Algorithm Execution on an existing Example [104](#algorithm-execution-on-an-existing-example)
        6.  Reactiveness [111](#reactiveness)
        7.  Safety [112](#safety-2)
8.  Fault Tolerance [113](#fault-tolerance)
9.  Complex Execution on Realistic Robots [114](#complex-execution-on-realistic-robots)

```{=html}
<!-- --
```

2.  Planning using A Behavior Language (ABL) [127](#planning-using-a-behavior-language-abl)
    1.  An ABL Agent [127](#an-abl-agent)
    2.  The ABL Planning Approach [130](#the-abl-planning-approach)
    3.  Brief Results of a Complex Execution in StarCraft [134](#brief-results-of-a-complex-execution-in-starcraft)
3.  Comparison between PA-BT and ABL [135](#comparison-between-pa-bt-and-abl)

```{=html}
<!-- --
```

8.  Behavior Trees and Machine Learning [137](#_bookmark293)
    1.  Genetic Programming Applied to BTs [137](#genetic-programming-applied-to-bts)
    2.  The GP-BT Approach [139](#the-gp-bt-approach)
        1.  Algorithm Overview [140](#algorithm-overview-1)
        2.  The Algorithm Steps in Detail [143](#the-algorithm-steps-in-detail-1)
        3.  Pruning of Ineffective Subtrees [144](#pruning-of-ineffective-subtrees)
        4.  Experimental Results [144](#experimental-results)
        5.  Other Approaches using GP applied to BTs [149](#other-approaches-using-gp-applied-to-bts)
    3.  Reinforcement Learning applied to BTs [149](#reinforcement-learning-applied-to-bts)
        1.  Summary of Q-Learning [150](#summary-of-q-learning)
        2.  The RL-BT Approach [150](#the-rl-bt-approach)
        3.  Experimental Results [151](#experimental-results-1)
    4.  Comparison between GP-BT and RL-BT [153](#comparison-between-gp-bt-and-rl-bt)
    5.  Learning from Demonstration applied to BTs [153](#learning-from-demonstration-applied-to-bts)
9.  Stochastic Behavior Trees [155](#_bookmark325)
    1.  Stochastic BTs [155](#stochastic-bts-1)
        1.  Markov Chains and Markov Processes [156](#markov-chains-and-markov-processes)
        2.  Formulation [159](#formulation)
    2.  Transforming a SBT into a DTMC [164](#transforming-a-sbt-into-a-dtmc)
        1.  Computing Transition Properties of the DTMC [166](#computing-transition-properties-of-the-dtmc)
    3.  Reliability of a SBT [169](#reliability-of-a-sbt)
        1.  Average sojourn time [169](#average-sojourn-time)
        2.  Mean Time To Fail and Mean Time To Succeed [170](#mean-time-to-fail-and-mean-time-to-succeed)
        3.  Probabilities Over Time [172](#probabilities-over-time)
        4.  Stochastic Execution Times [172](#stochastic-execution-times)
        5.  Deterministic Execution Times [173](#deterministic-execution-times)
    4.  Examples [175](#examples-1)
10. Concluding Remarks [187](#_bookmark389)

References [188](#references)
