---
layout: post
title: "Echo & Chamber - How It Works"
date: 2025-11-02
published: true
---

What's up - thanks for reading my first blog post. I wanted a place to talk about some of the tech stuff and other things I work on & so this blog was born. This first post is a summary of the [Echo & Chamber](https://echoandchamber.com) project I work on that came from my research project at Georgia Tech.

<p align="center">
  <img src="/assets/images/firstpost/echo-chamber-logo.svg" alt="EC Logo" width="400">
</p>

<div style="background: #f5f5f5; border: 1px solid #e0e0e0; border-radius: 8px; padding: 20px; margin: 30px 0;">
<h3 style="margin-top: 0; color: #2c3e50;">Table of Contents</h3>
<ol>
<li><a href="#project-overview">Project Overview</a></li>
<li><a href="#connection-to-georgia-tech-project">Connection to Georgia Tech Project</a></li>
<li><a href="#personal-learning-goals">Personal Learning Goals</a></li>
<li><a href="#architecture-overview">Architecture Overview</a></li>
<li><a href="#part-1-crawler">Part 1: Crawler</a></li>
<li><a href="#part-2-processor">Part 2: Processor</a>
<ul>
<li><a href="#article-embeddings">Article Embeddings</a></li>
<li><a href="#clustering">Clustering</a></li>
<li><a href="#story-prioritization">Story Prioritization</a></li>
<li><a href="#story-generation">Story Generation</a></li>
</ul>
</li>
<li><a href="#part-3-editing--publishing">Part 3: Editing + Publishing</a></li>
</ol>
</div>

## Project Overview

Echo & Chamber in its current form gives a daily side-by-side look at how Fox News and MSNBC cover the same stories. For context, I'm a Wall Street Journal person (my happy place is reading it in a sauna), & I also like Morning Brew. I generally skip partisan cable news, but a ton of people consume those brands, and it has a material impact on political perception. So Echo & Chamber started as a way for me to quickly see how each side frames the same event. Not to doomscroll or argue, but to understand the lens other people are looking through.

At the time of writing this post, the copy on echoandchamber.com still uses the broader "Left vs. Right, Break out of your echo chamber" framing. I'm thinking of changing that to a more specific Fox vs. MSNBC focus. From a marketing perspective, I'm not sure telling people they're in an echo chamber is the best approach. I think it would be better to lean more into curiosity and interest. Fox News and MSNBC often treat each other as the boogeyman, liberals hate Fox & conservatives hate MSNBC. But I'm finding that regardless of political lean people do find the short Fox/MSNBC comparison digest an interesting way to stay up to date on misc political events.  The goal is really to be just a short digest of political events specifically through the lense of partisanship in an entertaining way.

---

## Connection to Georgia Tech Project

At Georgia Tech I took a course on [Internet Research](https://omscs.gatech.edu/cs-8803-o23-modern-internet-research-methods), which focused on monitoring the internet from both a content and technical perspective. The project my team did centered on tracking the spread of misinformation. I really enjoyed it, we built a tool to track stories by topic and then used predictive ML to analyze hosting attributes, looking for patterns that could help identify malicious actors.

<p align="center">
  <img src="/assets/images/firstpost/gt.png" alt="Mike" width="250"><br>
  <span style="font-size: 0.9em; color: #666;">
    Graduation day at GT
  </span>
</p>



During my time at GT, most courses considered AI coding tools to be cheating. I think people not in the computer science world might be surprised that a CS degree is equal parts math and there is a lot that does not directly translate to actual software engineering. In the program I learned a **lot** about the math of machine learning / neural networks but actually very little about how to use the new AI coding tools that are coming out. So for me, I had this political news aggregator idea & I wanted to learn about coding with AI, there was a free student license for Cursor, the stars seemed to align so I went for it on the idea.

---

## Personal Learning Goals

My main goal with this project was to learn and get experience with tech I don't normally play with. This included building something complex with Cursor and also deploying multiple connected services to GCP. I needed it to be easy to operate once I built it, the goal being that the scraping of Fox and MSNBC is automated and I can just review/publish the newsletter. AI has its problems but the current models are very, very good at summarizing medium length articles. This leads to hallucination not being a problem because all context is contained in each LLM call. It also seemed like a fun way to think through a marketing exercise in how to attract subscribers.

In terms of learning goals I'd say the project has already been a success. I am planning to write a future blog post about using Cursor and my experience AI coding past a POC. If I could redo the build on this project no doubt it would be way cleaner & better, but I couldn't have figured that out without actually doing it. It has also been interesting to go deeper on using GCP and designing with cost management top of mind (everything with this runs me about $20 a month).

---

## Architecture Overview

The project has 3 main components:

1. **Crawler**
2. **Processor (Generator)**
3. **Editor + Publishing Flow**

<div style="margin: 30px 0; padding: 20px; background: white; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
<svg viewBox="0 0 900 500" width="100%" height="auto" style="max-width: 900px; display: block; margin: 0 auto;">
<defs>
<linearGradient id="gcpGrad" x1="0%" y1="0%" x2="0%" y2="100%">
<stop offset="0%" style="stop-color:#4285f4;stop-opacity:1" />
<stop offset="100%" style="stop-color:#1a73e8;stop-opacity:1" />
</linearGradient>
<linearGradient id="foxGrad2" x1="0%" y1="0%" x2="0%" y2="100%">
<stop offset="0%" style="stop-color:#ff6b6b;stop-opacity:1" />
<stop offset="100%" style="stop-color:#ee5a24;stop-opacity:1" />
</linearGradient>
<linearGradient id="msnbcGrad2" x1="0%" y1="0%" x2="0%" y2="100%">
<stop offset="0%" style="stop-color:#4dabf7;stop-opacity:1" />
<stop offset="100%" style="stop-color:#339af0;stop-opacity:1" />
</linearGradient>
<filter id="shadow2">
<feDropShadow dx="0" dy="3" stdDeviation="3" flood-opacity="0.15"/>
</filter>
<marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto">
<path d="M 0 0 L 10 5 L 0 10 z" fill="#666" />
</marker>
</defs>

<!-- Title -->
<text x="450" y="30" font-family="Arial, sans-serif" font-size="22" font-weight="bold" fill="#2d3748" text-anchor="middle">Echo & Chamber Architecture</text>

<!-- Phase 1: Crawler -->
<g>
<rect x="50" y="70" width="200" height="380" rx="10" fill="#f8f9fa" stroke="#dee2e6" stroke-width="2" filter="url(#shadow2)" />
<text x="150" y="100" font-family="Arial, sans-serif" font-size="16" font-weight="bold" fill="#2d3748" text-anchor="middle">1. CRAWLER</text>

<!-- News sources -->
<rect x="70" y="120" width="160" height="50" rx="5" fill="url(#foxGrad2)" />
<text x="150" y="150" font-family="Arial, sans-serif" font-size="14" fill="white" font-weight="bold" text-anchor="middle">Fox News</text>

<rect x="70" y="180" width="160" height="50" rx="5" fill="url(#msnbcGrad2)" />
<text x="150" y="210" font-family="Arial, sans-serif" font-size="14" fill="white" font-weight="bold" text-anchor="middle">MSNBC</text>

<!-- Cloud Scheduler -->
<rect x="70" y="250" width="160" height="40" rx="5" fill="url(#gcpGrad)" />
<text x="150" y="275" font-family="Arial, sans-serif" font-size="12" fill="white" text-anchor="middle">Cloud Scheduler</text>
<text x="150" y="305" font-family="Arial, sans-serif" font-size="11" fill="#666" text-anchor="middle">(Hourly Trigger)</text>

<!-- Cloud Functions -->
<rect x="70" y="320" width="160" height="40" rx="5" fill="url(#gcpGrad)" />
<text x="150" y="345" font-family="Arial, sans-serif" font-size="12" fill="white" text-anchor="middle">Cloud Functions</text>
<text x="150" y="375" font-family="Arial, sans-serif" font-size="11" fill="#666" text-anchor="middle">(Python Scraper)</text>

<!-- Cloud Storage -->
<rect x="70" y="390" width="160" height="40" rx="5" fill="url(#gcpGrad)" />
<text x="150" y="415" font-family="Arial, sans-serif" font-size="12" fill="white" text-anchor="middle">Cloud Storage</text>
</g>

<!-- Arrow 1 -->
<line x1="260" y1="250" x2="340" y2="250" stroke="#666" stroke-width="2" marker-end="url(#arrow)" stroke-dasharray="5,5" />

<!-- Phase 2: Processor -->
<g>
<rect x="350" y="70" width="200" height="380" rx="10" fill="#f8f9fa" stroke="#dee2e6" stroke-width="2" filter="url(#shadow2)" />
<text x="450" y="100" font-family="Arial, sans-serif" font-size="16" font-weight="bold" fill="#2d3748" text-anchor="middle">2. PROCESSOR</text>

<!-- Processing steps -->
<rect x="370" y="120" width="160" height="35" rx="5" fill="#f1f3f5" stroke="#adb5bd" />
<text x="450" y="143" font-family="Arial, sans-serif" font-size="12" fill="#495057" text-anchor="middle">Create Embeddings</text>

<rect x="370" y="165" width="160" height="35" rx="5" fill="#f1f3f5" stroke="#adb5bd" />
<text x="450" y="188" font-family="Arial, sans-serif" font-size="12" fill="#495057" text-anchor="middle">Cluster Stories</text>

<rect x="370" y="210" width="160" height="35" rx="5" fill="#f1f3f5" stroke="#adb5bd" />
<text x="450" y="233" font-family="Arial, sans-serif" font-size="12" fill="#495057" text-anchor="middle">Prioritize Matches</text>

<rect x="370" y="255" width="160" height="35" rx="5" fill="#f1f3f5" stroke="#adb5bd" />
<text x="450" y="278" font-family="Arial, sans-serif" font-size="12" fill="#495057" text-anchor="middle">Generate Summaries</text>

<!-- APIs -->
<rect x="370" y="310" width="160" height="40" rx="5" fill="#ffd43b" />
<text x="450" y="335" font-family="Arial, sans-serif" font-size="12" fill="#495057" text-anchor="middle" font-weight="bold">Anthropic API</text>

<rect x="370" y="360" width="160" height="40" rx="5" fill="#94d82d" />
<text x="450" y="385" font-family="Arial, sans-serif" font-size="12" fill="#495057" text-anchor="middle" font-weight="bold">Hugging Face</text>

<!-- Cloud Storage -->
<rect x="370" y="410" width="160" height="30" rx="5" fill="url(#gcpGrad)" />
<text x="450" y="430" font-family="Arial, sans-serif" font-size="12" fill="white" text-anchor="middle">Cloud Storage</text>
</g>

<!-- Arrow 2 -->
<line x1="560" y1="250" x2="640" y2="250" stroke="#666" stroke-width="2" marker-end="url(#arrow)" stroke-dasharray="5,5" />

<!-- Phase 3: Editor + Publishing -->
<g>
<rect x="650" y="70" width="200" height="380" rx="10" fill="#f8f9fa" stroke="#dee2e6" stroke-width="2" filter="url(#shadow2)" />
<text x="750" y="100" font-family="Arial, sans-serif" font-size="16" font-weight="bold" fill="#2d3748" text-anchor="middle">3. PUBLISH</text>

<!-- Editor UI -->
<rect x="670" y="120" width="160" height="80" rx="5" fill="#e3f2fd" stroke="#2196f3" stroke-width="2" />
<text x="750" y="150" font-family="Arial, sans-serif" font-size="13" fill="#1976d2" text-anchor="middle" font-weight="bold">Custom Editor UI</text>
<text x="750" y="170" font-family="Arial, sans-serif" font-size="11" fill="#666" text-anchor="middle">Review & Edit</text>
<text x="750" y="190" font-family="Arial, sans-serif" font-size="11" fill="#666" text-anchor="middle">(~10 min/day)</text>

<!-- Manual step indicator -->
<circle cx="750" cy="240" r="20" fill="#ffeaa7" stroke="#fdcb6e" stroke-width="2" />
<text x="750" y="245" font-family="Arial, sans-serif" font-size="16" fill="#f39c12" text-anchor="middle" font-weight="bold">✋</text>
<text x="750" y="270" font-family="Arial, sans-serif" font-size="11" fill="#666" text-anchor="middle">Manual Review</text>

<!-- Beehiiv -->
<rect x="670" y="300" width="160" height="60" rx="5" fill="#8e44ad" />
<text x="750" y="335" font-family="Arial, sans-serif" font-size="14" fill="white" text-anchor="middle" font-weight="bold">Beehiiv</text>
<text x="750" y="350" font-family="Arial, sans-serif" font-size="11" fill="#e8d5f2" text-anchor="middle">(Newsletter Platform)</text>

<!-- Subscribers -->
<rect x="670" y="380" width="160" height="50" rx="5" fill="#27ae60" />
<text x="750" y="410" font-family="Arial, sans-serif" font-size="14" fill="white" text-anchor="middle" font-weight="bold">📧 Subscribers</text>
</g>

<!-- Bottom labels -->
<text x="150" y="475" font-family="Arial, sans-serif" font-size="12" fill="#868e96" text-anchor="middle">Automated</text>
<text x="450" y="475" font-family="Arial, sans-serif" font-size="12" fill="#868e96" text-anchor="middle">Automated</text>
<text x="750" y="475" font-family="Arial, sans-serif" font-size="12" fill="#868e96" text-anchor="middle">Manual + Automated</text>
</svg>
</div>

---

## Part 1: Crawler

The crawler is relatively straightforward, using python I read the homepage of each Fox News and MSNBC hourly and index the top 20 articles. The idea here is that from a content perspective I am only interested in "top stories", and by creating an hourly index I can track the story prominence and also how long it is at the top of their page. For each of the top 20 I copy the content into my index which can then be analyzed by the processor.

I built this using Cloud Functions in GCP triggered by a scheduler, and everything is saved to Cloud Storage buckets. I was surprised at how cost-efficient these services are when used at small scale, for hobby projects serverless is awesome. I did also try using actual databases but the costs were a lot higher because you need continually running compute, the file storage worked for me and is dirt cheap.

The crawler is its own code repo and overall I haven't had much issue with the data collection. So, at this point I've collected an hourly snapshot of the top stories being posted with a prominence ranking, we are now ready to process the information.

---

## Part 2: Processor

The processor application does the bulk of the lifting on the project. It is for sure the most complex code repo in my project. To summarize what it does:

1. Extracts the article text from the files and creates an AI Embedding using the [Sentence Transformer model from Hugging Face](https://huggingface.co/sentence-transformers).
2. Clusters each embedding to group the stories by topic.
3. Prioritizes the topic matches based on what had the most prominence on the homepage for the longest time.
4. Using the top matches from step 3, passes each story pair to the Anthropic API to generate the event summary and framing comparison (uses Sonnet class models).

### 2.1) Article Embeddings

Most people use AI through LLMs which are considered encoder/decoder models. AI is all based around math which essentially takes your input, turns it to numbers (encoder), processes it, and then outputs the response (decoder). With our goal here being to match stories by topic, we only need the encoder: it turns text into a vector of numbers (an "embedding") that captures meaning. What happens is similar texts end up as nearby vectors, so we can compare them using math.

A simple example of this is a King and Queen. If you have the representation of King and subtract the idea of "man", then add the idea of "woman", in the vector space it winds up roughly putting you at queen. See below for a visual:

<div style="margin: 30px 0; padding: 20px; background: white; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
<svg viewBox="0 0 700 400" width="100%" height="auto" style="max-width: 700px; display: block; margin: 0 auto;">
  <defs>
    <linearGradient id="grad1" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:#667eea;stop-opacity:1" />
      <stop offset="100%" style="stop-color:#764ba2;stop-opacity:1" />
    </linearGradient>
    <linearGradient id="grad2" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:#f093fb;stop-opacity:1" />
      <stop offset="100%" style="stop-color:#f5576c;stop-opacity:1" />
    </linearGradient>
    <filter id="shadow" x="-50%" y="-50%" width="200%" height="200%">
      <feDropShadow dx="0" dy="2" stdDeviation="3" flood-opacity="0.2"/>
    </filter>
    <marker id="arrowhead1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto">
      <path d="M 0 0 L 10 5 L 0 10 z" fill="url(#grad1)" />
    </marker>
    <marker id="arrowhead2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto">
      <path d="M 0 0 L 10 5 L 0 10 z" fill="url(#grad2)" />
    </marker>
    <pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse">
      <path d="M 40 0 L 0 0 0 40" fill="none" stroke="#e8e8e8" stroke-width="1"/>
    </pattern>
    <style>
      .label { font-family: Arial, sans-serif; fill:#2d3748; font-weight:bold; 
               paint-order: stroke; stroke: white; stroke-width:4; } /* text halo */
      .small { font-weight:500; stroke-width:3; }
    </style>
  </defs>

  <!-- Grid -->
  <rect width="700" height="400" fill="url(#grid)" />

  <!-- Axes -->
  <line x1="50" y1="350" x2="650" y2="350" stroke="#333" stroke-width="2" />
  <line x1="50" y1="50"  x2="50"  y2="350" stroke="#333" stroke-width="2" />

  <!-- Axis labels -->
  <text x="340" y="390" font-family="Arial, sans-serif" font-size="14" fill="#666" text-anchor="middle">Gender Dimension</text>
  <text x="20" y="200" font-family="Arial, sans-serif" font-size="14" fill="#666" text-anchor="middle" transform="rotate(-90 20 200)">Royalty Dimension</text>

  <!-- 1) VECTORS FIRST (so they're behind everything) -->
  <g opacity="0.8">
    <line x1="150" y1="280" x2="345" y2="155" stroke="url(#grad1)" stroke-width="3" marker-end="url(#arrowhead1)" />
    <line x1="200" y1="280" x2="395" y2="155" stroke="url(#grad2)" stroke-width="3" marker-end="url(#arrowhead2)" />
  </g>

  <!-- Dotted helper lines -->
  <line x1="150" y1="280" x2="200" y2="280" stroke="#999" stroke-width="1" stroke-dasharray="5,5" opacity="0.5" />
  <line x1="350" y1="150" x2="400" y2="150" stroke="#999" stroke-width="1" stroke-dasharray="5,5" opacity="0.5" />

  <!-- 2) POINTS ABOVE VECTORS -->
  <g filter="url(#shadow)">
    <circle cx="150" cy="280" r="8" fill="white" stroke="#4a5568" stroke-width="2" />
    <circle cx="350" cy="150" r="8" fill="white" stroke="#4a5568" stroke-width="2" />
    <circle cx="200" cy="280" r="8" fill="white" stroke="#4a5568" stroke-width="2" />
    <circle cx="400" cy="150" r="8" fill="white" stroke="#4a5568" stroke-width="2" />
  </g>

  <!-- 3) ALL TEXT LAST (with halo) -->
  <text x="150" y="310" class="label" font-size="16" text-anchor="middle">man</text>
  <text x="350" y="130" class="label" font-size="16" text-anchor="middle">king</text>
  <text x="200" y="310" class="label" font-size="16" text-anchor="middle">woman</text>
  <text x="400" y="130" class="label" font-size="16" text-anchor="middle">queen</text>

  <!-- Vector labels -->
  <text x="240" y="210" class="label small" font-size="13" fill="#667eea" stroke="#fff">king − man</text>
  <text x="315" y="210" class="label small" font-size="13" fill="#f5576c" stroke="#fff">+ woman = queen</text>

  <!-- Title -->
  <text x="350" y="30" class="label" font-size="18" text-anchor="middle">Word Embeddings: Vector Arithmetic</text>
</svg>
</div>

The open source [Sentence Transformer model](https://huggingface.co/sentence-transformers) allows us to do this easily.  The example above scales from single words to full articles to compare similarity.

### 2.2) Clustering

Clustering is a pretty simple machine learning concept. We have some threshold of "similarity score" and group our embeddings from step 1. If the embedding distance is close enough together, we say the stories are about the same topic. This drives the quality of the match and mostly just took some tuning of the score threshold - in machine learning we call this tuning a [hyperparameter](https://en.wikipedia.org/wiki/Hyperparameter_(machine_learning)).  We use this concept to pair the stories.

<div style="margin: 30px 0; padding: 20px; background: white; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
<svg viewBox="0 0 800 450" width="100%" height="auto" style="max-width: 800px; display: block; margin: 0 auto;">
<defs>
<linearGradient id="clusterGrad1" x1="0%" y1="0%" x2="100%" y2="100%">
<stop offset="0%" style="stop-color:#667eea;stop-opacity:0.2" />
<stop offset="100%" style="stop-color:#764ba2;stop-opacity:0.2" />
</linearGradient>
<linearGradient id="clusterGrad2" x1="0%" y1="0%" x2="100%" y2="100%">
<stop offset="0%" style="stop-color:#f093fb;stop-opacity:0.2" />
<stop offset="100%" style="stop-color:#f5576c;stop-opacity:0.2" />
</linearGradient>
<linearGradient id="clusterGrad3" x1="0%" y1="0%" x2="100%" y2="100%">
<stop offset="0%" style="stop-color:#4facfe;stop-opacity:0.2" />
<stop offset="100%" style="stop-color:#00f2fe;stop-opacity:0.2" />
</linearGradient>
<filter id="blur" x="-50%" y="-50%" width="200%" height="200%">
<feGaussianBlur in="SourceGraphic" stdDeviation="2" />
</filter>
</defs>

<!-- Title -->
<text x="400" y="30" font-family="Arial, sans-serif" font-size="20" font-weight="bold" fill="#2d3748" text-anchor="middle">Story Clustering: Finding Similar Topics</text>

<!-- Embedding space background -->
<rect x="50" y="60" width="700" height="320" fill="#fafafa" stroke="#e0e0e0" stroke-width="1" rx="5" />
<text x="400" y="85" font-family="Arial, sans-serif" font-size="14" fill="#868e96" text-anchor="middle">Embedding Space (simplified to 2D)</text>

<!-- Cluster regions -->
<ellipse cx="200" cy="200" rx="120" ry="90" fill="url(#clusterGrad1)" filter="url(#blur)" />
<ellipse cx="400" cy="280" rx="110" ry="80" fill="url(#clusterGrad2)" filter="url(#blur)" />
<ellipse cx="600" cy="180" rx="100" ry="85" fill="url(#clusterGrad3)" filter="url(#blur)" />

<!-- Story points - Cluster 1: Election Coverage -->
<circle cx="180" cy="180" r="8" fill="#ee5a24" stroke="white" stroke-width="2" />
<text x="160" y="175" font-family="Arial, sans-serif" font-size="10" fill="#666">Fox</text>

<circle cx="200" cy="190" r="8" fill="#339af0" stroke="white" stroke-width="2" />
<text x="220" y="195" font-family="Arial, sans-serif" font-size="10" fill="#666">MSNBC</text>

<circle cx="170" cy="220" r="8" fill="#ee5a24" stroke="white" stroke-width="2" />
<circle cx="190" cy="230" r="8" fill="#339af0" stroke="white" stroke-width="2" />

<circle cx="230" cy="200" r="8" fill="#ee5a24" stroke="white" stroke-width="2" />
<circle cx="240" cy="210" r="8" fill="#339af0" stroke="white" stroke-width="2" />

<!-- Connection lines showing matches -->
<line x1="180" y1="180" x2="200" y2="190" stroke="#667eea" stroke-width="1.5" opacity="0.5" stroke-dasharray="2,2" />
<line x1="170" y1="220" x2="190" y2="230" stroke="#667eea" stroke-width="1.5" opacity="0.5" stroke-dasharray="2,2" />
<line x1="230" y1="200" x2="240" y2="210" stroke="#667eea" stroke-width="1.5" opacity="0.5" stroke-dasharray="2,2" />

<!-- Story points - Cluster 2: Economic News -->
<circle cx="380" cy="270" r="8" fill="#ee5a24" stroke="white" stroke-width="2" />
<circle cx="390" cy="280" r="8" fill="#339af0" stroke="white" stroke-width="2" />

<circle cx="420" cy="290" r="8" fill="#ee5a24" stroke="white" stroke-width="2" />
<circle cx="430" cy="300" r="8" fill="#339af0" stroke="white" stroke-width="2" />

<circle cx="370" cy="310" r="8" fill="#ee5a24" stroke="white" stroke-width="2" />
<circle cx="380" cy="320" r="8" fill="#339af0" stroke="white" stroke-width="2" />

<!-- Connection lines -->
<line x1="380" y1="270" x2="390" y2="280" stroke="#f5576c" stroke-width="1.5" opacity="0.5" stroke-dasharray="2,2" />
<line x1="420" y1="290" x2="430" y2="300" stroke="#f5576c" stroke-width="1.5" opacity="0.5" stroke-dasharray="2,2" />
<line x1="370" y1="310" x2="380" y2="320" stroke="#f5576c" stroke-width="1.5" opacity="0.5" stroke-dasharray="2,2" />

<!-- Story points - Cluster 3: International News -->
<circle cx="580" cy="160" r="8" fill="#ee5a24" stroke="white" stroke-width="2" />
<circle cx="590" cy="170" r="8" fill="#339af0" stroke="white" stroke-width="2" />

<circle cx="620" cy="180" r="8" fill="#ee5a24" stroke="white" stroke-width="2" />
<circle cx="630" cy="190" r="8" fill="#339af0" stroke="white" stroke-width="2" />

<circle cx="570" cy="200" r="8" fill="#ee5a24" stroke="white" stroke-width="2" />
<circle cx="580" cy="210" r="8" fill="#339af0" stroke="white" stroke-width="2" />

<!-- Connection lines -->
<line x1="580" y1="160" x2="590" y2="170" stroke="#00f2fe" stroke-width="1.5" opacity="0.5" stroke-dasharray="2,2" />
<line x1="620" y1="180" x2="630" y2="190" stroke="#00f2fe" stroke-width="1.5" opacity="0.5" stroke-dasharray="2,2" />
<line x1="570" y1="200" x2="580" y2="210" stroke="#00f2fe" stroke-width="1.5" opacity="0.5" stroke-dasharray="2,2" />

<!-- Cluster labels -->
<text x="200" y="150" font-family="Arial, sans-serif" font-size="14" font-weight="bold" fill="#667eea" text-anchor="middle">Election Coverage</text>
<text x="400" y="245" font-family="Arial, sans-serif" font-size="14" font-weight="bold" fill="#f5576c" text-anchor="middle">Economic News</text>
<text x="600" y="140" font-family="Arial, sans-serif" font-size="14" font-weight="bold" fill="#00b4d8" text-anchor="middle">International</text>

<!-- Distance indicator -->
<g transform="translate(100, 320)">
<line x1="0" y1="0" x2="40" y2="0" stroke="#666" stroke-width="1.5" />
<circle cx="0" cy="0" r="3" fill="#666" />
<circle cx="40" cy="0" r="3" fill="#666" />
<text x="20" y="-5" font-family="Arial, sans-serif" font-size="10" fill="#666" text-anchor="middle">distance</text>
<text x="20" y="15" font-family="Arial, sans-serif" font-size="11" fill="#666" text-anchor="middle">threshold</text>
</g>

<!-- Legend -->
<g transform="translate(60, 400)">
<rect x="0" y="0" width="15" height="15" rx="2" fill="#ee5a24" />
<text x="20" y="12" font-family="Arial, sans-serif" font-size="12" fill="#666">Fox News Article</text>

<rect x="150" y="0" width="15" height="15" rx="2" fill="#339af0" />
<text x="170" y="12" font-family="Arial, sans-serif" font-size="12" fill="#666">MSNBC Article</text>

<line x1="320" y1="7" x2="340" y2="7" stroke="#999" stroke-width="1.5" stroke-dasharray="2,2" />
<text x="345" y="12" font-family="Arial, sans-serif" font-size="12" fill="#666">Matched Pair</text>
</g>

<!-- Explanation (moved down to avoid overlap) -->
<text x="400" y="430" font-family="Arial, sans-serif" font-size="12" fill="#868e96" text-anchor="middle">Stories within the distance threshold are clustered as the same topic</text>
</svg>
</div>

### 2.3) Story Prioritization

So now we have story pairs about the same topic and we want to rank them by "top story".  In lieu of trying to explain a complex algorithm in paragraph form I fed the code to Opus 4.1 and asked for a visual to explain the algorithm and it turned out good enough:

<div style="margin: 30px 0; padding: 20px; background: white; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1);">
<svg viewBox="0 0 800 500" width="100%" height="auto" style="max-width: 800px; display: block; margin: 0 auto;">
<defs>
<linearGradient id="foxGrad" x1="0%" y1="0%" x2="0%" y2="100%">
<stop offset="0%" style="stop-color:#ff6b6b;stop-opacity:1" />
<stop offset="100%" style="stop-color:#ee5a24;stop-opacity:1" />
</linearGradient>
<linearGradient id="msnbcGrad" x1="0%" y1="0%" x2="0%" y2="100%">
<stop offset="0%" style="stop-color:#4dabf7;stop-opacity:1" />
<stop offset="100%" style="stop-color:#339af0;stop-opacity:1" />
</linearGradient>
<filter id="dropShadow">
<feDropShadow dx="0" dy="2" stdDeviation="2" flood-opacity="0.15"/>
</filter>
</defs>

<!-- Title -->
<text x="400" y="30" font-family="Arial, sans-serif" font-size="20" font-weight="bold" fill="#2d3748" text-anchor="middle">Story Prioritization Algorithm</text>

<!-- Time axis -->
<line x1="50" y1="450" x2="750" y2="450" stroke="#666" stroke-width="2" />
<text x="400" y="485" font-family="Arial, sans-serif" font-size="14" fill="#666" text-anchor="middle">Time (Hours)</text>

<!-- Hour markers -->
<text x="100" y="470" font-family="Arial, sans-serif" font-size="12" fill="#999" text-anchor="middle">8am</text>
<text x="250" y="470" font-family="Arial, sans-serif" font-size="12" fill="#999" text-anchor="middle">12pm</text>
<text x="400" y="470" font-family="Arial, sans-serif" font-size="12" fill="#999" text-anchor="middle">4pm</text>
<text x="550" y="470" font-family="Arial, sans-serif" font-size="12" fill="#999" text-anchor="middle">8pm</text>
<text x="700" y="470" font-family="Arial, sans-serif" font-size="12" fill="#999" text-anchor="middle">12am</text>

<!-- Position axis -->
<line x1="50" y1="60" x2="50" y2="450" stroke="#666" stroke-width="2" />
<text x="20" y="250" font-family="Arial, sans-serif" font-size="14" fill="#666" text-anchor="middle" transform="rotate(-90 20 250)">Homepage Position</text>

<!-- Position markers -->
<text x="35" y="100" font-family="Arial, sans-serif" font-size="12" fill="#999" text-anchor="end">#1</text>
<text x="35" y="200" font-family="Arial, sans-serif" font-size="12" fill="#999" text-anchor="end">#5</text>
<text x="35" y="300" font-family="Arial, sans-serif" font-size="12" fill="#999" text-anchor="end">#10</text>
<text x="35" y="400" font-family="Arial, sans-serif" font-size="12" fill="#999" text-anchor="end">#15</text>

<!-- Fox News stories -->
<g filter="url(#dropShadow)">
<rect x="100" y="90" width="250" height="20" rx="3" fill="url(#foxGrad)" opacity="0.9" />
<text x="110" y="104" font-family="Arial, sans-serif" font-size="12" fill="white" font-weight="500">Fox: Breaking News Story</text>

<rect x="200" y="180" width="180" height="15" rx="3" fill="url(#foxGrad)" opacity="0.7" />
<text x="210" y="191" font-family="Arial, sans-serif" font-size="11" fill="white">Fox: Political Update</text>

<rect x="350" y="120" width="220" height="18" rx="3" fill="url(#foxGrad)" opacity="0.8" />
<text x="360" y="133" font-family="Arial, sans-serif" font-size="11" fill="white">Fox: Economic Report</text>
</g>

<!-- MSNBC stories -->
<g filter="url(#dropShadow)">
<rect x="120" y="110" width="280" height="20" rx="3" fill="url(#msnbcGrad)" opacity="0.9" />
<text x="130" y="124" font-family="Arial, sans-serif" font-size="12" fill="white" font-weight="500">MSNBC: Breaking News Story</text>

<rect x="250" y="200" width="200" height="15" rx="3" fill="url(#msnbcGrad)" opacity="0.7" />
<text x="260" y="211" font-family="Arial, sans-serif" font-size="11" fill="white">MSNBC: Political Update</text>

<rect x="380" y="140" width="240" height="18" rx="3" fill="url(#msnbcGrad)" opacity="0.8" />
<text x="390" y="153" font-family="Arial, sans-serif" font-size="11" fill="white">MSNBC: Economic Report</text>
</g>

<!-- Priority scores -->
<g>
<circle cx="650" cy="100" r="35" fill="#ffd43b" stroke="#fab005" stroke-width="2" filter="url(#dropShadow)" />
<text x="650" y="95" font-family="Arial, sans-serif" font-size="12" fill="#495057" text-anchor="middle">Priority</text>
<text x="650" y="110" font-family="Arial, sans-serif" font-size="16" font-weight="bold" fill="#495057" text-anchor="middle">HIGH</text>

<circle cx="650" cy="190" r="30" fill="#94d82d" stroke="#74b816" stroke-width="2" filter="url(#dropShadow)" />
<text x="650" y="185" font-family="Arial, sans-serif" font-size="11" fill="#495057" text-anchor="middle">Priority</text>
<text x="650" y="200" font-family="Arial, sans-serif" font-size="14" font-weight="bold" fill="#495057" text-anchor="middle">MED</text>

<circle cx="650" cy="270" r="25" fill="#adb5bd" stroke="#868e96" stroke-width="2" filter="url(#dropShadow)" />
<text x="650" y="265" font-family="Arial, sans-serif" font-size="10" fill="#495057" text-anchor="middle">Priority</text>
<text x="650" y="278" font-family="Arial, sans-serif" font-size="12" font-weight="bold" fill="#495057" text-anchor="middle">LOW</text>
</g>

<!-- Legend -->
<g>
<rect x="70" y="340" width="15" height="15" fill="url(#foxGrad)" />
<text x="90" y="352" font-family="Arial, sans-serif" font-size="13" fill="#666">Fox News</text>

<rect x="170" y="340" width="15" height="15" fill="url(#msnbcGrad)" />
<text x="190" y="352" font-family="Arial, sans-serif" font-size="13" fill="#666">MSNBC</text>
</g>

<!-- Algorithm explanation -->
<text x="400" y="380" font-family="Arial, sans-serif" font-size="14" fill="#495057" text-anchor="middle" font-weight="500">Score = Position Weight × Duration × Coverage Overlap</text>
<text x="400" y="400" font-family="Arial, sans-serif" font-size="12" fill="#868e96" text-anchor="middle">Higher position + Longer duration + Both outlets = Higher priority</text>
</svg>
</div>

### 2.4) Story Generation

At this point we have a handful of story pairs with the story headline, text and URL for both articles. I pass this to the Anthropic API and use Sonnet to generate the summary. I have this flip back and forth so Fox/MSNBC change which is first so there are two prompts. I change this from time to time but at the time of this blog here are the prompts:

<div style="background: #f8f9fa; border: 1px solid #dee2e6; border-radius: 8px; padding: 20px; margin: 20px 0;">
<style>
.prompt-tabs { display: flex; gap: 10px; margin-bottom: 15px; border-bottom: 2px solid #dee2e6; }
.prompt-tab { padding: 10px 20px; background: none; border: none; cursor: pointer; color: #6c757d; font-size: 14px; font-weight: 500; transition: all 0.3s; border-bottom: 3px solid transparent; margin-bottom: -2px; }
.prompt-tab.active { color: #007bff; border-bottom-color: #007bff; }
.prompt-tab:hover { color: #495057; }
.prompt-content { display: none; max-height: 400px; overflow-y: auto; font-size: 13px; line-height: 1.5; font-family: 'Monaco', 'Consolas', monospace; white-space: pre-wrap; background: white; padding: 15px; border-radius: 4px; color: #495057; }
.prompt-content.active { display: block; }
</style>

<div class="prompt-tabs">
<button class="prompt-tab active" onclick="showPrompt('fox')">Fox News First</button>
<button class="prompt-tab" onclick="showPrompt('msnbc')">MSNBC First</button>
</div>

<div id="fox-prompt" class="prompt-content active">Analyze how Fox News and MSNBC covered the same story differently. Write engaging newsletter content that explains the story and contrasts their approaches.

<strong>FOX NEWS COVERAGE:</strong>
Headline: "{fox_headline}"
Article content: {fox_content}

<strong>MSNBC COVERAGE:</strong>
Headline: "{msnbc_headline}" 
Article content: {msnbc_content}

Write 2 paragraphs (250-280 words total) that:

<strong>First paragraph:</strong> Start by describing what actually happened. Set the scene with the key facts, events, people involved, and timeline. Draw from both articles to paint the complete picture. Don't mention "according to headlines" or "the outlets reported" - just tell the story naturally.

<strong>Second paragraph:</strong> Concisely explain how each outlet approached the story differently. Cover Fox News's emphasis, angle, and key framing first, then MSNBC's approach, highlighting the most notable contrasts between them. Focus on 2-3 concrete differences using specific examples.

<strong>Writing style requirements:</strong>
- Write like a daily newsletter that readers subscribe to and enjoy
- Use engaging, conversational tone
- Start directly with the facts of what happened - no headers, titles, or introductory phrases
- Don't include meta-commentary about "different lenses," "how events can be seen differently," or similar editorial observations
- Show concrete differences using specific examples from the articles
- Avoid academic language or LLM phrases like "the outlets diverged" or "according to the provided"
- Avoid editorial commentary like "the contrast is striking" or "the difference is stark" - let readers draw their own conclusions
- Keep the story summary objective and factual
- When describing coverage differences, present them neutrally without loaded terms or value judgments
- Write for intelligent readers who want to understand both the story and how it was presented differently
- Be selective - highlight the most important differences rather than cataloging everything

Make this content that people would want to read in their morning newsletter, not an AI's analysis of headlines.</div>

<div id="msnbc-prompt" class="prompt-content">Analyze how MSNBC and Fox News covered the same story differently. Write engaging newsletter content that explains the story and contrasts their approaches.

<strong>MSNBC COVERAGE:</strong>
Headline: "{msnbc_headline}" 
Article content: {msnbc_content}

<strong>FOX NEWS COVERAGE:</strong>
Headline: "{fox_headline}"
Article content: {fox_content}

Write 2 paragraphs (250-280 words total) that:

<strong>First paragraph:</strong> Start by describing what actually happened. Set the scene with the key facts, events, people involved, and timeline. Draw from both articles to paint the complete picture. Don't mention "according to headlines" or "the outlets reported" - just tell the story naturally.

<strong>Second paragraph:</strong> Concisely explain how each outlet approached the story differently. Cover MSNBC's emphasis, angle, and key framing first, then Fox News's approach, highlighting the most notable contrasts between them. Focus on 2-3 concrete differences using specific examples.

<strong>Writing style requirements:</strong>
- Write like a daily newsletter that readers subscribe to and enjoy
- Use engaging, conversational tone
- Start directly with the facts of what happened - no headers, titles, or introductory phrases
- Don't include meta-commentary about "different lenses," "how events can be seen differently," or similar editorial observations
- Show concrete differences using specific examples from the articles
- Avoid academic language or LLM phrases like "the outlets diverged" or "according to the provided"
- Avoid editorial commentary like "the contrast is striking" or "the difference is stark" - let readers draw their own conclusions
- Keep the story summary objective and factual
- When describing coverage differences, present them neutrally without loaded terms or value judgments
- Write for intelligent readers who want to understand both the story and how it was presented differently
- Be selective - highlight the most important differences rather than cataloging everything

Make this content that people would want to read in their morning newsletter, not an AI's analysis of headlines.</div>

<script>
function showPrompt(type) {
    document.querySelectorAll('.prompt-content').forEach(p => p.classList.remove('active'));
    document.querySelectorAll('.prompt-tab').forEach(t => t.classList.remove('active'));
    
    if (type === 'fox') {
        document.getElementById('fox-prompt').classList.add('active');
        document.querySelectorAll('.prompt-tab')[0].classList.add('active');
    } else {
        document.getElementById('msnbc-prompt').classList.add('active');
        document.querySelectorAll('.prompt-tab')[1].classList.add('active');
    }
}
</script>
</div>

---

## Part 3: Editing + Publishing

Alright so now we have a newsletter.  The generator usually is pretty good but needs manual review.  The output is raw HTML so I needed a way to quickly make edits since I plan to send this most weekdays.  A bit of upfront work saves time every day on this, so I built a UI that allows me to quickly make edits to the letter, which you can see below:

<p align="center">
  <img src="/assets/images/firstpost/echo-editor.png" alt="Editor UI" width="600"><br>
  <span style="font-size: 0.9em; color: #666;">
    Editor UI I built to easily modify the HTML
  </span>
</p>


It's pretty nice and overall I enjoy reading the daily summary so it doesn't feel like a chore and is just a part of my night routine.  Typically it will match 1-2 stories that really don't have to do with eachother so I'll remove those and then send it out.

My publishing platform is Beehiiv which overall has been great except they don't offer an API to publish the letter without an enterprise agreement.  So the last step is I hop through their web UI and copy paste my letter to schedule for the following morning.  Overall it takes about 10 minutes to read the letter and 5 minutes to schedule the send.

---

## Conclusion

Overall this has been a fun project and I learned a lot, hopefully you enjoyed reading about it.  The next learning for me here is how to market the idea without spending money.  For my next post I plan to talk about some of what I learned using AI coding tools.  Could certainly have gone way deeper here but this is long enough as it is, thanks for reading!!

[Echo & Chamber](https://echoandchamber.com)

- Mike