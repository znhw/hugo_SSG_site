+++
title = 'LLM-as-Judge For Contextual Evaluation'
date = '2026-09-30T13:30:19+08:00'
description = ""
tags = ['Software Engineering', 'LLM']
draft = false 
unlisted = false
cardColor = 'dark-orange'
+++

In my previous post, I documented moving from fuzzy keyword search (Fuse.js) to semantic vector search (all-MiniLM-L6-v2 + LanceDB). The upgrade felt huge: vector search looks past literal character matches to find conceptual intent.

However, as I ran deeper tests with emotional user inputs, I ran into a boundary that vector search cannot cross on its own: **vector search evaluates distance, not appropriateness**.

## The Boundary: Vector Search is a Mirror

At their core, transformer-based Large Language Models aren't just text generators. Their fundamental power lies in **context processing and pattern recognition:** analyzing relationships, organizing unstructured information, and evaluating constraints across high-dimensional token representations.
When developers build with LLMs, the default pattern is almost always text generation. But when building a retrieval-backed app (like an anime quote finder), generation introduces severe tradeoffs:

- **Hallucination**: An LLM prompted to "write a comforting anime quote" might invent a line that no character ever actually said.

- **Loss of Authenticity**: Fans expect canonical quotes from real series, not synthetic approximations.

- **Cost & Latency**: Generating 50–100 tokens of creative text takes far more time and compute than classifying existing strings.

On the other hand, relying strictly on **Vector Retrieval** avoids hallucination, but acts like a passive mirror. When a user inputs something dejecting, vector distance math dutifully pairs that query with the closest vectors in high-dimensional space, often creating an emotional echo chamber.

To solve this, we don't use the LLM to generate new quotes. We use its analytical capability as an **“Judge”** to evaluate a candidate set retrieved by our vector database.


## Testing in the Wild: A Case Study

To test this architecture, I queried my vector database of 7,372 anime quotes with a simple, emotionally charged phrase:

Query: "I feel hopeless"

Pure vector retrieval (LanceDB using all-MiniLM-L6-v2) pulled the top 10 nearest candidates by cosine distance:

![Semantic vector retrieval](/images/semantic-vector-quotes-retrived.png)


1. "Cheer up. No matter how hopeless you are, even if everyone else abandons you, I’ll always be here for you."
   — Kousaka Kirino (Oreimo) | Distance: 0.840

2. "We're all lost... That's why we're sad."
   — Van Hohenheim (Fullmetal Alchemist) | Distance: 0.918

3. "Even in moments of the deepest despair... I guess we can still find hope, huh?"
   — Hange Zoe (Shingeki no Kyojin) | Distance: 0.937

4. "There’s no despair that can’t be overcome by everyday life."
   — Otonashi Maria (Utsuro no Hako to Zero no Maria) | Distance: 0.956

5. "Your so called "hope" is to throw the past into despair?"
   — Natsu Dragneel (Fairy Tail) | Distance: 0.965

6. "Loneliness is a sickness that leads to death."
   — Horo (Spice and Wolf) | Distance: 0.984

7. "Those who know despair, once knew hope. Those who know loss, once knew love."
   — Ulquiorra Schiffer (Bleach) | Distance: 0.984

8. "I shall grieve, and I shall weep. But I shall never regret."
   — Rider (Fate/zero) | Distance: 1.002

9. "I've continually fought, and with each battle I've been killing my own heart..."
   — Trowa Barton (Gundam Wing) | Distance: 1.010

10. "Wherever there is hope, there is most definitely despair."
    — Junko Enoshima (Danganronpa) | Distance: 1.012

Why Pure Vector Search Selected Candidate #1
At 0.840, Kirino’s quote is the mathematical winner. Why? Because it literally contains the word *"hopeless"* and uses explicit reassuring language ("Cheer up", "I'll always be here").

However, look closely at the rest of the list: Candidates #2, #6, #9, and #10 are deeply bleak. Candidate #10 (Junko Enoshima) actively equates hope with despair. If we simply returned the top $K$ items without evaluation, we risk handing a distressed user quotes that validate isolation or despair.

## Enter the LLM Judge
Instead of serving candidate #1 automatically, I passed all 10 candidate vectors to Gemini as an LLM Judge.

The judge was instructed to evaluate the user's emotional state and select the quote that offers the most contextually appropriate and constructive perspective.

Here is the raw output returned by the LLM Judge:

![LLM-as-Judge quote selected](/images/llm-as-judge-selection-quote.png)

`{
  "selectedQuote": {
    "id": "6697e446e3adf3ea5a01e57a",
    "quote": "There’s no despair that can’t be overcome by everyday life.",
    "character": "Otonashi Maria",
    "show": "Utsuro no Hako to Zero no Maria",
    "distance": 0.970926821231842
  },
  "reason": "This quote directly addresses the feeling of hopelessness by suggesting that even in despair, everyday life can offer a path forward, making it semantically relevant and contextually appropriate as a comforting response."
}`

Why the Judge Skipped Candidate #1 for Candidate #4
The LLM Judge bypassed Candidate #1 (distance 0.840) and selected Candidate #4 (distance 0.956 / 0.970).
* Candidate #1 (Kirino): While reassuring, it introduces a dramatic, codependent framing ("even if everyone else abandons you").

* Candidate #4 (Maria): It directly addresses despair with a grounded, stoic, and universally comforting philosophy—that routine, daily life provides a natural path forward out of hopelessness.
The LLM didn't invent text or hallucinate anime lore. It performed contextual evaluation over pre-retrieved candidates, picking an answer that vector distance alone was too blunt to identify.

### Comparing the Approaches
| **System Layer** | **Primary Mechanism** | **Best Use Case** | **Primary Limitation** |
| --- | --- | --- | --- |
| **Keyword Search (Fuse.js)** | Lexical character matching | Exact names, titles, typo tolerance | Fails on abstract ideas |
| **Semantic Search (MiniLM)** | High-dimensional vector distance | Recall (narrowing 7k+ items to 10 in <10ms) | Blind to emotional nuance & appropriateness |
| **LLM-as-a-Generator** | Unconstrained token prediction | Creative writing, open chit-chat | High hallucination risk, non-canonical output |
| **Vector Search + LLM Judge** | Candidate retrieval + context analysis | High-fidelity database retrieval with reasoning | Adds minor API evaluation latency (~150ms) |

## Conclusion

Ultimately, viewing LLMs merely as text generators drastically understates their utility. The most durable business application of language models isn't having them draft prose, it is using them as context-aware reasoning engines. Whether acting as automated judges, structured parsers, or dynamic workflow routers, LLMs deliver their highest ROI when operating as the analytical brain behind structured, deterministic software systems.
