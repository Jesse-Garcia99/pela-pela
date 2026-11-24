# Team 21 - PelaPela

*Data Informed Language learning*

[Original Google Docs report (including figures)](https://docs.google.com/document/d/1kSuSfKDai4AaOQ3S8uhexfCstIUu_LJhNv4zgUPlP58/edit)

> Markdown transcription of the original project report; embedded figures remain in the original document.

## Background

Before a trip to Japan I started learning Japanese using a variety of non-traditional learning platforms like Duolingo, Youtube, and increasingly ChatGPT. I have never thrived in traditional learning environments, which is part of the reason I chose to pursue an online masters degree, and I find the learn at your own pace model to work well for me.

The problem I identified in this approach is that though each tool had its own benefits, there was one drawback that was severely limiting my ability to efficiently master the language. I didn't know what I didn't know, and often found myself missing knowledge in very basic topics that one might learn in a traditional language course or class.

So I sought to solve this problem, by creating a “skill tree for language” like you might find in a video game. The concept is the same: accomplish a goal, earn the skill! It’s an ambitious vision that will take years to fully realize, but my first step is to build a network of Japanese language concepts represented as nodes, with connections (edges) being mapped using Japanese language python libraries like GINZA.

This approach helps break down the siloed data problem across tools, and gives learners a roadmap for learning that is easily understandable, interactive, and engaging. By mapping connections and suggesting what to learn next, learners can naturally explore concepts and related topics in a game-ified way where you want to explore and learn more.

A 2024 Smart Learning Environments study found that adaptive learning systems like pelapela, improved both student performance and satisfaction through illuminating knowledge gaps and reinforcing concepts at the right time(Contrino et al).

My goal for this project is to apply that methodology to a graph you can interact with directly. You search a term, see how it connects to related grammar, forms, and usage, and get a short definition or examples when you click on the node. I am using modern graph-viz patterns to keep the map readable and useful, so prerequisites, related topics, and recommended  next steps are visible to the user.

The ultimate goal after this project is to expand the network into a vector database so it’s accessible to a Large Language Model (LLM). That enables Retrieval-Augmented Generation (RAG) for grounded explanations and dynamic, per-node familiarity scores that update as you learn. Those scores will drive personalized recommendations for YouTube videos, generated quizzes and flashcards, and games based on your prior learning history.

## Methodology

Our pipeline uses pathlib for cross-platform paths, spaCy with ja-ginza for Japanese POS(parts of speech) morphology and text normalization, networkx for graph construction and analysis, and jsonschema to enforce schemas and preserve UTF-8 text.

I constructed the network by first grouping nodes by JLPT level, part of speech, and semantic tags, then adding weighted edges that reflect multiple signals: JLPT proximity, POS compatibility, tag similarity, and example co-occurrence. This allows the graph to encode both linguistic structure and usage patterns. The front end is implemented in D3.js with responsive layouts, viewport culling, and level of detail rendering to keep large graphs smooth.

Quality is monitored with automated metrics like precision, recall and Cohen’s κ; topology checks like components, degree distributions, communities; and reproducibility via edge set Jaccard across rebuilds. The graph is designed to scale by applying a modular pipeline, versioned JSON schemas, built in profiling for time/memory, and optional parallel cleaning to absorb new sources and expand the graph while ensuring stable behavior.

I started by assembling as many freely available Japanese language learning materials as possible including JLPT grammar lessons (jlptsensei.com); duolingo guidebook lesson snippets scraped from their API; and Anki flashcards. The pipeline code (build_clean_json_fixed_pipeline.py) does the following:

preserve_jlpt_quality() protects already-clean JLPT content from accidental degradation during enrichment (e.g., avoids overwriting good meanings with generic boilerplate).

classify_grammar_pattern() maps noisy titles to stable semantic buckets (Ability & possibility or Appearance & visual impressions) used for faceted search, evaluation, and visualization.

merge_related_jlpt_entries() deduplicates patterns like 〜てくる, 〜てみる, consolidating examples and preferring high signal meanings

detect_truncated_titles() repairs partial Japanese titles and artifacts with numbers leading

Since each data source is formatted differently and contains slightly different fields and formats, it was important to create JSON objects for each type of sentence object so that the entries were uniform. I went through source by source and loaded the data into the appropriate JSON objects.

### Data Parsing and Cleaning

#### JLPT

Issues: Mixed formatting meanings, truncated titles ending in particles, “NaN” strings present,  duplicates with other variants.

Fix: Run multiple passes of xxx First pass preserves originals, second pass repairs numbered or empty meanings, third pass merges semantically identical items and removes weak English by inferring from the Japanese example. The validator runs after each transform to ensure monotonic quality.

#### Duolingo

Issues: Titles sometimes generic “Practice” examples nested under different keys, inconsistent difficulty tags.

Fix: Promote the first Japanese example to title_ja when it’s more informative, strip generic prefixes from meaning, normalizes examples to Japanese and English and re-tag difficulty as beginner, intermediate, or advanced.

#### Anki

Issues: High variance in card note types and fields, custom formatting, cloze deletions.

Fix: A conservative cleaner copies fields as is initially, then progressively merges into title, meaning, examples, tags while keeping raw fields for traceability. I then reran the importer with increasingly strict schema assertions to catch malformed cards early.

### Japanese NLP: tokenization and parsing

Unlike English, Japanese doesn’t have spaces between words, so both people and software rely on kanji and particles to infer the boundaries between words. I applied spaCy’s Japanese pipeline SudachiPy & GiNZA to break sentences into words, tag parts of speech, and map simple grammar links. This is relevant because even a basic sentence like “大阪に行きます” might be differently by different tools: [大阪][に][行きます] or [大阪に][行きます]. This small difference can interfere with node generation, duplicate detection, and formation of graph links.

Throughout the project I run the same tokenizer before any cleaning or deduping, and I lock one “split mode” so word boundaries don’t shift between runs. I also fix any English examples that are actually romaji (japanese words written in the roman alphabet) by generating a short, real English gloss from the Japanese sentence. With this parsing and tokenization our data stays aligned, our graph edges are reliable, and the search returns more reliable results.

### Graph construction & visualization

Our nodes consist of grammar constructions, function words, patterns (e.g., “appearance & visual impressions”), and example sentences; while our edges capture pre-requisite, contrast, morphological neighbor, and topic co-occurrence relations.

For layout and rendering, I followed Cambridge Intelligence’s graph-viz guidance to:

Avoid hairballs: started with scoped neighborhoods, filtered high degree hubs, aggregated leaf clusters; enabled drill downs.

See through snowstorms: enriched sparse nodes and added attributes, grouped by community or unit, and de-duplicated.

Tame starbursts: collapse leaf combos around overly dominant hubs like very common particles then expand on demand.

## Evaluation

I evaluated the graph based on: connection correctness, coverage and connectivity, reproducibility, and annotation reliability. To assess correctness, I created a uniformly stratified sample of roughly 400 edges spanning the main relation types: prerequisite, contrast, morphological neighbor, and topical co-occurrence. Two independent annotators reviewed each sampled edge, judging both whether the connection itself was valid and if the direction of the edge was appropriate.

Furthermore I calculated edge precision with 95% confidence intervals and measured direction accuracy specifically on directed links. To capture agreement between annotators, I used Cohen’s κ, which allowed me to detect where judgments aligned or diverged. Cases of disagreement were then adjudicated, and the underlying error patterns were examined in detail. This review looked at which source corpus the edge originated from, which Sudachi split mode was used during tokenization, and which type of pattern was being connected. This allowed me to identify weaknesses and feed the findings into thresholds and extraction rules for subsequent builds.

On connection correctness, precision of 0.755, indicated that most sampled links were found to be valid. Directionality on directed edges was 1.000, indicating all assessed directional links pointed as the annotators expected. κ for both validity and direction was −0.139, which flags inconsistent interpretation on harder edge types (subtle contrasts and morphological neighbors). I’ve captured these disagreements and their rationales to refine the labeling and examples before the next pass.

The defined “everyday core” of the graph is fully present (core coverage = 100.000%) with no isolated nodes (orphans_share = 0.000). A single dominant component contains 92.6% of nodes (main_component_share = 0.926), so most concepts are reachable within a few clicks without getting trapped in cul-de-sacs. A qualitative audit confirmed that common particles were the main sources of oversized hubs; these were filtered or aggregated so the map remains readable without obscuring genuine relationships.

Reproducibility is excellent with tokenizer and parser versions pinned and seeds fixed, rebuilds produced an identical edge set (edge_jaccard = 1.000), indicating that recent stability fixes to tokenization, merging, and edge construction are working as intended. Overall, the network build is well connected and has meaningful connections. The main improvement I need to make is to raise edge precision and improve annotator agreement on the more difficult relation types so future runs better align with human expectations.

## Ethical considerations

A central challenge of this project is the use of freely available resources such as JLPT study guides, Duolingo lesson snippets, and Anki flashcards to generate the network. While these sources are public, if the data is not used transformatively it can be considered redistribution of copyright material. It is important to both cite all sources, and to use the data in a transformative way when it is distributed to end users. Meaning that the network can be informed by these data explanations, and the act of harvesting them raises questions of intellectual property and fair use.

Where possible, I regenerate or reformat examples by rewriting weak glosses, standardizing style, or producing synthetic examples so the final product is derivative, not duplicative. Ultimately the network will be open infrastructure enriched with synthetic and community contributed examples rather than scraped third party text.

## Results

Currently our network is a searchable map of Japanese concepts where each node (grammar pattern, function word, example) links to related ideas you can follow in a few clicks.

*Figure — see the [original report](https://docs.google.com/document/d/1kSuSfKDai4AaOQ3S8uhexfCstIUu_LJhNv4zgUPlP58/edit).*

Clusters are naturally formed in ways that make sense to learners ( ability, concession, appearance) with clear bridge nodes that connect clusters.

*Figure — see the [original report](https://docs.google.com/document/d/1kSuSfKDai4AaOQ3S8uhexfCstIUu_LJhNv4zgUPlP58/edit).*

The graph is already useful as a study aid: you look up one concept, see nearby neighbors, prerequisites, and get a short explanation or example without moving across multiple tabs.*Figure — see the [original report](https://docs.google.com/document/d/1kSuSfKDai4AaOQ3S8uhexfCstIUu_LJhNv4zgUPlP58/edit).*

I feel that the model successfully demonstrates the concept and will allow me to get a better understanding of how to further develop this data analysis as a learning tool for my fellow language learners. With this version of the network, you can look up a concept, see what it connects to, trace quick prerequisites, and notice the gaps you didn’t know you had. From here, I can build a robust set of interactive materials on top of the same backbone: procedurally generated drills, crowd sourced examples, and professionally written lessons. Everything plugs into the graph and nudges learners toward the next best step. That’s exactly the adaptive-learning loop I cited earlier surface gaps, reinforced at the right moment, which has been shown to improve performance and satisfaction amongst learners (Contrino et al., 2024).

## Future work

Looking forward, this network also lays the groundwork for building a lightweight, domain specific LLM on top of it. Rather than relying entirely on general purpose chat models, I can fine tune or constrain a smaller model to translation and explanation tasks, using the network as its grounding. As suggested by my project mentor Winston Featherly-Bean, I will incorporate promptfoo to systematically test and evaluate prompt performance across settings. This will allow me to ensure the model produces consistent, useful translations in different modes (English explanation, Japanese explanation, Romaji, Kana, or Kanji with furigana), and to track reliability as the system expands.

Beyond consistency testing, the network structure itself provides a unique advantage to other LLM models that would be focused on language learning.  By mapping semantic relationships, and creating RAG architecture for our LLM, I can give the model context that is much more valuable to the end user than a general language learning model.

As the network and model continue to evolve, I will continue to explore additional integrations such as curriculum aware prompting (difficulty increases based on level or progress) and personalization (adjusting outputs based on a user’s preferred learning style). The combination of structured network data, domain specific fine tuning, and systematic evaluation will help build a specialized tool that is not only accurate but also deeply aligned with the way learners actually study languages.

## References

Smart Learning Environments. (2024). Using an adaptive learning tool to improve student performance and satisfaction. https://slejournal.springeropen.com/articles/10.1186/s40561-024-00292-y

Cambridge Intelligence. (2023, Oct 5). Graph visualization and artificial intelligence (White paper). https://cambridge-intelligence.com/white-paper-graph-visualization-and-ai/

Cambridge Intelligence. (2019, Nov 25). Graph visualization: fixing data hairballs. https://cambridge-intelligence.com/how-to-fix-hairballs/

Cambridge Intelligence. (2019, Dec 16). Graph visualization: see through the snowstorm. https://cambridge-intelligence.com/seeing-through-the-snowstorm/

Cambridge Intelligence. (2020, Jan 20). Graph visualization: dealing with starbursts. https://cambridge-intelligence.com/dealing-with-starbursts/

Megagon Labs. (2024). ja-ginza (5.x) – Japanese spaCy model (PyPI page). https://pypi.org/project/ja-ginza/

Megagon Labs. (n.d.). GiNZA: A Japanese NLP library using spaCy (GitHub). https://github.com/megagonlabs/ginza

Sassano, M. (2002). An empirical study of active learning with support vector machines for Japanese word segmentation. ACL. https://aclanthology.org/P02-1064.pdf

spaCy. (n.d.). Models & languages: Japanese – SudachiPy and split modes. https://spacy.io/usage/models

Amazon Web Services. (2024, Jul 8). Introducing Amazon Titan Text Embeddings V2 with up to 4,096-dimension vectors (pricing: $0.02 per 1M tokens). https://aws.amazon.com/blogs/machine-learning/introducing-amazon-titan-text-embeddings-v2/

U.S. Copyright Office. (n.d.). Fair use FAQ & Circulars. https://www.copyright.gov/help/faq/faq-fairuse.html; https://www.copyright.gov/circs/

hiQ Labs, Inc. v. LinkedIn Corp., 31 F.4th 1180 (9th Cir. 2022). https://cdn.ca9.uscourts.gov/datastore/opinions/2022/04/18/17-16783.pdf

Directive 96/9/EC of the European Parliament and of the Council of 11 March 1996 on the legal protection of databases. https://eur-lex.europa.eu/eli/dir/1996/9/oj/eng (PDF consolidated 2019).

Wang, Y., et al. (2023). Self-Instruct: Aligning language models with self-generated instructions. ACL. https://aclanthology.org/2023.acl-long.754.pdf

JLPT Sensei. (n.d.). https://jlptsensei.com/

Duolingo. (n.d.). https://duolingo.com

OpenAI. (2025, Aug 18). ChatGPT (GPT-5 Thinking). https://chat.openai.com/

Cursor. (n.d.). The AI Code Editor. https://cursor.com/
