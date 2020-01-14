---
templateKey: blog-post
title: Intro to the 4Chan JSON API
date: 2020-01-14T18:48:14.953Z
description: >-
  For all the controversy surrounding 4chan since, well... the start of 4chan I
  suppose, im surprised at the lack of blog posts on their JSON API.


  Yes! that's right, 4chan has a read only JSON API packed full of all kinds of
  de-moralizing, soul crushing possibilities.


  Yet for whatever reason this goldmine of strange insights is almost
  non-existant in the commentaries of the internet.


  If we search *'4chan JSON API python'* in duckduckgo or google, we get almost
  no blog post related hits.


  In fact we get so few hits, one could get conspiratorial in coming up with
  reasons about how this could be.
featuredpost: true
featuredimage: /img/blog_post_cover.jpg
tags:
  - 4chan
  - JSON
  - API
---
<link rel="stylesheet"
      href="//cdn.jsdelivr.net/gh/highlightjs/cdn-release@9.17.1/build/styles/monokai.min.css">
      
<script src="//cdn.jsdelivr.net/gh/highlightjs/cdn-release@9.17.1/build/highlight.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>

**Step 6:**
---
Now lets get the comments for each post.

They are stored in a key called **['last_replies']**.

This is tricky because not all threads have comments, and if they do, they're always being updated from the time the post is created until it's archived.

To do this I chose the [.get()](https://docs.python.org/3.7/library/stdtypes.html#mapping-types-dict) built-in dictionary method so we dont receive any [KeyErrors](https://docs.python.org/3/library/exceptions.html) in the event a key is not present, because there are no comments on the thread yet.

Fyi, this is a little helper function I made incase I want to change our extraction method later on.
```python
def get_threads(key: str, default='NaN'):
    return threads.get(key, default)
```

Here is the resulting addition to the code.

```python
import requests
from datetime import datetime as dt

r = requests.get('https://a.4cdn.org/pol/catalog.json')
r = r.json()

def gen_chan():
    for idx, page in enumerate(r):
        for thread in r[idx]['threads']:
            yield thread

def get_threads(key: str, default='NaN'):
    return threads.get(key, default)
    
for threads in gen_chan():
    no = get_threads('no')
    now = get_threads('now')
    time = get_threads('time')
    my_time = dt.today()
    com = TextMaster.strip_html(get_threads('com'))
    name = get_threads('name')
    trip = get_threads('trip')
    ids = get_threads('id')
    capcode = get_threads('capcode')
    filename = get_threads('filename') + get_threads('ext')
    resto = get_threads('resto')
    semantic_url = get_threads('semantic_url')
    replies = get_threads('replies')
    images = get_threads('images')
    url = TextMaster.extract_url(get_threads('com'))
    sent = TextMaster.textblob_sentiment(get_threads('com'))
    if 'last_replies' in threads:
        for comment in threads['last_replies']:
            com_com = TextMaster.strip_html(comment.get('com', 'NaN'))
            resto_com = comment.get('resto', 'NaN')
            now_com = comment.get('now', 'NaN')
            time_com = comment.get('time', 'NaN')
            filename_com = comment.get('filename', 'NaN') + comment.get('ext', 'NaN')
            # If your wondering what TextMaster is, check back for my follow up article.
            url_com = TextMaster.extract_url(comment.get('com'))
            sent_com = TextMaster.textblob_sentiment(comment.get('com'))
```

For this article that pretty much sums it up.

We can now build a NLP pre-processing pipeline around it and insert it into a database.

