---
layout: post
title: "Echo & Chamber Design & Architecture"
date: 2024-11-02
published: true
---

# Echo & Chamber - How It Works

## Project Overview

- Project: **Echo & Chamber**
- Aggregates headlines from **Fox News** and **MSNBC** to compare how each covers top stories.
- Inspired by similar comparison newsletters but designed to be shorter and more digestible (Morning Brew style).
- Goal: Provide a daily snapshot showing how partisan media frame the same news differently.
- Reader intent: Stay informed and entertained by seeing ideological spin from both sides.
- Personal context: User reads WSJ but wanted a lightweight way to glance at the extremes.

Hi, I am Mike, thanks for reading my first blog post.  I wanted a place to talk about some of the tech stuff and other things I work on so decided to create a blog to post about it.  This post is a summary of the Echo & Chamber project I work on that came from my research project at Georgia Tech.  

Echo & Chamber in its current form provides a daily comparison of how Fox News and MSNBC cover the same stories.  I personally enjoy getting most of my news from the Wall Street Journal (my happy place is reading it in the sauna at my gym), I also enjoy Morning Brew.  I believe the highly partisan news outlets are bad, however, many people consume them and I found myself skimming them often to see how each outlet reported the same topic.  I created Echo & Chamber primarily to automate that process for myself in a format that matched Morning Brew.  

At the time of writing this blog post the copy on echoandchamber.com is more general Left/Right "Break out of your echo chamber", but I am thinking I will change it to be more specific to Fox/MSNBC.  From a marketing perspective I am not sure telling people they are in an echo chamber is effective, I think instead I need marketing more centered around being interesting.  Fox News and MSNBC tend to call eachother the boogeyman, liberal people hate Fox News, conservative people hate MSNBC, but I am finding most people regardless of political lean do think the comparison is interesting.

---

## Connection to Georgia Tech Project

- Originally inspired by a Georgia Tech research project on **misinformation spread**.
- That project used **AI embeddings** and classic ML to model how topics evolved online.
- Echo & Chamber shares the *mission*, but is separate code-wise—more of a solo spinoff.

At Georgia Tech I took an "internet research" course that was about general monitoring of the internet, not just on content but also on hosting information and technical aspects.  The project we centered on was tracking the spread of "misinformation".  I enjoyed it, we built a tool to track stories by topic and then used predictive ML to analyze the hosting attributes to see if there were patterns in hosting about the same topic to identify malicious actors.  During my time at GT in most courses AI coding was cheating and given how valuable these tools are I wanted to try them more in depth after graduating.  Cursor had a free student license and seemed to be the main player so I used that.

---

## Personal Learning Goals

- Learn **AI coding**, especially embedding-based similarity.
- Gain hands-on experience with **Google Cloud Platform (GCP)**.
- Keep it low-lift and **low-maintenance**:
  - Max **15 minutes/day**
  - Minimal costs (GCP cost-conscious, no DB usage)
- Treat it like a creative outlet, not a chore.

My main goal with this project was/is to learn and get experience with tech I don't normally play with.  This included building something complex with Cursor and also deploying multiple connected services to GCP.  I needed it to be easy to operate once I built it, the goal being that the scraping of Fox and MSNBC is automated and I can just review/publish the newsletter.  AI has it's problems but the current models are very, very good at summarizing medium length articles.  This leads to hallucination not being a problem because all context is contained in each LLM call.  It also seemed like a fun way to think through a marketing exercise in how to attract subscribers.

---

## Architecture Overview

The project has 3 main components:

1. **Crawler**
2. **Processor (Generator)**
3. **Editor + Publishing Flow**

---

## Part 1: Crawler

- Runs hourly.
- Scrapes **top 20 stories** from Fox and MSNBC homepages.
- Ranks stories based on visual prominence on the homepage.
- Saves raw content to **Google Cloud Storage** (not a database).
- Storage structure = low-cost, file-based archive of daily news data.

The crawler is relatively straightforward, using python I read the homepage of each Fox News and MSNBC hourly and index the top 20 articles.  The idea here is that from a content perspective I am only interested in "top stories", and by creating an hourly index I can track the story prominence and also how long it is at the top of their page.  For each of the top 20 I copy the content into my index which can then be analyzed by the processor.

I built this using Cloud Functions in GCP triggered by a scheduler

---

## Part 2: Processor

- Uses **Sentence Transformers (Hugging Face)** for embeddings.
- Matches stories based on **semantic similarity** and **homepage rank**.
- Each day, creates:
  - **Matched Pairs** (Fox–MSNBC overlap)
  - **Blind Spots** (covered by one side only)
- “Blind Spot” doesn’t mean totally absent—more about what gets *emphasized* or *ignored*.
- Output is:
  - Fully structured **HTML newsletter**
  - Includes branding and layout for fast publishing.
- Optional future idea: use **React components** for visual explanations (e.g. similarity heatmaps).
- Might want to explain embeddings later in **non-technical terms** for readers.

The processor application does the bulk of the

### Embeddings, briefly

Most people meet AI through chatty encoder–decoder models. For matching headlines, we only need the encoder: it turns text into a vector of numbers (an “embedding”) that captures meaning. Similar texts end up as nearby vectors, so we can compare them using cosine similarity. This also enables neat vector arithmetic, like the classic example: `king − man + woman ≈ queen`.

1. LLMs often use an encoder/decoder; here we just use the encoder.
2. The encoder maps text → vector (hundreds of dimensions) capturing semantics.
3. We measure closeness with cosine similarity; closer = more related.
4. Vector offsets capture relations, e.g., `king − man + woman ≈ queen`.

<figure class="embedding-figure" style="margin: 20px 0;">
  <svg viewBox="0 0 660 360" width="100%" height="auto" role="img" aria-label="Vector analogy: king minus man plus woman equals queen">
    <defs>
      <marker id="arrow" viewBox="0 0 10 10" refX="7" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
        <path d="M 0 0 L 10 5 L 0 10 z" fill="#6b47dc" />
      </marker>
      <marker id="arrow2" viewBox="0 0 10 10" refX="7" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
        <path d="M 0 0 L 10 5 L 0 10 z" fill="#2a7" />
      </marker>
      <style>
        .axis { stroke: #e6e6e6; stroke-width: 1; }
        .label { fill: #757575; font-size: 12px; font-family: -apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,Helvetica,Arial,sans-serif; }
        .point { stroke: #292929; stroke-width: 1; }
        .ptext { fill: #292929; font-size: 14px; font-family: -apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,Helvetica,Arial,sans-serif; }
        .v1 { stroke: #6b47dc; stroke-width: 2.5; marker-end: url(#arrow); }
        .v2 { stroke: #2a7; stroke-width: 2.5; marker-end: url(#arrow2); }
        .hint { fill: #757575; font-size: 13px; font-family: -apple-system,BlinkMacSystemFont,'Segoe UI',Roboto,Helvetica,Arial,sans-serif; }
        .dash { stroke-dasharray: 6 4; }
      </style>
    </defs>

    <!-- Axes -->
    <line x1="40" y1="300" x2="620" y2="300" class="axis" />
    <line x1="40" y1="40" x2="40" y2="300" class="axis" />
    <text x="622" y="304" class="label">x</text>
    <text x="30" y="46" class="label">y</text>

    <!-- Points -->
    <!-- Coordinates chosen for clarity (not real embeddings) -->
    <circle cx="140" cy="240" r="5" fill="#fff" class="point" />
    <text x="150" y="244" class="ptext">man</text>

    <circle cx="320" cy="160" r="5" fill="#fff" class="point" />
    <text x="330" y="164" class="ptext">king</text>

    <circle cx="160" cy="140" r="5" fill="#fff" class="point" />
    <text x="170" y="144" class="ptext">woman</text>

    <circle cx="340" cy="60" r="5" fill="#fff" class="point" />
    <text x="350" y="64" class="ptext">queen</text>

    <!-- Vector king - man -->
    <line x1="140" y1="240" x2="320" y2="160" class="v1" />
    <text x="210" y="188" class="hint">king − man</text>

    <!-- Apply same offset to woman → queen -->
    <line x1="160" y1="140" x2="340" y2="60" class="v2" />
    <text x="205" y="110" class="hint">+ woman → queen</text>

    <!-- Dotted helpers to show parallelogram idea -->
    <line x1="320" y1="160" x2="340" y2="60" class="axis dash" />
    <line x1="140" y1="240" x2="160" y2="140" class="axis dash" />

    <!-- Caption -->
    <text x="40" y="330" class="hint">Analogy in vector space: king − man + woman ≈ queen</text>
  </svg>
</figure>

---

## Part 3: Editing + Publishing

- Built a **custom editor** for editing the final HTML.
  - Not raw HTML—editor supports quick changes to layout/headlines.
- Beehiv publishing step is **manual** (due to API limitations on non-enterprise plans).
- Daily routine:
  - 8 PM: Generate newsletter
  - Quick read-through to catch mismatches (1–2 max usually)
  - Paste into Beehiv and send
- Total time: **10–15 minutes**
- Designed to feel fun, not like work.

---

