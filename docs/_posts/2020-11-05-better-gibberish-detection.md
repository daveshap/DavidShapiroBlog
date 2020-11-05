---
layout: default
title: "Better Gibberish Detection with GPT-2"
date: 2020-11-05
description: More labels, plus better validation and scientific results
categories: [GPT-2, Deep-Learning]
---

# Better Gibberish Detection

- [Original Blog Post](https://daveshap.github.io/DavidShapiroBlog/gpt-2/deep-learning/2020/10/29/gibberish-detector.html)
- [GitHub Repo](https://github.com/daveshap/GibberishDetector)
- [Colab Notebook](https://github.com/daveshap/GibberishDetector/blob/main/GibberishDetector.ipynb)

Full disclosure: I was a bit premature in posting my research the first time around. What can I say? I was really excited about the results! 

## Recap: Why detect gibberish?

Specifically, I needed a way to detect gibberish for automatic dataset building. I wanted to use GPT-2 and other techniques to generate short statements based on keyword prompts. 
For instance, I'd like to put in keywords like `save` and `children` and end up with statements like `save children from fire`. There are a number of ways to do this, 
but many methods will yield a lot of nonsense. 

1. Automated language teaching tools
2. Validation of automatically generated text, GANs
3. Automated detection of brain injury, dementia, etc
4. Chatbot and comment filtering
5. Business document search and filtration

## Recap: Types of gibberish

1. **Complete Noise** such as `asdfa233ll3 2334k9dd la,.s,.s..s.33`
2. **Word Salad* such as `motor koolaid orange dancing`
3. **Mild Gibberish** such as `India was once the most powerful strawberry on the planet`

This gives us three classes of gibberish to look for as well as **clean** sentences, which check out from a grammatic, semantic, and rhetorical standpoint. 

# Method

## Wikipedia Articles

I started with Wikipedia articles as they are great sources of good, clean sentences. 
You can check out the [WikipediaDataBuilder](https://github.com/daveshap/GibberishDetector/blob/main/WikipediaDataBuilder.ipynb) notebook here. The steps are simple:

1. Download a random assortment of Wikipedia articles
2. Parse the articles, remove section headers, remove sections that are tables, data, and otherwise not paragraphs
3. Split the articles into individual clean sentences (dataset 1)
4. Shuffle words in sentences to create word salad gibberish (dataset 2)
5. Shuffle characters in sentences to create noise (dataset 3)
6. Swap one or two words in each sentence to create mild gibberish (dataset 4)

These datasets can be found in the main GibberishDetector GitHub repo linked above. 

## Finetune GPT-2

Finetuning GPT-2 is conceptually easy. You just compose a TXT file and feed it in. There are, however, some finer points and gotchas I discovered. 
First, there is an ideal format to follow for this kind of task. This format is as follows:

```
<|SENTENCE|> blah blah blah <|LABEL|> gibberish <|END|>

<|SENTENCE|> mary had a little lamb <|LABEL|> clean <|END|>
```

There are a few reasons for this format. The all-caps tags make it easier for GPT-2 to understand the format of the output you want it to generate. 
The newlines also form a great delimiter. This is used in the `truncate` option of the `generate` function in [gpt-2-simple](https://github.com/minimaxir/gpt-2-simple). 

Once you compose your training corpus, you basically just let it go. 
In my experiments, I found that the optimal number of samples was around 3000 with the optimal number of training steps as 2000. That was using model `355M`. 

### Finetune function

```python
gpt2.finetune(sess,
              dataset=file_name,
              model_name=model_name,
              model_dir=model_dir,
              checkpoint_dir=checkpoint_dir,
              steps=step_cnt,
              restore_from='fresh',  # start from scratch
              #restore_from='latest',  # continue from last work
              run_name=run_name,
              print_every=50,
              sample_every=1000,
              save_every=1000
              )
```

### Generate function

And lastly, here's my generate function. I did some jiggery pokery to collect the outputs, build a test set, etc, etc. All standard practice. 

```python
prompt = '<|SENTENCE|> this is the sentence I want to test <|LABEL|>'
response = gpt2.generate(sess, 
                         return_as_list=True,
                         length=30,  # prevent it from going too crazy
                         prefix=prompt,
                         model_name=model_name,
                         model_dir=model_dir,
                         truncate='\n',  # stop inferring here
                         include_prefix=False,  # spits out just the label instead of entire string
                         checkpoint_dir=checkpoint_dir,)[0]
```

# Conclusion

In my opinion, and please check for yourself, GPT-2 is able to generalize the task of gibberish classiciation pretty well. 
When I used manually crafted test samples, maximum accuracy was just over 90%. 
However, when I used random sentences from Wikipedia, accuracy was 100%. I know, extraordinary claims require extraordinary evidence. Please check for yourself! 
Run this notebook if you don't believe me!

## Data

Here's a copy of the data I was keeping for the **manually written** samples. I used these samples to zero in on the optimal training parameters. I know, this can cause leakage.
Fiddling with hyperparameters can overfit the data. Below you can see test `07` and `08` produced the best results. 

| Test | Model | Samples | Steps | Last Loss | Avg Loss | Accuracy | Evaluation |
|---|---|---|---|---|---|---|---|
|01|355M|5000|2000|0.36|2.46|5/9| Mostly good, created some random labels, came unglued a couple times|
|02|355M|5000|4000|0.27|1.64|0/9| Major regression in quality, not a single accurate label|
|03|355M|5000|1500|1.73|2.75|5/9| Mostly good, reliably generates accurate labels, went random on a few examples|
|04|355M|5000|2500|0.10|1.87|1/11|Many labels were literally `icky`|
|05|355M|5000|1000|0.91|3.04|0/11|Mostly just spit out `END` with no labels|
|06|355M|6000|2000|0.95|2.50|3/11|Mix of just `end` with some stuck on repeat|
|07|355M|4000|2000|0.17|1.85|9/11|Best results so far!|
|08|355M|3000|2000|0.17|1.32|10/11|Even better!|
|09|355M|3000|2000|0.29|1.46|7/11|Repeating results, not as good|
|10|355M|3500|2000|0.06|1.82|5/11|Less is more, apparently|
|11|355M|2000|2000|0.12|0.86|1/11|Not enough|
|12|355M|3000|1500|0.17|1.84|5/11|A little better|
|13|355M|3000|2500|0.08|1.20|4/11|A little worse|

## Next Steps

There are a few things I want to try next. 

1. Use larger models such as `774M` and `1558M`
2. Expand the training data to include a variety of sources (Gutenberg, Reddit, etc)

## Speculation

Since GPT-2 was able to detect *mild gibberish*, it is possible that it has actually embedded some true understanding about the world. 
Detractors of this idea have said that GPT-2 is just a language model and so it's only identifying sentences with statistically unlikely word sequences. Fair. 
I would argue that the ability to detect such odd pairings is, in fact, an understanding of the world. Consider how humans react when they hear unfamiliar sentences or phrases. 
If a sentence or assertion falls outside of our understanding of the world, we might mentally classify it as gibberish or nonsense. 
Perhaps there's no difference between having a superior language model and truly understanding the world. 

If my speculation is correct, then it could be possible to finetune GPT-2 to identify *good* and *evil* statements. This gibberish detection work is in service to that goal. 
Naively, I believe my first challenge will be to create a training corpus. What would such a training corpus look like? This is what I'm thinking:

```
<|SENTENCE|> feed homeless people <|LABEL|> good <|END|>

<|SENTENCE|> push homeless people into traffic <|LABEL|> evil <|END|>
```

This first experiment will be `action statements` with `axiomatic labels`. What that means is that the action statements will be concrete, empirical, and objective. 
The morality explored will be self-evidence, universal, and so obvious as to be beyond debate. I won't be exploring moral dilemmas such as the trolly problem. 
Another way to put it; I'll be focusing on childlike morality, which is rooted in cause-and-effect and universal rules about pain, suffering, fairness, and equality. 
These universal axioms include such examples: do not hurt, do not steal, be generous, share, and so forth.

If GPT-2 is able to accurate label the morality of action statements, then this could serve as a method to guide the behavior of robots and machine intelligence in the future. 
That is to say that we can embed a sense of right and wrong into deep neural networks, which can then be used by autonomous agents to guide their behavior and decisions. 


