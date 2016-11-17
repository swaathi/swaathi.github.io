---
layout: post
title: How you can read a 100 responses in 30 minutes
date: 2017-11-17 16:30:00 +5:30 GMT
share: y
---

When we built the responses module of [UnderstandBetter](https://understandbetter.co) we focused on making it the best experience for employees. The ease of typing in answers, editor features that can make expressing emotions better and providing a clean interface were our main goals. For a long time we equated a great answer to the length of it. Worthless, I know (we do have spam filters though!).

<!--break-->

But we never thought about the CEOs or the HRs who are going to spend a considerable amount of time reading (skimming) through these responses. A 100 responses! Even if you manage to read them all, it's quite hard to relate each answer to a user and give them appropriate feedback.

At Skcript, we have a company wide belief that technology's best use is increasing the productivity of humans. We knew it isn't efficient for a human to skim through a 100 longform responses and convert them into actionable goals. So that's when we called upon our little AI friend the UnderstandEngine.

<img src="/public/posts/2017-11-17/understand_engine.jpeg" class="img" alt="UnderstandEngine Logo" />

When an answer which is of considerable length hops by us, we parse just about everything we can about it.

### Automatic Summary Generation

This has been by far the most used feature by CEOs and HRs. Based on the context of the question we extract a gist of the users response. The context of the question helps the algorithm best guess what the most important phrase of the answer could be.

Here's how it looks like in action,

<img src="/public/posts/2017-11-17/automatic_summary.jpeg" class="img" alt="Automatic Summary Generation with UnderstandEngine" />

### Keyword Extraction

Humans are lazy sometimes. In a few cases, we found that summaries might not be the best way to derive context. Keyword extraction saves the day in these places. An overview of the keywords used can be a quicker way to understand the overall emotion of the response.

<img src="/public/posts/2017-11-07/keyword_extraction.jpg" class="img" alt="Keyword Extraction with UnderstandEngine" />

### Polarity and Subjectivity Analysis

Never mind humans are lazy all the time! What's even quicker than a bunch of words? A score of course! Polarity is a measure of the sentiment of a response. A score of 10 means that it is completely positive, while a score of 0 means that it is completely negative. We found out that sometimes this may not be meaningful in every scenario. For a question like "What is the vision of the company in your own words?" a sentiment doesn't really make sense. Most responses would default to a 5, a neutral, which makes sense. In such cases we use a subjectivity analysis, which gives us an insight on how "personally" you've answered the question. Did you use a lot of "I"s, or a lot of feeling words? Then you would get a a higher subjectivity score.

There are over 20 data points that we process, and 15 actionable metrics UnderstandEngine quantifies to help CEOs and HRs make more accurate decisions and get to "understand" their company a lot more "better"! (How cool is that nameplay?)

---

We really hope you enjoy using UnderstandBetter, we love perfecting this platform. If you would like to give a test drive with UnderstandBetter, just send us an email emoji. (Literally an email with just 👆🏼 emoji!) If you're in Chennai, we would love to meet you, coffee and cake is on us! If not, we love Skype, Hangouts and Facetime.

Have a great week.
