---
## Architecting and Evaluating an AI-First Search API

Building a scalable Search API that handles 200 million daily queries using hybrid retrieval and intelligent context curation for AI models

*Building a scalable, AI-forward Search API presents unique research challenges at the intersection of information retrieval, AI, and system design. Unlike traditional services that operate on circumscribed data, a search API must continuously index billions of documents, understand query intent across languages and domains, and surface relevant results within milliseconds. In a world of AI agents, a search API must also carefully curate the **context** provided to developers’ models to ensure those models perform optimally and incorporate maximally accurate information. Effective context engineering requires the API to not only capture document-level relevance, but to treat the individual sections and spans of documents as first-class units in their own right.*

*In this article, we present the design and evaluation of the [Perplexity Search API](https://www.perplexity.ai/api-platform). We first explore the architecture and key technical decisions behind our production search infrastructure that processes 200 million daily queries. Our architecture combines hybrid retrieval mechanisms, multi-stage ranking pipelines, distributed indexing, and dynamic parsing through a self-improving content understanding module.*

*We then describe our approach to empirically evaluating search API performance through a newly open-sourced evaluation framework. Through comprehensive benchmarking alongside existing commercial APIs, we demonstrate state-of-the-art performance on both quality and latency. The Perplexity Search API is available for developers worldwide starting today.*

* [Architecture & Design](https://research.perplexity.ai/articles/architecting-and-evaluating-an-ai-first-search-api#:~:text=Building%20a%20search%20system%20optimized%20for%20AI%20models)
* [Evaluations](https://research.perplexity.ai/articles/architecting-and-evaluating-an-ai-first-search-api#:~:text=Establishing%20a%20new%20frontier%20for%20quality%20and%20performance)
* [Find search_evals on GitHub](https://github.com/perplexityai/search_evals)
* [Get Started on API Platform](https://www.perplexity.ai/api-platform)

---
## Building a search system optimized for AI models

*Since our founding, we understood that robust search capabilities would be critical to enable the category-defining AI products we wanted to ship. Originally, we hoped that these capabilities already existed in a publicly available form. We experimented with the portfolio of search APIs available in late 2022, and the [very first version of Perplexity’s answer engine](https://x.com/perplexity_ai/status/1600551871554338816) actually incorporated some of these offerings. Specifically, we would analyze user queries, formulate and send requests to these APIs, and gather the results as context for our downstream generation pipeline.*

*However, as users flocked to Perplexity, we quickly realized that these APIs weren’t going to scale with our growth. The prices we encountered were exorbitant, with one leading API provider charging [$200 per thousand queries](https://web.archive.org/web/20230217061754/https://www.microsoft.com/en-us/bing/apis/pricing-update). But more fatally, these legacy APIs simply weren’t designed for the new retrieval paradigms introduced by frontier AI systems.*

### One of the earliest versions of Perplexity’s answer engine.

*For starters, the request and response formats of these legacy APIs were designed for a world in which humans would be the exclusive consumers of search results. As a result, these APIs prioritized document-level results, to the exclusion of more fine-grained information revealed by sub-document units. The success of an AI model turns on the quality and compactness of the provided context, and we saw firsthand how overbroad retrieval could derail AI models, regardless of how much the retrieved documents as a whole might appear relevant. Our attempts to shoehorn precision onto imprecise results were met with limited success.*

*We also observed that the legacy APIs were either too stale or too slow. Independently developed search APIs relied on indices that were incomplete and too infrequently updated. Scraping-based APIs, such as the various unofficial SERP APIs for Google Search, benefitted from a more comprehensive index but offered unacceptably high latency due to the inefficiency inherent in querying a third-party search engine. High latency made APIs unsuitable for user-facing applications, while staleness rendered the result sets unusable for any AI system hoping to be accurate and trustworthy.*

*Finally, we observed the AI ecosystem bifurcate between lexical search and semantic search, with systems emerging that were over-optimized for one modality over the other. We saw the rise of vector databases that couldn’t adequately support even simple boolean filters. Traditional search backends fared little better, with primitive support for the semantic ranking and retrieval opportunities unlocked by AI.*

*As we realized that making Perplexity even better would require building our own dedicated search infrastructure, we distilled our observations into three key criteria:*

* **Completeness, freshness, and speed**: the search index must be comprehensive, constantly updated, and engineered for realtime latency.
* **Fine-grained content understanding**: AI model brittleness and limited context windows require search results to be presented and ranked at the most granular level possible.
* **Hybrid retrieval and ranking**: both the retrieval and ranking pipelines must incorporate lexical **and** semantic signals for the results to be useful for AI.

### Perplexity today, powered by our internet-scale search infrastructure now available via Perplexity Search API.

---
## Crawling and indexing infrastructure

*Operating at Perplexity scale requires infrastructure that can comprehensively map the web and respond to its constant evolution. Our search index tracks over 200 billion unique URLs, with capacity to track many hundreds of billions more.*

*Ensuring that our index covers a sufficiently comprehensive swath of the internet **and** that each document reflects the latest state is a difficult feat. There exists an inherent tension between [completeness and freshness](https://mitmecsept.wordpress.com/wp-content/uploads/2018/05/stefan-bc3bcttcher-charles-l-a-clarke-gordon-v-cormack-information-retrieval-implementing-and-evaluating-search-engines-2010-mit.pdf): for a fixed indexing budget, refresh operations for existing pages must compete with indexing operations for new unvisited pages. We balance this tension through a combination of scaling (which relaxes the optimization constraint by increasing the indexing budget) and ML-driven prioritization (which ensures that each indexing operation is maximally valuable to the index as a whole).*

*We’ve built an exabyte-scale index and crawling apparatus that scales horizontally with both size and query volume. Perplexity’s crawler and indexing fleets collectively comprise tens of thousands of CPUs and hundreds of terabytes of RAM, with considerable headroom for further scaling. We marry this compute infrastructure with multi-tier storage: over 400 petabytes in hot storage, with the balance in cold and warm tiers. We use both learned models and tuned heuristics to determine which documents should be kept hot, prioritizing documents from authoritative domains, documents involving undercovered topics, and other categories with outsize potential to enrich our index.*

*This infrastructure enables Perplexity to process tens of thousands of indexing operations per second. But even with this relatively generous budget, the internet’s scale is far too vast to index every single URL on a frequent and recurring basis. This is where ML comes in. We train an ML model to predict whether any given candidate URL needs to be indexed. The training objective is crafted to ensure that we calibrate our indexing decision to both the importance and likely update frequency of the specific URL. We also rely on ML to tell us precisely **when** to schedule the indexing operation. For sites with routine publication and refresh cadences, this allows us to defer operations to the point in time that they are likely to be most useful. Through continual refinement of these ML models, we’re able to maintain an index that meets the mark on both completeness and freshness.*

*Although our crawling infrastructure easily scales horizontally, we’re aware that website operators face differing technical constraints. We are careful to maintain a predictable and manageable rate of requests to any given domain. PerplexityBot, our search crawler, complies with explicit limits set forth in robots.txt files, or in the absence of domain-specified limits, industry-standard norms concerning request rates. We also adjust our crawling behavior when we detect site unavailability. Our distributed system is architected to enforce these limits even across tens of thousands of concurrent operations.*

---
## Self-improving content understanding at internet scale

*Even with one of the largest and freshest search indices on the planet, the result quality of each query hinges on our system’s understanding of each document’s content. Here, we face a fundamental challenge: the web is a messy place. No fixed set of parsing rules can handle the infinite variety of layouts, structures, and styles found across the entire internet.*

*We solved this challenge by developing a content understanding module that parses websites and extracts semantically meaningful content using dynamic rulesets. A website heavy on well-structured data in list or table form may benefit from more formulaic parsing and extraction rules, whereas another website primarily featuring freeform user-generated content may require a looser ruleset. Because the web is mostly static across short timeframes, we find that the lifespan of a given rule is relatively long. But as sites evolve over time and as we learn more about the best ways to extract information, our rulesets require continuous gardening as well.*

*To facilitate this dynamic adaptation, we turn (of course) to AI. At Perplexity’s scale, we observe an incredibly rich cross-section of the web in the course of serving millions of user queries hourly. This forms the foundation for an iterative self-improvement loop. We observe the parsing behavior and extracted outputs of our current rulesets across our query universe. We then use frontier LLMs to assess the performance across two primary dimensions:*

* **Completeness**: our index should receive as much semantically meaningful content as possible, while avoiding spurious extraction of uninformative or nonsensical matter.
* **Quality**: our index relies on content that’s high quality in both substance and form. Information should be captured in a manner that preserves the original content structure and layout.

*Note that completeness and quality present another inherent tension: manually optimizing for quality is possible, but the resulting parsing rules may fail to capture critical content. Similarly, we could handcode permissive rules that apply minimal filtering to the raw content, which might improve completeness but at the expense of quality. This is why we turn to AI-driven self-improvement to evolve our ruleset in a way that can optimize for both at once in a domain-adaptive manner.*

*Concretely, after each LLM-driven assessment of parsing performance, our system formulates a set of proposed ruleset changes to address the most significant error classes. These rulesets undergo further validation, after which we deploy the new version to our indexer. Given that improvements in parsing and extraction logic can only benefit future indexing operations, our investments in ML-optimized reindexing are especially important: continually refreshing high-traffic documents using the latest version of this logic ensures a tight self-improvement loop.*

*These rules matter not only for baseline parsing performance but also for segmentation into sub-document units. When it comes to frontier LLMs, each token of context is precious. A large body of prior work has documented how LLM performance degrades with increases in context size and the pollution of context with irrelevant information. Having well-structured, comprehensive parsing allows us to decompose documents into self-contained spans, each of which can be individually retrieved and ranked at query time.*

---
## Retrieval and ranking pipeline

*Modern search requires ensuring both completeness and relevance under a tight latency budget. Perplexity’s retrieval and ranking pipeline achieves this through a multi-stage architecture that progressively refines results with a series of increasingly sophisticated modules. We solve for potential bottlenecks by horizontally scaling any would-be chokepoints to the maximum extent possible.*

1.  The pipeline begins with retrieval from our index. We do not force a choice between lexical and semantic retrieval. Rather, we query the search index via both modalities and merge the results into a hybrid candidate set. At this stage, the priority is comprehensiveness over precision - subsequent stages of the pipeline will optimize for precision.
2.  The pipeline continues with prefiltering stages, in which we apply basic heuristics and first-cut filters to remove clearly non-responsive or stale content from the candidate set.
3.  We then employ multiple stages of progressively advanced ranking. Earlier stages rely on lexical and embedding-based scorers optimized for speed. As the candidate set is gradually winnowed down, we then use more powerful cross-encoder reranker models to perform the final sculpting of the result set. Thanks to our numerous investments in inference performance, we’re able to leverage state-of-the-art model architectures for these later stages while keeping overall ranking latency low.

*Throughout this pipeline, we retrieve and score results at both the document and sub-document levels. As discussed above, AI requires careful context engineering. Our pipeline is designed to surface the most atomic units possible to the model, such that downstream consumers can incorporate those precise units without pulling in irrelevant context from elsewhere in the source documents.*

*Ensuring high quality throughout each stage of this pipeline is possible only through co-design with Perplexity’s portfolio of products. Every detail of the ranking stack is designed to leverage the rich signal produced by the millions of user requests we serve each hour of the day. The vast majority of this signal is automated, with each generated Perplexity answer providing an empirical datapoint on search quality as used in a live agentic workflow. We combine AI-driven evaluations with live human feedback to enrich the training data for our embedding and ranking models far beyond what any manually constructed dataset can offer.*

*This synergy between search and product is unique to Perplexity. Legacy search engines are limited to the coarse signal provided by blue-link telemetry. Newer search offerings have attempted to implement some of these AI-forward concepts, but they lack real world environments that can validate search quality and provide the richness of signal needed for rapid improvement loops. We’re fortunate to have the missing ingredient: a live proving ground for search quality, powered by frontier LLMs and continually battle-tested by millions of curious users worldwide.*

---
## Establishing a new frontier for quality and performance

*After building the [Perplexity Search API](https://www.perplexity.ai/api-platform) and [SDK](https://docs.perplexity.ai/guides/perplexity-sdk) to make our search infrastructure available to developers worldwide, we wanted to understand how the various commercial API offerings measured up in real-world AI use cases. It has become commonplace to evaluate AI models themselves, but we needed a twist on this pattern: to evaluate the quality of search APIs when invoked by AI models.*

*We first looked to the open-source ecosystem to understand the prior art. We found many of the key ingredients for rigorous evaluation, but no framework that integrated these ingredients into a plug-and-play benchmarking solution. We therefore set out to develop an evaluation framework that could be used not only by Perplexity, but by any researcher or developer seeking to benchmark search APIs.*

*Using this evaluation framework, we rigorously benchmarked Perplexity Search API alongside other offerings. Optimizing search performance is complex, with many nuanced tradeoffs between quality and speed. In a win for developers, we found that Perplexity Search API sets a new state-of-the-art for both quality and latency. Developers no longer need to choose between building blazing-fast applications and ensuring high-quality results: with Perplexity, they can have both.*

---
## Designing reliable evaluations for agentic search workflows

*We began by defining three guiding principles for our evaluation framework:*

1.  **Spectrum of complexity**: the evaluation framework should measure performance on both straightforward and complex agentic AI use cases.
2.  **Neutrality**: the evaluation framework should assess all APIs on equal footing.
3.  **Simplicity**: although certain use cases might be complex, the framework itself should be as simple as possible so that others can understand it, reproduce the results, and extend the framework with minimal tedium.

*Our lightweight evaluation framework, [search_evals](https://github.com/perplexityai/search_evals), achieves these goals through a simple modular design. We developed two AI agent harnesses to capture both classical single-shot search use cases and advanced agentic workflows (exemplified by various “deep research” products widely available in the consumer marketplace). Both harnesses were implemented in a straightforward manner that leaves little room for hidden advantage.*

*To drive our evaluations, we adopted four benchmarks: [SimpleQA](https://arxiv.org/abs/2411.04368) and [FRAMES](https://arxiv.org/abs/2409.12941) for single-step search, along with [BrowseComp](https://arxiv.org/abs/2504.12516) and [HLE](https://arxiv.org/abs/2501.14249) for deep research. Collectively, these benchmarks provide well-accepted yardsticks for knowledge agents. Following established convention, we grade all benchmarks using the same prompted classifier methodology used in the original work. The precise benchmark implementations (including grader logic) can be [reviewed](https://github.com/perplexityai/search_evals/tree/main/search_evals/suites/) in our GitHub repo.*

*We designed the framework with reproducibility and extensibility in mind. Both the task-level scores and and the underlying agent outputs are stored with each run. Additional harnesses, benchmarks, and search API offerings can be easily integrated. Researchers and developers can read our GitHub [docs](https://github.com/ppl-ai/search_evals/blob/main/docs/DEVELOPMENT.md) for more information on extending the framework to address your precise needs.*

---
## Evaluation results

*Armed with this evaluation framework, we set out to assess our Search API in comparison with other API offerings. Across all tested benchmarks, Perplexity stands as both the fastest and highest-quality API on the marketplace.*

*We found that our above-described investments in index, retrieval, and ranking efficiency paid off handsomely. Perplexity’s median latency clocks in at **358ms**, over 150ms ahead of the second-fastest provider. 95th-percentile latency stays comfortably under **800ms**. With the long TTFT (time-to-first-token) metrics of today’s frontier models, every millisecond matters in creating delightful user experiences. We’ve spent the past two years optimizing latency for our own users, and we’re gratified that those same investments will now benefit the developer community as well.*

### Search API Latency
(requests initiated from AWS us-east-1)

| | *p50* | *p75* | *p90* | *p95* |
| :--- | :--- | :--- | :--- | :--- |
| **Perplexity** | **358ms** | **443ms** | **604ms** | **763ms** |
| *Exa* | *1375ms* | *1499ms* | *1798ms* | *2188ms* |
| *Brave* | *513ms* | *637ms* | *728ms* | *808ms* |
| *SERP Based\** | *1342ms* | *1586ms* | *1743ms* | *1790ms* |

*\* Many search API vendors use SERP scraping as the basis for their results. To capture the performance of such approaches, we use a representative Google SERP-based API offered by Tavily.*

*This speed advantage doesn’t come at the expense of quality. Perplexity Search API yields state-of-the-art performance in both single-step search and deep research settings. On each tested benchmark, Perplexity leads the pack in providing high-quality results that can power AI workflows of varying complexity. The margin between Perplexity and the best alternative is especially pronounced on SimpleQA and HLE, indicating that these results are robust across the spectrum of task complexity.*

*As with latency, our leadership in search quality flows downstream of our leadership in building delightful, highly trafficked AI products. Our millions of users use Perplexity to satisfy their curiosity on every topic imaginable, and the resulting quality and relevance signals allow our search models to improve with each passing hour.*

### Search API Quality Benchmarks

| | *Single-step search* | *Deep research* |
| :--- | :--- | :--- |
| | *SimpleQA* | *FRAMES* | *BrowseComp* | *HLE* |
| **Perplexity** | **.930** | **.453** | **.371** | **.288** |
| *Exa* | *.781* | *.399* | *.265* | *.242* |
| *Brave* | *.822* | *.320* | *.221* | *.191* |
| *SERP Based\** | *.890* | *.437* | *.348* | *.248* |

*We’re still in the early days of AI-native search engines, and the art and science of measurement will no doubt evolve. We invite the community to contribute by developing rigorous and creative methods for evaluating search engines in the age of AI agents.*

---
## The building block for AI’s next chapter

*The future of agentic systems and intelligent assistants will hinge on the availability of rich programmatic interfaces into the world’s collective knowledge. Today’s legacy search engines have saturated in usage at roughly $10^{10}$ queries per day, a figure that has likely peaked. The next generation of search and retrieval systems will serve orders of magnitudes more requests, as an ever-growing universe of AI agents relies on them for accurate realtime information.*

*Our efforts to date have positioned us as the fastest, most performant search engine accessible via API. But even more ambitious innovation will be needed for us to fuel AI’s next chapter. How do we efficiently scale these systems in the face of ever-increasing demand? How can these search systems be optimized in light of novel approaches to context engineering and posttraining? And how do we navigate the ever-present tension between comprehensiveness, recency, and latency? These are open questions that Perplexity, with our combination of product footprint, scale, and [technical talent](https://research.perplexity.ai/#careers), is uniquely positioned to tackle.*