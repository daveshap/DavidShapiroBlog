---
layout: default
title: "Prediction: Human-equivalent neural networks by 2030"
date: 2021-02-19
description: Neural network sizes are doubling multiple times per year right now
categories: [Singularity]
---

# Moore's Law, but on Spice

GPT2's final update came in November 2019 and had 1.5B parameters. By comparison, GPT3 was released 6 months later (May 2020) with 175B parameters. 
A few months prior, Microsoft released a large transformer (Turing-NLG) with 17B parameters. This growth rate is unprecedented and, likely, is completely unsustainable. 

But along came Google. 

In January 2021, they released their Switch Transformer with 1.6T parameters (1600B). That's a growth curve of roughly 10x every 3 months. 
If that pattern continues (which I doubt it will), we're looking at 1600T parameters by the end of 2021. 
This parabolic growth would probably consume more electricity than the entire human race is capable of generating.

I suspect that this jump in parameters was largely driven by a few technological breakthroughs with neural architectures and massively distributed computing. Once the novelty
of those breakthroughs wears off, we'll see a much gentler growth. 

# Human Brain Equivalent?

The human brain has roughly 100 billion neurons. Each neuron has roughly 7000 synapses. If (and this is a big 'if') each synaptic connection in the brain is roughly equivalent to
one parameter in a Deep Neural Network, then the human brain has roughly 700T parameters. We'll get to architecture in a minute, but suffice to say, I don't think the human brain is a transformer.

These are some really broad, bold assumptions on my part. Please understand that they come from a place of WAGs and good intentions. How many deep neural network parameters does it take
to equate to the processing power of a single synapse? I have no idea. I could be off by an order of magnitude in either direction. But I'm about to demonstrate why that doesn't matter too much.

# Growth Rate

We've been seeing 1000x growth for a couple years running now. So cutting that down to 2x annual growth should be reasonable. 

| Parameters (T) | Year |
|---|---|
| 1.6 | 2021 |
| 3.2 | 2022 |
| 6.4 | 2023 |
| 12.8 | 2024 |
| 25.6 | 2025 |
| 51.2 | 2026 |
| 102.4 | 2027 |
| 204.8 | 2028 |
| 409.6 | 2029 |
| 819.2 | 2030 |
| 1,638.4 | 2031 |
| 3,276.8 | 2032 |
| 6,553.6 | 2033 |
| 13,107.2 | 2034 |
| 26,214.4 | 2035 |
| 52,428.8 | 2036 |
| 104,857.6 | 2037 |

As you can see, even if we average only a modest growth rate, compared to our current trends, we should exceed the 700T mark by 2030. 
Even if I'm off by an order of magnitude, it will only take a few more years to catch up.

Now let's see what that looks like at 4x annual growth (still a far cry of the current 1000x annual growth).

| Parameters (T) | Year |
|---|---|
| 1.6 | 2021 |
| 6.4 | 2022 |
| 25.6 | 2023 |
| 102.4 | 2024 |
| 409.6 | 2025 |
| 1,638.4 | 2026 |
| 6,553.6 | 2027 |
| 26,214.4 | 2028 |
| 104,857.6 | 2029 |
| 419,430.4 | 2030 |
| 1,677,721.6 | 2031 |
| 6,710,886.4 | 2032 |
| 26,843,545.6 | 2033 |
| 107,374,182.4 | 2034 |
| 429,496,729.6 | 2035 |
| 1,717,986,918 | 2036 |
| 6,871,947,674 | 2037 |


# Why Moore's Law Doesn't Matter (much)

Moore's Law looks at the transistor count on a single processor wafer.

We haven't used single processors in more than a decade. If you want more CPU horsepower, you can throw in more processors or more cores. 
OpenAI, IBM, Google, and Microsoft have been distributing their workloads across thousands of dedicated machines. This is nothing but good old fashioned clustering.
Along comes Cerebras and they overcome a design barrier and stuff 400,000 cores on a single wafer. Sure, it's a CPU that takes 20kW of energy, but the equivalent
computational power in Google's datacenter would take closer to a MW of juice. 

Companies like Nvidia, Google, and Cerebras are investing deeply in AI-specific hardware. A general purpose Intel CPU can't hold a candle to these AI-optimized beasts.
This is another reason that Moore's Law doesn't really represent a limiting factor. Optimization trumps brute force. We're now arriving at the time of refinement, nuance, and mastery.

# The Last Problem: Architecture

The transformer architecture has demolished all previous limitations. OpenAI is turning the power of GPT3 to other domains, including visual and audio. Will the transformer be the basis of True Cognition?

I doubt it. 

Will the achievement of transformer architectures contribute greatly to achieving true machine cognition? Absolutely. There are, however, still many problems that neural networks today haven't even begun to tackle, specific kinds of cognitive tasks.

## Short and Mid Term Memory

The greatest limitation of neural networks today is that they have no short term memory. The exception would be RNNs but their memory is so short-term that they can't even remember what they were talking about by the end of a few paragraphs.

If you set up a chat with GPT3 and tell it about your day, it will have no memory of that conversation unless you feed it back in at next instantiation. You can fine-tune these networks with one-shot training and zero-shot inference. 
But that's a far cry from spontaneous human memory. 

GPT3 is effectively only extremely short-term memory and long term memory.

## Metacognition

We humans can "think about our thought". We can play with our thoughts internally, mulling ideas and mental approaches. We can deliberately stop and think about something, and more importantly, we can ponder our own thought process.

Metacognition is, in my estimation, one of the most critical functions for true intelligence. Right now, deep neural networks spit out answers without any reasoning. It's pure intuition. GPT3 has no way to explain why it says what it says. 
There's no debating its output, as it has no grasp of logic or reasoning.

## Goal tracking

Human brains can spontaneously imagine a future goal state and continuously monitor our progress towards that goal. We can integrate feedback in real time. This is not something that has been demonstrated, to my knowledge, in neural networks.

## Theory of Mind

I can take a look at my friends and listen to a few sentences and get an accurate idea of their emotional and mental state. I can remember what they know, believe, and like. 

Most importantly, I can use this information to anticipate their behaviors and reactions. I can predict how best to interact with them. 

Yes, if you've watched The Social Dilemma or The Great Hack then you know that Facebook and Google absolutely can model your mind. But these technologies have not been integrated into the giant language models, such as GPT3. 

## Energy

The human brain consumes about 10W of energy. Cerebras' chip consumes 20kW. None of this will be practical or commercially viable until those power requirements come down.

## Commercial Lag

Supercomputers and massive clusters are about 10 years ahead of commercial deployments, which are again about 10 years ahead of consumer-grade computers. My desktop PC is an order of magnitude more powerful than Deep Blue was back in 1998. 

So Microsoft, IBM, and Google will be able to run human-scale networks by 2030. Your average company can run their own by 2040, and then everyone can have a digital human brain on their laptop by 2050.

## Tying it all together

We can slice and dice neural networks. This is called transfer learning and it has been around for a while. I suspect that human-level cognition will emerge, partly, through composing and compiling pieces of specifically trainined neural networks, stitching them together like a Frankenstein's monster of neural networks.

I also suspect that there will be other architectures that are more decentralized. Why compose a monolithic neural network when you can have smaller networks stitched together with middleware? But then again, I'm a systems engineer by trade. 

# The Implications

Let's say we get human-level intelligence by 2030. What then? I think the biggest impact will be to economics and jobs. We suddenly may find ourselves in the midst of the end of work as we know it. 

Capitalism is the inexorable and ruthless search for efficiency. As an engineer, I am all about some efficiency. I have taken countless processes in my career and automated them away. Automation is faster, cheaper, and more reliable. 

But what happens when a machine can learn to do anything that any human can do? What happens when you can copy/paste that machine infinitely? What happens when you can sell that machine's service online within a year?

OpenAI and Microsoft are working on monetizing GPT3 via API. Companies are going to figure out how to use that technology and then GPT4 is going to come out, as well as numerous other competitors. Companies will have already figured out how to use AI.

Imagine this: OpenAI comes out with GPT4 and releases dozens of specialized versions. Some help out with biotech or chemistry. The folks over at Volkswagen are working on their next gen battery and so they upload some formulas and specs to GPT4 and it spits out some testable results. Suddenly, GPT4 has done months worth of work in seconds. VW then goes and tests the results in a lab and then the following year their batteries are 20% better.

Imagine this: Google Brain releases a medical diagnostic bot in 2025 that can consume any genetic data, medical test, or medical image and recommend followup tests. It gives a diagnosis with 99.99% accuracy and works tirelessly, around the clock. Not only does it diagnose symptomatic problems, it predicts problems up to 20 years in advance. 

These tools will become cheaper, faster, and more reliable than humans. At that point, any and every company still around will make the financial calculation and switch. 





