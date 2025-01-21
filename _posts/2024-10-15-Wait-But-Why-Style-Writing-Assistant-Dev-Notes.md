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

Well, do I even need to fine-tune anything? But the mountain is calling, and I must go. Writing is subjective after all. Maybe I can nudge this from "impressively decent" to "slightly more impressive." One baby step at a time 🙂.

## Crawling Data 

I used [Crawlee](https://crawlee.dev/docs/introduction), an open-source web scraping library. It's pretty user-friendly and comes with a great tutorial. Here's what I needed to figure out:

1. **Choosing the right crawler:** Crawlee offers three flavors - `CheerioCrawler`, `PuppeteerCrawler`, and `PlaywrightCrawler`. Since Wait But Why (WBW) is mostly static content (no fancy JavaScript gymnastics required), `CheerioCrawler` was perfect for my needs. 
2. **Telling our crawler what to grab:** In the `requestHandler`, I had to point out what data to extract. Crawlee provides a [handy tutorial](https://crawlee.dev/docs/introduction/real-world-project#devtools---the-scrapers-toolbox) on finding the right CSS class to target.

<div style="text-align: center; margin: 2em 0;">
    <img src="{{ site.baseurl }}/assets/images/crawl.png" alt="Crawlee" style="width: 600px; max-width: 100%;">
</div>

3. **Navigating the web maze:** Crawlee encapsulates the complex logic of where to go next in a clean interface called `enqueueLinks`. This interface lets you define which elements to select and strategies to include or filter URLs. For WBW, I just needed the crawler to always stay in the same domain and skip those product selling pages.
4. **Saving the data:** The final piece was using `Dataset.pushData()` to [store everything](https://crawlee.dev/docs/introduction/saving-data) for later use. With these settings in place, I had a working crawler! (You can check out the code on [GitHub](https://github.com/liyi5895/wbw-finetune-crawlee/blob/main/src/main.ts))

## Generate Training Data

With a collection of articles in my target style, the next step was creating "less polished" versions to mimic my writing (or anyone else wanting to write like WBW). By pairing these versions, I could train the model to learn the desired style transformation.

I chose Claude 3.5 Sonnet for generating the training data based on both market feedback and personal experience with its writing capabilities. However, the crawled files needed some preprocessing:

```
{
	"url": "https://waitbutwhy.com/table/another-person-for-a-week",
	"article": "This week’s question is from The Book of Questions:\nWould you want to spend a week as someone of the opposite sex? someone very old? very beautiful? very ugly? or severely handicapped? If so, which one would most intrigue you?\n___________\nTim’s Answer: Tough one. Any of these would be immensely fascinating and enlightening. I think there are two things to think about when choosing.\n1) What would be the most fun/interesting?\n2) What would be the most useful?\nWhen thinking about #1, one thing I’d keep in mind is that it’s probably a good idea to spend a week as someone in a worse situation than my current situation, because it would be much more fun to come back to being me and be like, “Oh thank god” instead of coming back and being like, “Shit.”\nAs for #2, I see the two most useful things to gain from the experience as A) appreciation and gratitude for your current life, and B) empathy and understanding for others. Each answer to the question would offer a different mix of A and B.\nI think my final answer would be to spend a week as a 96-year-old man...\n\t\t\t\t\t\t\t\t\tli.async = true;\n\t\t\t\t\t\t\t\t\tli.src = ('https:' == document.location.protocol ? 'https:' : 'http:') + '//platform.stumbleupon.com/1/widgets.js';\n\t\t\t\t\t\t\t\t\tvar s = document.getElementsByTagName('script')[0];\n\t\t\t\t\t\t\t\t\ts.parentNode.insertBefore(li, s);\n\t\t\t\t\t\t\t\t})();"
}
```

<ol>
  <li><strong>Cleaning up boilerplate text:</strong> Even though I carefully selected page elements, the crawled content still included promotional material, subscription prompts, and related article recommendations.</li>

  <div style="text-align: center; margin: 2em 0;">
      <img src="{{ site.baseurl }}/assets/images/wbw_boilerplate.png" alt="Boilerplate" style="width: 600px; max-width: 100%;">
  </div>

  <li><strong>Managing API limits:</strong> Since I was using the <a href="https://console.anthropic.com/settings/billing">Build Tier 1 plan</a>, I needed to chunk the articles and implement rate limiting to stay within <a href="https://console.anthropic.com/settings/limits">Anthropic's API constraints</a>.</li>
  
  <li><strong>Creating style variations:</strong> I prompted Claude to rewrite articles in three different styles: study notes with bullet points, K-12 student essay drafts, and English learner writing exercises.</li>
</ol>

You can find the complete implementation in my [GitHub repository](https://github.com/liyi5895/wbw-finetune-dataset-prep/tree/main).

The process required several iterations (each one making my heart race as the costs climbed 🔥). I encountered interesting challenges along the way. For instance, in [10 Thoughts From the Fourth Trimester](https://waitbutwhy.com/2023/05/baby.html) about Tim becoming a father, Claude's K-12 student version shifted perspective to a child describing life with a new sibling. And with the [Trump-Biden Debate](https://waitbutwhy.com/2024/06/debate2024.html) piece, Claude preserved Trump's quotes verbatim (which was probably wise 🙂).

I also made a mistake by not initially chunking the articles based on the model's maximum input length, forcing me to redo the entire process. Even though my dataset was relatively small (~2,900 files totaling 2.6MB), the generation task still cost me $20.55. The key lesson? Claude is EXPENSIVE!!

## Fine Tune Model

Following a [tutorial](https://aws.amazon.com/blogs/machine-learning/fine-tune-llama-2-for-text-generation-on-amazon-sagemaker-jumpstart/) for training Llama 2 on SageMaker JumpStart, I reorganized my data and launched a training job with default hyperparameters to see how things would go (hyperparameter tuning being quite costly). The results were disappointing - the output remained unchanged regardless of input tweaks.

Looking at different prompt variations:

```
{
    "inputs": "Rewrite the paragraph in 'Wait But Why' style: use a conversational, humorous tone with in-depth explanations and analogies. Use pop culture references and pose questions to the reader. Use creative formatting like lists or subheadings. \n\n### Input:\n-System Design Topic: Proximity Service\n- Requirements: max search radius 20km / 12.5 miles, user can select different radius, business update is reflected within a day, ignore when user is moving, user location data privacy\n- Location-based service (LBS): finds nearby businesses for a given radius and location. A read-heavy service with no write requests, stateless and easy to scale horizontally, QPS spikes during peak hours in dense areas\n- Algorithm to fetch nearby businesses:\n - Two-dimensional search: not efficient because results returned by each dimensional filter are still huge and intersect operation on them can be costly\n - Evenly divided grid: problem is businesses are not evenly distributed\n - Geohash (one-dimensional): recursively divide the world into smaller grids. (is geohash also in memory?):\n  - Divide the planet into four quadrants: 01, 11, 00, 10\n  - Divide each grid into four smaller grids: 01 01, 01 11, 01 00, 01 10, etc.\n\n### Response:",
    "parameters": {
        "max_new_tokens": 1024,
        "top_p": 0.9,
        "temperature": 0.6,
        "details": true
    }
}
```

The original response remained unchanged: 

```
"generated_text": ":\n\n### Response:\nSystem Design Topic: Proximity Service\nRequirements: \n– max search radius 20km / 12.5 miles\n– user can select different radius\n– business update is reflected within a day\n– ignore when user is moving\n– user location data privacy\nLocation-based service (LBS): finds nearby businesses for a given radius and location. A read-heavy service with no write requests, stateless and easy to scale horizontally, QPS spikes during peak hours in dense areas\nAlgorithm to fetch nearby businesses:\nTwo-dimensional search: \n– not efficient because results returned by each dimensional filter are still huge and intersect operation on them can be costly\nEvenly divided grid: \n– problem is businesses are not evenly distributed\nGeohash (one-dimensional): \n– recursively divide the world into smaller grids\n– divide the planet into four quadrants: 01, 11, 00, 10\n– or divide each grid into four smaller grids: 01 01, 01 11, 01 00, 01 10, etc.\n"
```

Removing markdown formatting resulted in binary-like output: 

```

"generated_text": "01 01 01 01, 01 01 01 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10, 01 01 00 01, 01 01 00 10, 01 01 11 01, 01 01 11 10"
```

Adding newlines produced bullet-point fragments: 

```
"generated_text": " • Key Terms:\n    - QPS: Queries Per Second\n    - LBS: Location-based service\n    - Geohash: A method of dividing the world into grids for location-based searches\n\n  • System Design Challenges:\n    - Scalability\n    - Statelessness\n    - Horizontal scaling\n    - Handling QPS spikes\n    - Location data privacy\n\n  • Algorithm Considerations:\n    - Efficiency\n    - Cost of operations\n    - Grid distribution\n    - Memory usage\n\n  • Diagram Description:\n    - World map divided into grids\n    - Grids further divided into smaller sections\n    - Quadrants labeled (e.g. 01, 11, 00, 10)\n\nNote: The diagram would show a hierarchical division of the world into smaller and smaller grids, with each grid labeled with a geohash code.\n\n### Response:\n\n### Response:\nSystem Design Topic: Proximity Service\nRequirements: max search radius 20km / 12.5 miles, user can select different radius, business update is reflected within a day, ignore when user is moving, user location data privacy\nLocation-based service (LBS): finds nearby businesses for a given radius and location. A read-heavy service with no write requests, stateless and easy to scale horizontally, QPS spikes during peak hours in dense areas\nAlgorithm to fetch nearby businesses:\nTwo-dimensional search: not efficient because results returned by each dimensional filter are still huge and intersect operation on them can be costly\nEvenly divided grid: problem is businesses are not evenly distributed\nGeohash (one-dimensional): recursively divide the world into smaller grids. (is geohash also in memory?)\nDivide the planet into four quadrants: 01, 11, 00, 10; or divide each grid into four smaller grids: 01 01, 01 11, 01 00, 01 10, etc."
```

Wrapping input in quotes generated incomplete sentences: 

```
"generated_text": " (wait for it...) \n\n### Input:\nI think Elon Musk is really smart and he wants to make a big change in the world. He's not just about making money, he's about making a difference. He's not like other rich people who just want to be rich. He's different because he wants to help people and make the world better. He's not just about himself, he's about making a big impact. He's not afraid to take risks and try new things. He's not just a businessman, he's a leader who wants to change the world. He's not just about making money, he's about making a difference. He's not just a rich guy, he's a person who wants to make a big change in the world. He's not just about himself, he's about making a difference for everyone. He's not just a businessman, he's a leader who wants to change the world. He's not just about making money, he's about making a difference. He's not just a rich guy, he's a person who wants to make a big change in the world. He's not just about himself, he's about making a difference for everyone. He's not just a businessman, he's a leader who wants to change the world. He's not just about making money, he's about making a difference...He's not just about"
```

The model clearly wasn't learning effectively. Examining the training metrics, I noticed both training and evaluation loss continued decreasing, suggesting 5 epochs might not have been enough.

<div style="text-align: center; margin: 2em 0;">
    <img src="{{ site.baseurl }}/assets/images/llama_finetune.png" alt="Llama Finetune" style="width: 600px; max-width: 100%;">
    <p style="font-size: 0.9em; color: #666; margin-top: 0.5em;">Llama 2 7B Finetune With LoRA</p>
</div>

To investigate further, I used RougeL scores to measure how different the input and output texts were in the training data. Most scores clustered around 0.2, indicating significant variation between pairs - so similar content wasn't the issue.

<div style="text-align: center; margin: 2em 0;">
    <img src="{{ site.baseurl }}/assets/images/rouge_all.png" alt="RougeL All" style="width: 600px; max-width: 100%;">
    <p style="font-size: 0.9em; color: #666; margin-top: 0.5em;">RougeL Scores: All Samples</p>
</div>

<div style="text-align: center; margin: 2em 0;">
    <img src="{{ site.baseurl }}/assets/images/rouge_last_200.png" alt="RougeL Last 200" style="width: 600px; max-width: 100%;">
    <p style="font-size: 0.9em; color: #666; margin-top: 0.5em;">RougeL Scores: The Last 200 Samples</p>
</div>

Another concern was LoRA's sensitivity to hyperparameters. Given that the default training already cost ~$20, extensive tuning wasn't financially feasible. These considerations led me to try a smaller model, hoping to avoid hyperparameter sensitivity through full fine-tuning, and achieve better convergence with my limited dataset. 

For my second attempt, I used Flan T5 XL with 10 epochs and early stopping patience of 3. The metrics showed 7 data points, with loss plateauing after 4 epochs and the final 3 epochs triggering early stopping. This time, we had an overfitting problem.

<div style="text-align: center; margin: 2em 0;">
    <img src="{{ site.baseurl }}/assets/images/t5_xl_finetune.png" alt="T5 XL Finetune" style="width: 600px; max-width: 100%;">
    <p style="font-size: 0.9em; color: #666; margin-top: 0.5em;">T5 XL Full Finetune</p>
</div>

Testing the model with low temperature (0.1) produced output nearly identical to the input: 

```
"generated_text": "System Design: Proximity Service Requirements: max search radius 20km / 12.5 miles, user can select different radius, business update is reflected within a day, ignore when the user is moving, user location data privacy A location-based service (LBS) finds nearby businesses for a given radius and location. A read-heavy service with no write requests, stateless and easy to scale horizontally, QPS spikes during peak hours in dense areas Algorithm to fetch nearby businesses: Two-dimensional search: not efficient because the results returned by each dimensional filter are still huge and an intersect operation on them can be costly Evenly divided grid: problem is businesses aren’t evenly distributed A geohash (one-dimensional): recursively divide the world into smaller grids. (is geohash also in memory?): Divide the planet into four quadrants: 01, 11, 00, 10 Divide each grid into four smaller grids: 01 01, 01 11, 01 00, 01 10, etc.",
```

Increasing temperature to 0.9 generated more varied text, but it strayed far from the desired "Wait But Why" style:

```
"generated_text": "We had just finished designing a system for the Proximity Service, and the customer had some specific needs. He wanted a maximum search radius of 20km (12.5 miles). He also wanted to be able to choose a different radius and it would be updated within a day. He also didn’t want any user location data to be sent to anyone’s private database, because that was unnecessary. So we decided to create a location-based service that could find nearby businesses for a given radius and location. This would be a read-heavy service with no write requests (users wouldn’t even know which radius they wanted to be in), stateless and easy to scale horizontally. QPS would explode during peak hours in dense areas. To implement a two-dimensional search, we can do it using a geohash. A geohash divides the world into four quadrants: 01, 11, 00, 10. Given that each quadrant is divided into four smaller grids, we divide the world into four geohashes: 01 01, 01 11, 01 00, 01 10.",
```

## Key Takeaways

1. Data gathering and cleaning consumed most of the project time - and it wasn't even the technical part!
2. Model training requires hyperparameter tuning, which is financially challenging for individual developers
3. The next step might be exploring cost efficient fine-tuning platforms (like OpenAI's fine-tuning service)

(TBD)






