# Genshin Llama

An experimental RAG (Retrieval Augmented Generation) application that assists Llama in providing accurate and up-to-date information about the video game Genshin Impact based on Genshin Impact guides and wiki web pages.

This is an extension to my Local Llama RAG project from this [repo](https://github.com/jian-li1/local-llama-rag). More information about the code and implementation can be found there.

The LLM and emdedding model used are Llama 3.1 8B and ``mxbai-embed-large``, respectively.

## Motivation

It's no big surprise that Large Language Models such as ChatGPT and Llama perform poorly at answering questions regarding a video game, and thus, it is never reliable to use LLMs as a "guide" for video games. Although it can pick up general information to a certain extent, it usually fails to provide a correct response for specific topics, and generates irrelevant or nonsensical responses for newest game information that has not been trained on the model yet, which is a consequence of LLM hallucination. 

As a vetern Genshin Impact player, I strive to build an application that bridges the gap between these limitations and improve the LLM, namely Llama, to become well-versed in all areas of Genshin Impact using RAG. Although there are already projects that work on RAG applications for Genshin Impact, they seem to focus on specific domains of the game. My ultimate goal is to design a comprehensive solution such that Llama can answer everything about Genshin Impact, from lore and character builds to exploration and news/announcements.

## Data

The websites that were crawled for Genshin data include:
- [Genshin Impact Wiki](https://genshin-impact.fandom.com/wiki/Genshin_Impact_Wiki): The comprehensive Wikipedia for Genshin Impact
- [Genshin Impact Game8](https://game8.co/games/Genshin-Impact): This website provides exploration and character build guides, and news/announcements.
- [Traveler.gg](https://traveler.gg/): This website provides information about news and character builds.
- [KQM](https://keqingmains.com/): This website provides character build guides.
- [Genshin.gg](https://genshin.gg/): This website also provides information about character builds.

The Chroma database used to store vector embeddings for these data is divided into two collections, namely ``wiki`` for the [Genshin Wiki](https://genshin-impact.fandom.com/wiki/Genshin_Impact_Wiki) web pages, and ``guide`` for the other web pages.

## Document Retrieval

Larger corpora introduces more complexity to the application especially with document retrieval. One issue that emerged in this project was the documents (web pages) from the wiki often overshadow those from the other websites. This means that the top relevant documents returned during the document retrieval stage comprises mostly of wiki web pages. The wiki has a vast number of pages, each dedicated to specific information about Genshin Impact, which leads to it dominating the results most of the time. This might not be effective for generating comprehensive answers for certain prompts, as the information from other websites might provide additional valuable context.

Furthermore, only a certain limit of accepted documents is allowed during the document grading stage because the LLM cannot remember too much context during inference. When this limit is exceeded, the LLM will not consider any more documents. But since the top-k documents are mostly wiki web pages, the LLM usually doesn't have the chance to grade documents from other web pages before reaching the maximum amount of accepted documents.

For example, when asking about the constellations of a particular character, most of the top-k documents come from the wiki, but they contain scattered information of individual constellation levels. Due to the performance of the embedding model, the list of constellations of a character in the wiki may not appear in the results. This would result in incomplete answers, where the LLM may miss information about the other character's constellations. On the other hand, other web pages from Game8 or KQM that contain the list of constellations of a character would rank lower, but they would actaully appear in the top results if the wiki pages were not present. However, their low ranking would not be considered for grading.

The idea is to allow web pages from both the wiki and other websites to be equally considered so the sources can be diversified during the generation stage. While cleaning the data and opting in for a better embedding model can help mitigate this problem, it can also be addressed during the retrieval stage. By merging the results of the top-k wiki documents and the top-k documents from the other websites using Python's ``zip()`` function, the LLM can consider the documents from both the wiki and the other websites at the same time. This can allow the documents from other websites to fill in the gap if the embedding model fails to query specific information from the wiki documents.

## Usage
First, install the required packages:
```
pip install requirements.txt
```

To start the crawler, run
```
python crawler.py [-h] --url URL --base BASE --out OUT [--depth DEPTH] [--exclude [EXCLUDE ...]] [--no-update]
```
or for the multithreaded crawler, run
```
crawler_multi.py [-h] --url URL [URL ...] --base BASE --out OUT [--depth DEPTH] [--exclude [EXCLUDE ...]] [--no-update]
```
See ``python crawler.py -h`` or ``python crawler_multi.py -h`` for more information about the argument options.

Next, to build the database, run
```
python build_database.py [-h] -d DIR [DIR ...] -o OUT -n NAME [-u]
```
Make sure to set the ``-n`` argument to ``wiki`` and ``guide`` for the wiki web pages and the other web pages, respectively. See ``python build_database.py -h`` for more information about the argument options.

Finally, the RAG application can be run under ``genshin_rag.ipynb``.

## Results

Some of the generated responses from specific prompts are quite impressive, like this one here:

**Q: What are all the constellations for Mualani? List them in bullet points.**
```
Based on the provided information, I can identify the relevant documents and extract the desired information.

Mualani has a total of 6 Constellations. Here they are listed in bullet points:

* C1: The Leisurely "Meztli"... - Increases the DMG dealt by the first Sharky's Surging Bite after Mualani enters Nightsoul's Blessing and the Shark Missiles it triggers, and reduces Phlogiston and Nightsoul point consumption while out of combat.
* C2: Mualani, Going All Out! - Grants 2 stacks of Wave Momentum when Mualani enters Nightsoul's Blessing state, and recovers additional Nightsoul points when obtaining Puffers within the same state.
* C3: Surfing Atop Joyous Seas - Increases the Level of Surfshark Wavebreaker by 3.
* C4: Sharky Eats Puffies - Regenerates Energy when Mualani obtains a Puffer, and increases Boomsharka-laka's DMG.
* C5: Same Style of Surfboard on Sale! - Increases the Level of Boomsharka-laka by 3.
* C6: Spirit of the Springs' People - Removes the limitation on triggering the DMG increase from "The Leisurely 'Meztli'...".

Please let me know if you'd like me to rephrase or clarify any part of my answer!
```

However, the application is still far from my ultimate goal. It contains some flaws, but part of the reason is due to the pre-processing stage and the embedding model. As mentioned in my other [repo](https://github.com/jian-li1/local-llama-rag?tab=readme-ov-file#future-improvements), the page content crawled by my crawler is messy, which often contain irrelevant text from nagivation bars and sidebars, for example. Moreover, the embedding model fails to query prompts with specific keywords, resulting in an entire list of irrelevant documents. One of the initial steps to addressing these issues could be cleaning the web pages to reduce junk and improve document retrieval. Overall, I believe this project has potential and I'll most certainly continue to look for solutions and make improvements to this application.