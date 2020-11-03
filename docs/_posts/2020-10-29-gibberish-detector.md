---
layout: default
title: "Gibberish Detection with GPT-2"
date: 2020-10-29
description: Has Anyone Really Been Far Even as Decided to Use Even Go Want to do Look More Like?
categories: [GPT-2, Deep-Learning]
---

# Gibberish Detection

TLDR:

- [Notebook on Google Colab](https://github.com/daveshap/GibberishDetector/blob/main/GibberishDetector.ipynb)
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

After 400 iterations, here are some results. One thing you might notice is the lack of capital letters and periods. I realized that it was getting really good very quickly not because it was generalizing the rule that I wanted, but just because it was noting the location of caps and periods. Without those clues, training has gone far slower but has produced dramatically better results. 

```
// the first of these was the brazil of spanish colonists which was the same period which produced german immigrants || clean

// for example, in contrast to what he believed, the bhikkhu analayo taught that we have no theta (free will), "soul" or "wisdom" or "knowledge" (dhammā) nor our experiences are our real kamma (divine/sentient essence) but are but "possessions" (subordinate aggregates) || clean

// a later influence on the buddhist tradition was the dhamma (sanskrit: "school", "religions" or "discipleship") tradition of buddha vinaya which is an offshoot of sanskrit buddhist culture from the vajjian region of central india || clean

// the the the of and the of system global for of of the the environment of the air and the and global for and all the of for as of in pollution, the of areas the of and the for the in air, all air the for most of are air and or the and most from and levels particulate are || gibberish

// the of in the by tax the of and were the the that of value-added the sales 20 percent of the the and from sales and excise 20 percent the of and || gibberish

// this was also the case even if workers were not exposed to the same pathogens during their working hours || clean

// the of in that the was the first paper of the journal was a of the first the printed in that paper published in to of widely published the of the a in was circulation the of and and in in of for first the appeared in of of an was || gibberish

// the in developed in developed economic the the its the world of country into post-industrial the || gibberish

// some species of viruses were isolated by d'Orazio et al (1995) and identified by the d'Orazio genome-wide viral association study (2011) || clean

// during the nineteenth and twentieth centuries, many nations (notably spain and cambodia) adopted methods of tax and customs that greatly reduced the need for hunting; however, after 1924, due to overfishing by spanish colonists and the spanish-speaking nations, and a lack of suitable habitat, intensive agriculture was introduced as a result || clean

// the economic and social costs of these practices include forced labor, child labor, trafficking of labor, and subsistence farmers || clean

// of in to that for of between was the the and for of of trade of and countries in trade and of and the economic the world trade the tariff, export of tariff from import duty, of export of state and of from goods tariff trade, the export of goods trade protection were on and exports the and of tariffs, tariff and || gibberish

// of trade world, brazil and and pachamama the indian ocean world are countries and regions the regions countries and countries by and region the and of of asia, || gibberish

// the united states has a population of approximately 123 million (2010) and the lowest levels of infant mortality in the world || clean

// they are used for both medicinal and recreational purposes || clean

// in the 1950, the republic of mexico signed on as a brazil, and in 1953, the republic of mexico joined the organization, becoming a member of the confederation of states || clean

// the that of for of in his his with the of and be that of is his "one truth" is sūtrāhitvāda shaiva-siddhā, the also is the kalpa a a the dukkha in jainism and the aravāda-sutta of the theravāda-sīhanāda, mahāvastu "the the is mahāvastu the (c)in (c)the (c) || gibberish

// a the he british, of the kingdom of of and is of french (1798–1854), was english explorer, "father of modern world", || gibberish

// in the year 1824, a party formed by the representatives of the cia, confederation, and of the and states' trade unions and which had the support of pitt and caracasin the name of national sovereignty and of trade union of union congress of of was a the the the act to created the the of national federation the || gibberish
```

You can decide for yourself, but overall I think it has done pretty well! Up next, I am going to work on making it more consumable. 

# Follow-up Work: Good and Evil

I created this tool because I wanted to have the ability to check the quality of automatically generated text. There are numerous ways to generate text, including Transformers and GANs but there are also simpler, dumber ways. You can literally just choose words are random from a dictionary. You could populate sentences and phrases mad-libs style. Either way, I wanted the ability to create huge a huge corpus of training high quality training data and validate that each sample actually made sense. This current model mostly just detects word salad. I don't think it's sophisticated enough to detect rhetorical gibberish. For that, I will probably have to wait until I can run the 1.5B node GPT-2 or get access to GPT-3. 

Ultimately, my idea is to test the GPT technology's ability to recognize `good` and `evil`. Since good and evil are squishy concepts, the samples would have to be broken down into parables or examples. Iff (if and only if) the GPT technology has embedded not just language models but some higher order understanding of the world, then it should be able to generalize the rules of what makes something "good" or "evil"

```
// put live puppies in a blender || evil
// give soup to homeless children || good
```

That sort of thing. My hypothesis is that we can create moralistic models that can serve as a sort of "moral compass" for AGI. Whenever a robot or AGI needs to make a decision, it can feed its potential actions into a moral compass microservice to determine which actions are good or evil. 
