---
layout: default
title: "Parsing all of Wikipedia to an Offline Encyclopedia"
date: 2020-11-15
description: I want an offline encyclopedia and am extremely masochistic
categories: [Automation]
---

# Background

I'm working on a project where I want to have an offline encyclopedia. It's for training deep learning networks so it needs to be in plain text English. 
No markup, no special characters. Human readable without any interpreters, renderers, or parsers. I've got plenty of disk space, so that's not a concern. 
Once I parse out all the Wikipedia articles I can get my hands on, I will create an index. Or I might index them into SOLR or something like that. Not sure yet. 

# WikiMedia Text

The first thing I should have learned is that Wikipedia is written in a demented Frankenstein language called WikeMedia Text. It's a hybrid of HTML/XML and Markdown. 
It has no consistency and is the worst example of spaghetti code I've ever seen. I'm sure there are better implementations today, but I can see how and why it ended up the way it did.
For instance, you need to be able to create robust references and links, so the URL syntax is way jacked. It relies on a lot of procedural generation at display time.
Personally, if it were done again today, I think something like Jekyll would be way better. Instead of rendering again and again every time someone visits a page, render it once after each edit.
But that's just me. So instead we're left with this horrible hybrid language that should die in a fire. 

Fine, it is what it is. I'm an expert automator, dammit, and if a machine can automatically render this nonsense, then I sure as hell can **unrender it**. 

## Attempt 1 - Brute Force Regex

"Brute Force Regex" (BFR) is not a real thing. It's just something I've been doing for years now in my automation habits. Usually, as a naive approach, I'll try and do some 
search-and-replace jiggery pokery to just remove unwanted junk. Sometimes this is textual formatting, like brackets around tables or other HTML tags. 
So I ended up with the following function. Caution, it's not pretty. This was just an experiment, and I wanted to share it so you would see what doesn't work.

```python
def basic_replacements(text):
    replacements = [
    ('&lt;','<'),
    ('&gt;','>'),
    ('&quot;','"'),
    ("'''",' = '),
    ("'{2,}",''),
    ('\n',' '),
    (r'\n',' '),
    ('\\n',' '),
    ('r\\n',' '),
    ('<ref.*?>',''),
    ('</ref>',''),
    ('http.*?\s',''),
    ('\s+',' '),
    ]
    text = text.replace('\\n',' ')
    for r in replacements:
        text = re.sub(r[0], r[1], text)
    return text
```

I came up with this scheme because, at first glance, WikiMedia Text looked like a mixture of some basic HTML and some Markdown. 
I figured I could handle it with some basic regex replacements. This worked... to an extent. There were a few problems with it though.

1. Couldn't handle nested square brackets or curly brackets, and it turns out there are a lot of those
2. Quickly became intractable when I encountered escaped unicode literals like `\u2013`. They are frigging everywhere. 

So I wrote two more functions to try and tackle the bracketed stuff. These are things like links, citations, and pictures. Since I want a text-only Wikipedia, 
I really just needed to strip it all away. 

```python
def remove_double_curly(text):
    while True:
        before = len(text)
        text = re.sub('{{[^{]*?}}', '', text) 
        after = len(text)
        if before == after:
            return text


def remove_double_brackets(text):
    while True:
        before = len(text)
        double_brackets = re.findall('\[\[.*?\]\]', text)
        for db in double_brackets:
            if '|' in db:
                new = db.split('|')[-1].strip(']')
                text = text.replace(db, new)
            else:
                new = db.strip('[').strip(']')
                text = text.replace(db, new)
        after = len(text)
        if before == after:
            return text
```            

These functions worked-ish. You might notice the carat `^` in the curly function. This told it to match anything except another open curly bracket. This forced it to find the
innermost nested curly brackets. Again, this mostly worked, but it failed a few times and I gave up trying to figure out why. The square brackets are a bit different, as
they tend not to be nested but the inner syntax could be several different things. I opted for the simplest possible way and even so, it missed a few things. No idea why. 

## Attempt 1.5 - Literal Evals

I suppose I should rewind and give some context. Wikipedia dump files are effing huge. Even with 32GB of RAM on my desktop, I was rapidly running out of memory just 
loading one chunk at a time. So that meant I had to read each file line by line. Like so:

```python
with open(file, 'r', encoding='utf-8') as infile:
    for line in infile:
        line = literal_eval(f'"""{line}"""')  # this works... sometimes
        if '<page>' in line:  # new article
            article = ''
        elif '</page>' in line:  # end of article
            article += line
```

This works great for just reading the thing one at a time. One consistency in the Wikipedia dumps is that every page starts and ends with `<page>` and `</page>` respectively.
This served as a great demarcation. So I tried to handle the unicode literals as they were coming in with [ast.literal_eval](https://www.kite.com/python/docs/ast.literal_eval). 
Spoiler: It worked. A little bit. This function frequently bombs out for various reasons.

## Attempt 2 - Existing Parsers

I finally gave up on manually parsing WikiMedia Text and found some extant parsers. First up is [wikitextparser](https://pypi.org/project/wikitextparser/) which, as of this writing, is actively maintained.
Second up was the simple [html2text](https://pypi.org/project/html2text/) which got some of the stuff the first missed. These premade parsers are great in that they 
don't require me to use any of my own brain power! They are, however, far slower than my regex replace functions. It can't be avoided, though.

So now my output looks more like this:

```json
[
 {
  "id": "4413617",
  "text": "The Samajtantrik Sramik Front is a national trade union federation in Bangladesh. It is affiliated with the World Federation of Trade Unions...",
  "title": "Samajtantrik Sramik Front"
 },
 {
  "id": "2618",
  "text": "Aeacus (; also spelled Eacus; Ancient Greek: \u0391\u1f30\u03b1\u03ba\u03cc\u03c2 Aiakos or Aiacos) was a mythological king of the island of Aegina...",
  "title": "Aeacus"
 },
 {
  "id": "3201",
  "text": "[[File:Global_Temperature_And_Forces.svg|thumb|upright=1.35|right|Observed temperature from NASA. vs the 1850\u20131900 average used by the IPCC as a pre- industrial baseline.. The primary driver for increased global temperatures in the industrial era is human activity, with natural forces adding variability. Figure 3.1 panel 2, Figure 3.3 panel 5.]] Attribution of recent climate change is the ",
  "title": "Attribution of recent climate change"
 },
]
``` 

It's much cleaner and moving in the right direction but I still have to figure out the literal eval reliably and a few square brackets are making it through as well. 
These premade parsers are far slower but one advantage of cleaning up Wikipedia articles is that they end up far smaller without the markup. If you just want the accumulated 
knowledge in plain text format, it ends up being a fraction of the size. 

# Work Continues...

I can tolerate a few aberrations here and there but the perfectionist in me wants to do better. Anyways, here's my script as it stands today:

```python
import re
import os
import json
from uuid import uuid4
import gc
from html2text import html2text as htt
import wikitextparser as wtp


archive_dir = 'd:/WikipediaArchive/'
dest_dir = 'D:/enwiki20201020/'
chars_per_file = 40 * 1000 * 1000  # create a consistently sized chunk (~40MB each)


def dewiki(text):
    text = wtp.parse(text).plain_text()
    text = htt(text)
    text = text.replace('\\n',' ')
    text = re.sub('\s+', ' ', text)
    return text
    

def analyze_chunk(text):
    try:
        if '<redirect title="' in text:  # this is not the main article
            return None
        else:
            title = text.split('<title>')[1].split('</title>')[0]
            if ':' in title:  # this is a talk, category, or other (not a real article)
                return None
        serial = text.split('<id>')[1].split('</id>')[0]
        content = text.split('</text')[0].split('<text')[1].split('>', maxsplit=1)[1]
        content = dewiki(content)
        return {'title': title, 'text': content, 'id': serial}
    except:
        return None


def save_data(data):
    if len(data) == 0:
        return
    filename = dest_dir + str(uuid4()) + '.json'
    print('Saving:\t', filename)
    with open(filename, 'w', encoding='utf-8') as outfile:
        json.dump(data, outfile, sort_keys=True, indent=1)


def main(file):
    print(file)
    outdata = list()
    article = ''
    total_len = 0
    with open(archive_dir + file, 'r', encoding='utf-8') as infile:
        for line in infile:
            if '<page>' in line:  # new article begins
                article = ''
            elif '</page>' in line:  # end of article
                doc = analyze_chunk(article)
                if doc:
                    outdata.append(doc)
                    total_len += len(doc['text'])
                    if total_len >= chars_per_file:
                        save_data(outdata)
                        outdata = list()
                        total_len = 0
            else:
                article += line
    save_data(outdata)

    
if __name__ == '__main__':
    for file in os.listdir(archive_dir):
        if 'bz2' in file:
            continue
        main(file)
        gc.collect()
```

It's not the most elegant solution but for just over 100 lines of code, it will parse almost all of Wikipedia and save it to 40MB chunks of JSON.
Running this script looks like the following:

```bash
(base) C:\OfflineWikipedia>python jsonify_wikipedia.py
enwiki-20201020-pages-meta-current1.xml-p1p41242
Saving:  D:/enwiki20201020/95952e69-ed97-4807-bd47-f7019242a5dd.json
Saving:  D:/enwiki20201020/7f5c8f8f-bc73-4e92-9fe6-24be8076d71e.json
Saving:  D:/enwiki20201020/df3de04a-b088-4daa-94e5-6368d9c631a9.json
Saving:  D:/enwiki20201020/dc8360a0-4d0f-45cd-93c9-4530cf5a8265.json
Saving:  D:/enwiki20201020/0754c3c0-52ae-41f7-a5d5-2b2f1509b663.json
Saving:  D:/enwiki20201020/6eb66a52-3b49-47bb-a74f-c4b9e6f4578f.json
Saving:  D:/enwiki20201020/356f8242-fc36-4517-be32-7dd9e7af6307.json
Saving:  D:/enwiki20201020/83247fe5-223f-4a8b-a940-551cbc3cd21a.json
Saving:  D:/enwiki20201020/d8546da5-762c-48af-bba1-c8dacf92d257.json
Saving:  D:/enwiki20201020/4d25c46b-0d1d-4031-94f6-071c39fa91a5.json
Saving:  D:/enwiki20201020/511f5c91-6347-449d-a1a3-ce384988d497.json
Saving:  D:/enwiki20201020/cad2328c-dcaf-4e8e-bc6b-c68016e32ebc.json
enwiki-20201020-pages-meta-current10.xml-p4045403p5399366
Saving:  D:/enwiki20201020/79d4bd2b-4548-44e8-b7be-e3fe813272f2.json
Saving:  D:/enwiki20201020/baca8df6-8a6b-4c57-a3de-cce4cacceac0.json
Saving:  D:/enwiki20201020/919c6b37-b18f-46e5-917b-6e38553e7c26.json
...
```

Every article can be referenced by a UUID for the filename and an integer ID number. This means that all of Wikipedia can be easily indexed and is human readable. 
Of course there is some loss due to removing contextual information like links and pictures. But whatever. You still get >95% of the information. 
