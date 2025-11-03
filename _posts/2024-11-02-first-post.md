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

