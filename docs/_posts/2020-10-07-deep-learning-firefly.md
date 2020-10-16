---
layout: default
title: "How deep learning could give us more Firefly"
date: 2020-10-07
description: Take my love, take my land. Take me where I cannot stand.
categories: [Singularity, Deep Learning]
---

We all want more Firefly, so let's do a thought experiment as to how deep learning could achieve this!

# The Screenplay

Technologies like GPT-3 show that we now have deep learning models that can generalize and reproduce a broad variety of text outputs. For instance, [GPT-3 allegedly went undetected on r/AskReddit](https://www.kmeme.com/2020/10/gpt-3-bot-went-undetected-askreddit-for.html) for a week or so.
GPT-3 is what's known as a "one shot" or "few shot" technology, where it has baked in ability to recognize the type of output you want from a single example. So let's say we fine tune GPT-3 with a bunch of TV show scripts and sci-fi novels and then show it the actual original Firefly screenplay and see what it produces.
"Fine tuning" is something you can do on GPT-2 whereby you [add more data to make it more purpose-built for your particular task](https://openai.com/blog/fine-tuning-gpt-2/). Fine-tuning allows you to specialize a big giant model without starting from scratch every time. 

Stay tuned, I think I will give this a shot on my own sometime soon! If I get around to it, I will post an update here, with the results and the code!

EDIT: [Oh look, someone already did it!](https://towardsdatascience.com/film-script-generation-with-gpt-2-58601b00d371). There's a lot of attention on [GPT-3 for this task already](https://www.gwern.net/GPT-3)

## What a screenplay looks like

Just for reference, screenplays are highly standardized with very specific syntax and formats so they are universally accessible and interpretable by directors and producers. They include descriptions, actions, and dialog.

[Here's an example of what one looks like on paper](https://www.raindance.org/scripts/Firefly_1x02_-_Bushwhacked.pdf)

# The Video

First we need to break it down into chunks. We don't need to use a deep neural network to make an entire 44 minute episode. Episodes are broken down into scenes, and scenes are broken down into cuts, or "transitions".
Everytime the camera cuts away to a new perspective is a film segment. Some directors and producers make use of rapid cuts, meaning we would only need to generate a few seconds of video at a time, and then stitch it together. 
Firefly, which I just rewatched with my partner, seems to be pretty standard. A long cut in that show would be 30 to 60 seconds, but most are shorter, close up of faces and dialog with some action sequences, establishing shots of Serenity and scenery.
[DVD-GAN](https://medium.com/syncedreview/deepmind-dvd-gan-impressive-step-toward-realistic-video-synthesis-12027d942e53) is already on the way to full video synthesis.

## Scenery and Settings

GANs (Generative Adversarial Networks) have become exceptionally good at [generating realistic images](https://www.marktechpost.com/2020/10/06/nvidia-releases-imaginaire-a-universal-pytorch-library-designed-for-various-gan-based-tasks-and-methods/) from basic information. 
NVIDIA released a library called Imaginaire that attempts to standardize this technology, making it more and more accessible. I think it's only a matter of time before this increases in sophistication and quality.
Right now, [text-to-image](https://deepai.org/machine-learning-model/text2img) technology leaves a bit to be desired! As GPUs get more powerful and data increases, we will inevitably see better models.

## Characters

We humans (and yes, I'm a human) are finely calibrated to recognize faces. The uncanny valley has been the death of many early technologies, from CGI to video games. Sites like [This Person Does Not Exist](https://thispersondoesnotexist.com/), however, demonstrate that we are well past the uncanny valley of face generation. 
Nathan Fillion has already been [deep-faked into live-action footage](https://www.eurogamer.net/articles/2020-05-02-uncharted-4-deepfake-starring-nathan-fillion-is-as-impressive-as-it-is-scary). So why not Firefly?

# The Audio

## Speech

Text-to-speech is nothing new. The latest and greatest speech synthesis adds inflection, tone, and style tags. The subtle quality of human emotion poured into speech can be summed up as "prosody". [Here's a creepy-realistic example of prosody embedding!](https://ai.googleblog.com/2018/03/expressive-speech-synthesis-with.html)

## Sound Effects

Back in 2016, [MIT published some work](https://www.wired.com/2016/06/mit-artificial-sound-effects/) about natural sound generation for video. Today, Adobe can provide this as a service, and it's eerily high quality. Seriously, check out [this train](https://research.adobe.com/news/ai-can-generate-realistic-sound-for-video-clips/).

## Music

Synthetic music is also nothing new, it's just getting [way better](https://towardsdatascience.com/neuralfunk-combining-deep-learning-with-sound-design-91935759d628). And today, music AI is getting to an entirely new [level of beautiful](https://www.zmescience.com/science/ai-musical-composer/).

# Implications

## Netflix and GAN?

So what does this mean? I think the only logical conclusion is we're going to see a massive explosion of consumer media. If they aren't already, I suspect that Netflix and Amazon are hard at work creating fully synthetic consumer media. We'll probably see books and short stories first, but it's only a matter of time before that morphs to TV and movies. Imagine this: Netflix creates a library full of tens of thousands of movies and shows, all procedurally generated and rated by the masses. Those that are good percolate up. They are filled with actors that never existed, written and produced by directors who never existed. This level of automated media production is still prohibitively expensive. It took millions of dollars just to train GPT-3, which can only do generalized text tasks. It will take something a bit more powerful and sophisticated to do high quality TV screenplays and 4k 60fps film. 

## IP Laws

I haven't the foggiest clue as to how this is going to play out. I suspect that IP (intellectual property) laws will say that AI models trained in-house are propriety and that any data they use is part of the model. Thus, they will need rights or license to train on other TV and movies. You can't just copy the screenplays of every Marvel movie and not expect Disney to sue your pants off! Even if an AI is then just learning from the screenplay, the same that another director might. I could see that going to the Supreme Court.

## Personalized TV, Movies, Music, and Books

As GPU technology advances, which it is rapidly due to demand, it will become cheaper and cheaper to train giant models. My personal desktop computer, with its NVIDIA RTX 2070, is more powerful than ASCI Red, the top supercomputer from 1997. Commercial industry is usually about 10 years behind the cutting edge of massive computing power and private homes are about 20 years behind. It took supercomputer level processing to train GPT-3, so we can expect run-of-the-mill businesses to be able to do that by 2030, and everyone else by 2040. 
As the cost of producing these giant models comes down, and the quality of data and output increases, we will soon see hyper-personalized entertainment. Want more Firefly? Just ask Netflix or Amazon. Want more Game of Thrones, except the ending is way different? That could be possible, too! 
And I don't mean recommender systems, either. I mean stuff that is generated *just for you*. 
