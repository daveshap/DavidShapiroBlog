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

Even with an RTX 2070, I lacked adequate resources to train locally. I kept getting `OOM`, the dreaded "Out of Memory" error. So I moved over to Google Colab. 

## Install stuff

Start your environment as GPU or TPU. I like to install stuff quietly upfront. 

```python
!pip install wikipedia --quiet
!pip install spacy --quiet
!pip install pysbd --quiet
!pip install tensorflow-gpu==1.15.0 --quiet #--force-reinstall
!pip install gpt2-client==2.1.5 --quiet --no-dependencies #--force-reinstall 
```

## Download Wikipedia articles

We need some good, solid, rhetorically sound data to work with. 

```python
import wikipedia

# todo: come up with a cool way to automatically create topic search terms
keywords = ['india', 'ocean', 'astronomy', 'economics', 'economy', 'earth', 
            'english', 'bacon', 'egg', 'dinosaur', 'rabbit', 'america', 'usa']

def save_article(title, article):
  with open('wiki_' + title + '.txt', 'w', encoding='utf-8') as outfile:
    outfile.write(article)

for keyword in keywords:
  try:
    search = wikipedia.search(keyword)
    for result in search:
      article = wikipedia.page(result)
      #print(result, article.url)
      save_article(result, article.content)
  except Exception as oops:
    #print(oops)
    continue
print('Done saving articles!')
```

## Parse the articles

Let's parse the Wikipedia articles out to single-lines per section. 

```python
import os 
import re

result = list()

for file in os.listdir('.'):
  if not 'wiki_' in file:
    continue
  #print(file)
  with open(file, 'r', encoding='utf-8') as infile:
    text = infile.read()
  sections = re.split(r'={2,}.{0,80}={2,}', text)
  for section in sections:
    try:
      trimmed = section.strip()
      wordchars = re.findall(r'\w', trimmed)
      ratio = len(wordchars) / len(trimmed)
      if ratio > 0.80:
        final = re.sub(r'\s+', ' ', trimmed)
        result.append(final)
      # it seems like a ratio of greater than 80% word chars is ideal
    except:
      continue
  
print('Wikipedia sections parsed:', len(result))
with open('wikiparsed.txt', 'w', encoding='utf-8') as outfile:
  for line in result:
    outfile.write(line+'\n')
```

## Isolate sentences

We want to keep this as simple as possible so let's break it down into the smallest rhetorical units in the English language: Sentences.

```python
import spacy
from pysbd.utils import PySBDFactory

nlp = spacy.blank('en')
nlp.add_pipe(PySBDFactory(nlp))
infile = 'wikiparsed.txt'
outfile = 'wikisentences.txt'
result = list()

with open('wikiparsed.txt', 'r', encoding='utf-8') as infile:
  lines = infile.readlines()
for line in lines:
  doc = nlp(line)
  #print('Parsing line:', line[0:80])
  for sent in list(doc.sents):
    result.append(sent)
    #print(sent)
#print('Sentences found:', len(result))
with open('wikisentences.txt', 'w', encoding='utf-8') as file:
  for line in result:
    if str(line) == '':
      continue
    file.write(str(line)+'\n')
print(outfile, 'saved!')
```

## Generate grammatic gibberish

Let's start with clean sentences and shuffle the words.

```python
from random import shuffle, seed

infile = 'wikisentences.txt'
outfile = 'wikiscrambled.txt'
result = list()

def scramble_sentence(sentence):
  sentence = sentence.strip()
  split = sentence.split()
  shuffle(split)
  return ' '.join(split)

seed()
with open(infile, 'r', encoding='utf-8') as file:
  lines = file.readlines()
for line in lines:
  line = line.strip()
  if line == '':
    continue
  scrambled = scramble_sentence(line)
  result.append(scrambled)
  #print('Scrambled sentence:', scrambled[0:100])
with open(outfile, 'w', encoding='utf-8') as file:
  for line in result:
    file.write(line+'\n')
print(outfile, 'saved!')
```

## Generate semantic gibberish

Same as above, let's just shuffle all the characters!

```python
from random import shuffle, seed

infile = 'wikisentences.txt'
outfile = 'wikiscrambled2.txt'
result = list()

def scramble_sentence(sentence):
  sentence = sentence.strip()
  sentence = list(sentence)
  shuffle(sentence)
  return ''.join(sentence)

seed()
with open(infile, 'r', encoding='utf-8') as file:
  lines = file.readlines()
for line in lines:
  line = line.strip()
  if line == '':
    continue
  scrambled = scramble_sentence(line)
  result.append(scrambled)
  #print('Scrambled sentence:', scrambled[0:100])
with open(outfile, 'w', encoding='utf-8') as file:
  for line in result:
    file.write(line+'\n')
print(outfile, 'saved!')
```

## Compile a training corpus

Finetuning GPT-2 requires a single text file as training corpus. 

```python
from random import sample, seed

files = [
('wikisentences.txt', 'Clean'), 
('wikiscrambled2.txt', 'Gibberish'), 
('wikiscrambled.txt', 'Gibberish')
]

result = list()
max_samples = 100  # adjust this
corpus = 'corpus.txt' 

for file in files:
  with open(file[0], 'r', encoding='utf-8') as infile:
    lines = infile.readlines()
  for line in lines:
    line = line.strip()
    if line == '':
      continue
    line = '// %s || %s' % (line, file[1])
    result.append(line)
    #print(file, line[0:80])

seed()
subset = sample(result, max_samples)

with open(corpus, 'w', encoding='utf-8') as outfile:
  for line in subset:
    outfile.write(line+'\n\n')
print(corpus, 'saved!')
```

## Finetune GPT-2

This is where it gets exciting! It's just a handful of lines of code now:

```python
from gpt2_client import GPT2Client

gpt2 = GPT2Client('345M')  # options: 117M, 345M, 774M, or 1558M
gpt2.load_model(force_download=False) 

corpus = 'corpus.txt'

result = gpt2.finetune(corpus, return_text=True)
print(result)
```

## Training results

So far, so good! Because GPT is a pre-trained technology (duh, it's in the name!) it actually doesn't take too long to fine-tune. 

```python
[57 | 5268.46] loss=0.18 avg=2.65
[58 | 5360.14] loss=0.57 avg=2.61
[59 | 5453.29] loss=0.46 avg=2.56
[60 | 5545.35] loss=0.13 avg=2.51
[61 | 5635.39] loss=0.15 avg=2.45
[62 | 5726.99] loss=0.14 avg=2.40
[63 | 5818.09] loss=0.18 avg=2.36
[64 | 5912.42] loss=0.19 avg=2.31
[65 | 6005.45] loss=0.10 avg=2.27
[66 | 6098.33] loss=0.12 avg=2.22
[67 | 6192.00] loss=0.43 avg=2.18
[68 | 6284.62] loss=0.07 avg=2.14
[69 | 6375.34] loss=0.06 avg=2.10
[70 | 6466.31] loss=0.05 avg=2.06
[71 | 6558.57] loss=0.12 avg=2.02
[72 | 6668.53] loss=0.14 avg=1.98
[73 | 6760.46] loss=0.06 avg=1.95
[74 | 6849.87] loss=0.10 avg=1.91
[75 | 6940.43] loss=0.63 avg=1.89
[76 | 7030.83] loss=0.07 avg=1.85
```

## Real-world test results

Stay tuned!
