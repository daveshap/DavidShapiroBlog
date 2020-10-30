---
layout: default
title: "Gibberish Detection with GPT-2"
date: 2020-10-29
description: Has Anyone Really Been Far Even as Decided to Use Even Go Want to do Look More Like?
categories: [GPT-2, Deep Learning]
---

# Gibberish Detection

TLDR:

- [Notebook on Google Colab](https://colab.research.google.com/drive/1or04YRQ3LbnotC8LCo232vYf6qsZT7Cb?usp=sharing)
- [Repo on GitHub](https://github.com/daveshap/GibberishDetector)

## What is gibberish?

In my estimation, there are 3 kinds of gibberish.

1. Textual noise: `asdfasdfa asdf2233k3k3kk`
2. Word salad: `Monkey blue running incandescent`
3. Nonsense: `Sometimes I sneeze out a universe`

The first type would be easy enough to detect by simply matching each token to a dictionary term. The second type is much harder, and that's what I'm focusing on today. 
Number 2 is composed of words, yes, but they are random with no syntactic meaning. Another way of saying this is that it is not grammatically correct. 
Here's another way of looking at these types of gibberish from a linguistics perspective:

1. Semantic gibberish - no discernible meaning can be detected.
2. Grammatic gibberish - on their own, the words have meaning, but no higher order meaning can be extracted from phrases, clauses, or sentences.
3. Rhetorical gibberish - The sentence is grammatically correct but in the context of reality, it does not check out. 

The first level of gibberish detection simply requires a dictionary. The second level requires some kind of language model. The third level requires actual understanding of the world. 
Each level of sophistication represents an order of magnitude more complexity. 

## Why detect gibberish?

There are a lot of possible uses!

1. Automated language teaching tools
2. Validation of automatically generated text
3. Automated detection of brain injury, dementia, etc
4. Chatbot and comment filtering
5. Business document search and filtration

As you can see, there are quite a few possible uses right off the top of my head. 

## Dictionary, Language Model, and Understanding

These are not small problems! Where to begin? It occurred to me that GPT is trained on a huge corpus of text. 
It's possible that the training procedure of GPT actually embeds all three of these components - dictionary, language model, and knowledge about the world.
If this is true, then it should be possible to finetune GPT-2 or GPT-3 to give a binary output - `gibberish` or `clean`. 

A dictionary can help you detect level 1 gibberish: semantic gibberish. 

A language model can help you detect level 2 gibberish: grammatic gibberish.

The previous two combined with true understanding of the world could, in theory, help you detect level 3 gibberish: rhetorical gibberish.

## This is basically Sentiment Analysis

I found [this helpful repo](https://github.com/spronkoid/GPT2-sentiment-analysis) about using GPT-2 for sentiment analysis. So we have here a **posteriori** example of 
GPT-2 performing SA. Here, I am making the assumption that SA requires the same components in order to make semantic, grammatical, and rhetorical evaluations of a statement.
It stands to reason - the more you know about grammar and the world in general, the better you can perform on Sentiment Analysis. There's also the example of 
[BERT being used for SA](https://www.topbots.com/sentiment-analysis-with-bert/) and achieving world-class results. So the proof is in the pudding: Transformers can do SA. 

The only place I'm reaching here is whether or not SA can generalize even more broadly to evaluate statements for whether or not they make rhetorical sense. 

# The Code

I won't duplicate the code here. Please check out the GitHub repo and Google Colab notebook for the code! 

# Results

Stay tuned!
