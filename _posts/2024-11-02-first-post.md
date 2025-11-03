---
layout: post
title: "Hello World: Starting Fresh"
date: 2024-11-02
published: true
---

# Echo & Chamber Project Notes

## Project Overview

- Project: **Echo & Chamber**
- Aggregates headlines from **Fox News** and **MSNBC** to compare how each covers top stories.
- Inspired by similar comparison newsletters but designed to be shorter and more digestible (Morning Brew style).
- Goal: Provide a daily snapshot showing how partisan media frame the same news differently.
- Reader intent: Stay informed and entertained by seeing ideological spin from both sides.
- Personal context: User reads WSJ but wanted a lightweight way to glance at the extremes.

---

## Connection to Georgia Tech Project

- Originally inspired by a Georgia Tech research project on **misinformation spread**.
- That project used **AI embeddings** and classic ML to model how topics evolved online.
- Echo & Chamber shares the *mission*, but is separate code-wise—more of a solo spinoff.

---

## Personal Learning Goals

- Learn **AI coding**, especially embedding-based similarity.
- Gain hands-on experience with **Google Cloud Platform (GCP)**.
- Keep it low-lift and **low-maintenance**:
  - Max **15 minutes/day**
  - Minimal costs (GCP cost-conscious, no DB usage)
- Treat it like a creative outlet, not a chore.

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

