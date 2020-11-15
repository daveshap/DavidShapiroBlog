---
layout: default
title: "Question Detection with GPT-2"
date: 2020-11-15
description: Detecting questions in lower case text without punctuation
categories: [GPT-2, Deep-Learning]
---

# TLDR

- [GitHub Repo](https://github.com/daveshap/QuestionDetector)
- [Detector Notebook](https://github.com/daveshap/QuestionDetector/blob/main/QuestionDetector.ipynb)
- [Data Prep Notebook](https://github.com/daveshap/QuestionDetector/blob/main/DownloadGutenbergTop100.ipynb)

# Abstract

Detecting questions from text without capitalization or punctuation is a non-trivial problem. 
In this work, I demonstrate that GPT-2 is capable of accurately inferring whether or not a sentence is a question just from the letters and spaces. 
There are several cases in which this technique can help, such as with ASR or chat. At the highest level, this is a type of Intent Detection. 

# Training Data

I used the [top 100 e-books from Gutenberg](https://www.gutenberg.org/browse/scores/top#books-last30) as a data source. 
I then split each e-book into chunks based upon double-lines of vertical whitespace. Each chunk was then condensed into a single line with extraneous whitespace removed. 
Thus, each line represented a paragraph of text. However, many lines contained unwanted information, such as titles, chapters, and other such metadata. 
These lines were filtered out using a variety of techniques:

- Remove lines that are all caps
- Remove lines that are mostly symbols or numbers
- Remove lines that are too short
- Remove lines that don't contain punctuation or quotation marks

This array of techniques proved to be mostly effective at distilling many books into their primary contents. 

Penultimately, the chunks were further split into sentences by using 
[SpaCy Sentence Boundary Disambiguation](https://spacy.io/universe/project/python-sentence-boundary-disambiguation).
The ultimate step was to format the training data and save it to file. An example follows:

```
<|SENTENCE|> why do you doubt your senses <|LABEL|> question <|END|>

<|SENTENCE|> you receive stolen goods do you <|LABEL|> question <|END|>

<|SENTENCE|> would you like to be taught latin <|LABEL|> question <|END|>

<|SENTENCE|> what could be taking him so long <|LABEL|> question <|END|>

<|SENTENCE|> that is unbelievable <|LABEL|> exclamation <|END|>

<|SENTENCE|> my name is indred cold <|LABEL|> other <|END|>
```

## Test Data

Test data was created with the same methodology as the training data with one final step added; trimming the label and end tag. For instance:

```
<|SENTENCE|> why do you doubt your senses <|LABEL|> 

<|SENTENCE|> you receive stolen goods do you <|LABEL|> 

<|SENTENCE|> would you like to be taught latin <|LABEL|> 

<|SENTENCE|> what could be taking him so long <|LABEL|> 

<|SENTENCE|> that is unbelievable <|LABEL|> 

<|SENTENCE|> my name is indred cold <|LABEL|> 
```

This format has been demonstrated to be effective with GPT-2. It leaves a space for GPT-2 to "fill in the blank". 
At inference time, GPT-2 is instructed to truncate (stop inference) at a newline. 

# Results

Even the smallest version of GPT-2 produced far better-than-random results. 

| Run | Model | Steps | Samples | Last Loss | Avg Loss | Accuracy |
|---|---|---|---|---|---|---|
| 01 | 124M | 2000 | 9000 | 0.07 | 0.69 | 71.4% |

The results speak for themselves!

# Limitations

- Training data used includes several examples of non-English text
- Sentence boundary disambiguation does not take dialog tags into account, such as `she said` or `he replied`
- Sentence boundary disambiguation also does not take consistently quotations into account 

# Discussion

This technology can be used to enhance chatbot functionality, allowing a dialog system to infer high level intent without the use of punctuation.
This could be doubly useful for ASR systems, such as those used for mobile devices or dictation software. Higher accuracy and more labels should be possible with better data,
such as with a Dialog Act corpus. There are a few dozen types of Dialog Acts in human conversation, each of which can better inform a dialog systems or chatbot. 
As with most problems in Machine Learning and Artificial Intelligence, the quality of results is wholly dependent upon the quality of data. 
Therefore, follow-up work should include better refinement of training data. 



