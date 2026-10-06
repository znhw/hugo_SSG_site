+++
title = 'Building A Retrieval System: Keywords vs. Natural Language'
date = '2026-09-28T16:49:29+08:00'
description = "Comparing fuzzy search and semantic retrieval using Fuse.js, MiniLM, and LanceDB."
tags = ["Software Engineering", "Search"]
draft = false 
unlisted = false
cardColor = 'orange'
+++

Imagine a foreign tourist who speaks limited English coming up, making a few hand gestures, and saying: “Train. Airport.” 

You understand both words, but do you know what he wants to convey? He could be asking where to catch the train to the airport, when the next train leaves, or simply whether a train to the airport exists. 

This short interaction highlights the fundamental challenge of building a text search engine: keywords carry specific terms, but they omit intent and context. 

I ran into this exact barrier while building a simple anime quote search app.The idea was straightforward: the user types something, the backend searches through a collection of quotes and returns the most relevant one. 

In this article, I’ll demonstrate my initial approach using Fuse.js and what changed when I transitioned to vector embeddings for semantic search. 

To compare both approaches, I used the same dataset of 7,372 anime quotes and ran the same queries against each retrieval system. 

## Searching with Fuse.js 

` 
const fuse = new Fuse(quotes, {
    keys: ["quote"],
    includeScore: true,
    threshold: 0.4
});
const results = fuse.search(query);
`

![Fuse.js keyword search result](/images/fusejs-semantic-search-result.png)

The result shows: 
> 
Query: "What is my purpose in life?" <br><br>
Total quotes: 7372
Results found: 0

The first query I used was **“What is my purpose in life?”** However, I did not get any results because none of the quotes were textually similar enough to the complete query.

So I reduced the query to a single keyword: **"purpose"**

This time, Fuse.js returned several results:

![Fuse.js search result](/images/fusejs-keyword-search-result.png)

>  
1. The purpose of practice is to improve your power. The purpose of the real race is to win.
2. Is there any purpose to wings that can't fly?
3. The purpose of an alliance is not simply for the sake of preventing an enemy from attacking you. Rather, importance lies in what one can obtain after the alliance. As well as your actions after the alliance.
4. I’ll use my flames for a better purpose!
5. It's not that I missed. I missed on purpose.
6. Keep the past, for all intents and purposes, where it is.
7. You only need a strong will and a clear purpose.
8. You're not supposed to find yourself, you're supposed to choose yourself!

Notice that not every result contains the exact word *“purpose.”*

Fuse.js performs fuzzy matching, so it can also return text that is approximately similar at the character level. This is why *“purposes”* can match *“purpose,”* and why a sentence containing *“supposed”* can appear despite not discussing purpose at all.

This makes fuzzy search more forgiving than exact keyword matching. It can handle variations and imperfect matches, but similarity between characters or words does not necessarily imply similarity in meaning.

## Searching by meaning instead

To search by meaning rather than textual similarity, I replaced Fuse.js with a semantic search pipeline using **all-MiniLM-L6-v2 and LanceDB**.

Instead of comparing the query directly against the words in each quote, MiniLM converts both the query and the quotes into numerical representations called vector embeddings.

Texts with similar meanings should produce vectors that are closer together in the embedding space. This allows us to search for quotes that are conceptually related to the query, even when they don't use the same words.

The retrieval flow becomes:
> User query → MiniLM → vector embedding → LanceDB → nearest quote vectors

Running the same query again: **“What is my purpose in life?”**

![MiniLM semantic search result](/images/miniLM-semantic-search.png)

> 
1. Maybe, just maybe, there is no purpose in life... but if you linger a while longer in this world, you might discover something of value in it.
2. I've been wondering... There must be a purpose for people being born into this world. Why are we here? What does it mean? I've been thinking about it a lot lately. I realized that finding our purpose IS the meaning. That's why we're here. And the ones who find it... They're the only ones who are truly free.
3. I've been thinking... for a long while. For what purpose was I born into this world? Whenever I resolved one question, another would arise in its place. I sought the beginning. I sought the end. I've just been walking and walking, thinking all the while. Perhaps nothing will change, no matter how far I go. If I'm to stop my journey, that is fine, too. Even if I were informed that everything has reached its end, I'd just accept it. But even so, I found an answer to yet another of my questions today.
4. Humans... Do humans have a purpose when they are born? I have been wondering recently. Because they are born, do they have an important duty? The meaning of being born... For humans to find that answer... It is the one freedom God gave them.
5. Everyone’s gonna die. It’s a natural part of life. But if life has no purpose, you’re dead already.
6. It is only through the eyes of others that our lives have any meaning.
7. I... don't know why I'm alive. I have no reason to live. I have no interest in others. Living without concern for others... That's the way to live. Working a job that earns me just enough to eat... A life like that was enough...

The difference becomes more interesting when a relevant quote doesn't use the same words as the query at all.

For example, the quote from number 6: “It is only through the eyes of others that our lives have any meaning.” 

It can still be semantically related to “What is my purpose in life?”, even though *purpose* does not appear in the quote. 

This is the key difference between the two approaches: fuzzy search can find text that **looks similar**, while semantic search can find text that **means something similar**.


## Conclusion: Picking the Right Tool for the Job

Neither approach is inherently "better," they simply solve fundamentally different problems.

* **Fuzzy Keyword Search (Fuse.js)** excels at speed, exact string matching, and handling typos. If a user searches for *"Naruto"* or a specific character name, keyword search will reliably pull up exact matches without needing an AI model or a vector database.

* **Semantic Search (MiniLM + LanceDB)** excels at capturing intent, tone, and abstract ideas. It bridges the gap between what the user asks and what the content actually *means*.

For structured fields, names, or strict lexical queries, keyword search remains indispensable. But when you are trying to understand what a user actually *intends*, like translating *"What is my purpose in life?"* into existential quotes, vector embeddings are hard to beat.

Remember our foreign tourist asking for the `"Train. Airport"`?

Fuse.js is like listening strictly to those two words and pointing them toward a dictionary entry. Semantic search, on the other hand, listens to the context, reads between the lines, and gives them directions to the platform.

When building a search engine, your choice comes down to what your users are bringing to the input box: **exact words, or general ideas**. 
