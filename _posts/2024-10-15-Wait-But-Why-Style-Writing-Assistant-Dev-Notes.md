---
layout: post
title: "Wait But Why Style Writing Assistant Dev Notes"
date: 2024-10-15
categories: [ai, 2024]
---

TL;DR: The project is temporarily paused due to financial constraints. (Let's just say my wallet is more "wait" than "why" right now 😅)

<style>
  hr:before, hr:after {
    content: none !important;
  }
</style>

<hr style="margin: 2em 0; border-top: 1px solid #ccc;">

As a first-generation immigrant who spent most of my alone time in bookstores back in my home country, and as a software developer who initially entered the industry for sponsorship, writing in English at the same proficiency level as my native language has been a farfetched dream for the past decade.

AI made this dream seem attainable. In 2023, I had the idea to train my own AI writing assistant (or should I say, tutor) so that I could masquerade as an Americanized writer or blogger. The plan was to hide behind the machine while secretly building my own skills to navigate a cross-cultural writing journey.

That's probably more backstory than you bargained for. Practically speaking, many chatbots on the market can already meet this need. However, I also wanted to use this opportunity to learn some cool tech stuff. So, here's a blog about how I built the first version of an AI writing assistant, which transforms my immature, language learner style of writing into that of one of my favorite blogs: Wait But Why (https://waitbutwhy.com/).

I have very limited knowledge of how to fine-tune a foundation model. So here's my somewhat naive training plan for my first grand experiment:

1. Crawl all the blogs from WBW; ask ChatGPT if there are any legal concerns (because who doesn't love a good chat about intellectual property rights?)
2. Ask Claude to rewrite WBW blogs as poorly as my writing to form the training data (it's like asking a master chef to burn toast)
3. Fine-tune Llama or T5 model (because they're popular 🙂) on AWS SageMaker (because I’m lazy 🙂) 
4. Enjoy my almighty writing assistant and kick off my journey to becoming a great writer (or at least a slightly less confused one)

What follows is a detailed account of the entire process. I hope you'll find it interesting, even if it's just a humble attempt at navigating the world of AI and language learning.

## Evaluating the Baseline

Being a thrifty developer (read: broke), I turned to [Perplexity AI](https://labs.perplexity.ai/?ref=hackernoon.com) to test out Llama 3.1 8B Instruct (it’s free). To my surprise, the results were impressively decent:

Here's the prompt I used:

```
Rewrite the paragraph in 'Wait But Why' style: use a conversational, humorous tone with in-depth explanations and analogies. Use pop culture references and pose questions to the reader. Use creative formatting like lists or subheadings.

Input:
- System Design Topic: Proximity Service
- Requirements: max search radius 20km / 12.5 miles, user can select different radius, business update is reflected within a day, ignore when user is moving, user location data privacy
- Location-based service (LBS): finds nearby businesses for a given radius and location. A read-heavy service with no write requests, stateless and easy to scale horizontally, QPS spikes during peak hours in dense areas
- Algorithm to fetch nearby businesses
 - Two-dimensional search: not efficient because results returned by each dimensional filter are still huge and intersect operation on them can be costly
 - Evenly divided grid: problem is businesses are not evenly distributed
 - Geohash (one-dimensional): recursively divide the world into smaller grids. (is geohash also in memory?)
  - Divide the planet into four quadrants: 01, 11, 00, 10
  - Divide each grid into four smaller grids: 01 01, 01 11, 01 00, 01 10, etc.

Response:
```

And here's what the AI cooked up:

```
The Proximity Service: A Location-Based Service That's Not as Easy as It Sounds

Imagine you're at a music festival, and you want to find the nearest food truck that serves vegan options. You open your phone, and voilà A list of nearby food trucks appears, complete with their addresses and menus. But have you ever wondered how your phone managed to find those food trucks in the first place? That's where the Proximity Service comes in – a location-based service (LBS) that's responsible for finding nearby businesses for a given radius and location.

The Requirements: A Tall Order
So, what does this service need to do? Well, for starters, it needs to be able to search within a radius of up to 20km (12.5 miles). But here's the thing: users can select different radii, so the service needs to be flexible. And, because businesses are constantly updating their information, the service needs to reflect these changes within a day. Oh, and one more thing: when you're on the move, the service should ignore your location data to preserve your privacy. Easy peasy, right?
The Algorithm: A Geohash-ing Good Time
To fetch nearby businesses, the Proximity Service uses a clever algorithm called Geohash. It's like a game of Tetris, but instead of blocks, you're dividing the world into smaller and smaller grids. Here's how it works:

Geohash 101
1. Divide and Conquer: The world is divided into four quadrants: 01, 11, 00, 10. Think of it like a giant game of Risk, but with latitude and longitude instead of armies.
2. Recursion: Each grid is then divided into four smaller grids: 01 01, 01 11, 01 00, 01 10, etc. It's like a never-ending game of Tetris, but with grids instead of blocks.
3. One-Dimensional Search: By using Geohash, the service can perform a one-dimensional search, which is much faster than a two-dimensional search. It's like searching for a specific book in a library, instead of searching through every book on every shelf.

But Wait, There's a Problem
The problem with Geohash is that it's not perfect. Businesses are not evenly distributed, so the grid system can be a bit wonky. Imagine trying to find a specific book in a library, but the books are stacked haphazardly on the shelves. It's not exactly the most efficient system.

Two-Dimensional Search: The Not-So-Efficient Option
For comparison, let's look at a two-dimensional search. It's like searching through every book on every shelf in the library. The results returned by each dimensional filter are still huge, and the intersect operation on them can be costly. It's like trying to find a needle in a haystack, but the haystack is on fire.

The Verdict
So, there you have it – the Proximity Service and its clever use of Geohash. It's a read-heavy service with no write requests, stateless and easy to scale horizontally. But, as with any system, there are trade-offs. QPS spikes during peak hours in dense areas, and the grid system can be a bit wonky. But hey, at least it's not as bad as trying to find a specific book in a library with no cataloging system.
```

Well, do I even need to fine-tune anything? 

## Crawling Data 

I used [Crawlee](https://crawlee.dev/docs/introduction), an open-source web scraping library. It's pretty user-friendly and comes with a great tutorial. Here's what I needed to figure out:

1. **Choosing the right crawler:** Crawlee offers three flavors - `CheerioCrawler`, `PuppeteerCrawler`, and `PlaywrightCrawler`. Since Wait But Why (WBW) is mostly static content (no fancy JavaScript gymnastics required), `CheerioCrawler` was perfect for my needs. 
2. **Telling our crawler what to grab:** In the `requestHandler`, I had to point out what data to extract. Crawlee provides a [handy tutorial](https://crawlee.dev/docs/introduction/real-world-project#devtools---the-scrapers-toolbox) on finding the right CSS class to target.

<div style="text-align: center; margin: 2em 0;">
    <img src="{{ site.baseurl }}/assets/images/crawl.png" alt="Crawlee" style="width: 600px; max-width: 100%;">
</div>

3. **Navigating the web maze:** Crawlee encapsulates the complex logic of where to go next in a clean interface called `enqueueLinks`. This interface lets you define which elements to select and strategies to include or filter URLs. For WBW, I just needed the crawler to always stay in the same domain and skip those product selling pages.
4. Finally, `Dataset.pushData()` is needed to [save data](https://crawlee.dev/docs/introduction/saving-data) for future use. And that concludes all the necessary settings for Crawlee. Easy right :D



