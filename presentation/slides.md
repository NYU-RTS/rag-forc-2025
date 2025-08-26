---
# You can also start simply with 'default'
theme: academic
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
#background: 
# some information about your slides (markdown enabled)
title: RAG @ FORC 2025
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
monaco: false
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# open graph
seoMeta:
  # By default, Slidev will use ./og-image.png if it exists,
  # or generate one from the first slide if not found.
  ogImage: auto
  # ogImage: https://cover.sli.dev

hideInToc: true
---

# Retrieval Augmented Generation
Foundations of Research Computing (FORC) Camp 2025


<div class="abs-br m-6 text-xl">
  <a href="https://github.com/NYU-RTS/rag-forc-2025" target="_blank" class="slidev-icon-btn">
    <carbon:logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->


---
hideInToc: true
---

# Table of contents

<Toc text-sm minDepth="1" maxDepth="1" />

---
layout: figure-side
figureCaption: "Context Rot"
figureFootnoteNumber: 1
figureUrl: context_rot.png
---
# Why not pass all the context to the LLM?

- LLM context windows are limited
- The advertised window =! usable window
- Even if it was, longer context -> slower responses, more expensive!

<Footnotes separator>
  <Footnote :number=1><a href="https://research.trychroma.com/context-rot" rel="noreferrer" target="_blank">From ChromaDB, Context Rot: How Increasing Input Tokens Impacts LLM Performance</a></Footnote>
  <Footnote :number=2><a href="https://arxiv.org/abs/2502.05167" rel="noreferrer" target="_blank">See also "NoLiMa: Long-Context Evaluation Beyond Literal Matching"</a></Footnote>
</Footnotes>


---
transition: fade-out
---

# What is Retrieval Augmented Generation?

<br>
<br>

Retrieval Augmented Generation is a technique to inject custom context to prompts when invoking an LLM. LLMs may be able to answer questions based on their training data, but the training 
data is frozen before the LLM is released for general use. 

<br>


<p style="color:#009b8a" v-click>
  <b>
    To effectively investigate the contents of your customized corpus, you need to be able to pass along the relevant bits to the LLM based on the query.
  </b>
</p>

<!--
Here is another comment.
-->

---
layout: default
transition: fade-out
hideInToc: true
---
# Overview

Here's what we are trying to achieve:
<br>

```mermaid
flowchart LR;
    A["Documents"]
    B["Indexed Corpus"]
    C["prompt without context"]
    D["prompt with added context"]
    E["response using context"]
    subgraph Ingestion
    A-- "Parse and Chunk" -->B;
    end
    subgraph Retrieval
    C-- "Retrieve from index" -->B;
    B-- "Retrieved chunks" -->D;
    end
    subgraph Augmented Generation
    D-- "LLM" -->E;
    end
```

<p style="color:#009b8a" v-click>
  <b>
    What is an <span v-mark="{at:1, type:'underline'}"> index </span> ?
  </b>
</p>

<p style="color:#009b8a" v-click>
  <b>
    How do we <span v-mark="{at:2, type:'underline'}"> retrieve </span> relevant bits from an index?
  </b>
</p>

<!--
Here is another comment.
-->


---
layout: figure-side
figureCaption: Overview of retrieval
figureFootnoteNumber: 1
figureUrl: architecture-biencoder.png
hideInToc: true
---

# Retrieval overview

- Subsets of documents (hereafter known as chunks) are encoded by an indexing algorithm
- These indices are then stored in a database
- Query is indexed by the same indexing algorithm
- Query index is compared with existing indices
- `k` document chunks that are similar to the query are returned

<Footnotes separator>
  <Footnote :number=1><a href="https://github.com/castorini/pyserini/blob/master/docs/conceptual-framework.md" rel="noreferrer" target="_blank">"From pyserini, conceptual overview"</a></Footnote>
</Footnotes>

---
layout: figure-side
figureCaption: Overview of retrieval with BM25
figureFootnoteNumber: 1
figureUrl: architecture-bm25.png
---

# BM25 indexing (Sparse Lexical indexing)

- The most popular search algorithm is "BM25" which represents document chunks as bag of (important) words, completely ignoring the order of words.

- Commonly occurring words like "the", "a", "their", etc are removed as they do not differentiate document chunks.  

- Documents and queries are stored as "sparse" vectors (vectors filled mostly with zeros, the non-zeros indicate which parts of the vocabulary are present).

<Footnotes separator>
  <Footnote :number=1><a href="https://github.com/castorini/pyserini/blob/master/docs/conceptual-framework.md" rel="noreferrer" target="_blank">"From pyserini, conceptual overview"</a></Footnote>
</Footnotes>

---
layout: figure-side
figureCaption: "VectorDB-as-a-library runs in notebooks/laptops with a pip install"
figureFootnoteNumber: 1
figureUrl: milvus-lite-1.png
---
# Milvus Lite

- Local, file-based vector database for processing up to a million chunks of text.
- Full text search out of the box (with opitonal enhancnments like phrase matching if needed).
- Integrates well with a wide variety of LLM tools/libraries.  

<br/>

<v-click>
  <img src="/milvus.png" alt="Both?" class="center" style="width: 100%; height: 25%">
</v-click>

<Footnotes separator>
  <Footnote :number=1><a href="https://milvus.io/" rel="noreferrer" target="_blank">"Milvus documentation"</a></Footnote>
  <Footnote :number=2><a href="https://milvus.io/blog/introducing-milvus-lite.md" rel="noreferrer" target="_blank">"Introducing Milvus Lite: Start Building a GenAI Application in Seconds"</a></Footnote>
</Footnotes>

---
layout: figure
figureCaption: Full text search with MilvusLite
figureFootnoteNumber: 1
figureUrl: milvus_bm25.png
hideInToc: true
---
# Full text search with MilvusLite

<Footnotes separator>
  <Footnote :number=1><a href="https://milvus.io/docs/full-text-search.md" rel="noreferrer" target="_blank">"From Milvus documentation"</a></Footnote>
</Footnotes>

---
layout: figure-side
figureCaption: Overview of Docling
figureFootnoteNumber: 1
figureUrl: docling_overview.png
---
# Docling pipeline
- Docling is an library that can be used to process HTML/PDF/DOCX/etc documents to chunks of text that can be indexed. 
- Docling converts all douments to a unified "DoclingDocument" format that can be optionally exported to Markdown. 
- Comes with an document structure aware chunker that can be customized.

<Footnotes separator>
  <Footnote :number=1><a href="https://docling-project.github.io/docling/" rel="noreferrer" target="_blank">"Docling simplifies document processing, parsing diverse formats and providing seamless integrations with the gen AI ecosystem."</a></Footnote>
</Footnotes>


---
layout: default
transition: fade-out
---
# Demo with NYU Library Research Guides!
- Compile a list of URLs for the reseasch guides
- Use docling to process HTML pages
- Use `MilvusLite` to create a BM25 index
- Try this yourself during the hands-on later!


---
layout: figure-side
figureCaption: Encoder-only transformer
figureFootnoteNumber: 1
figureUrl: embedding-models.png
---
# Encoder Transformers (embedding models)

- LLMs that generate text/images/audio/etc are decoder models, i.e. they are designed to generate content one token at a time. 
- Encoder LLMs on the other hand are used for pre defined tasks like generating embeddings for semantic search, clustering, classification, etc.

<Footnotes separator>
  <Footnote :number=1><a href="https://www.oreilly.com/library/view/hands-on-large-language/9781098150952/" rel="noreferrer" target="_blank">"We use an embedding model to convert textual input, such as documents, sentences, and phrases, to numerical representations, called embeddings." <br/> From Hands-On Large Language Models By Jay Alammar, Maarten Grootendorst</a></Footnote>
</Footnotes>

---
layout: default
transition: fade-out
hideInToc: true
---
# Embeddings demo

- Traditional indexing techniques did not account for contexts for a word, wherever it occurred the word was indexed the same way.  
- LLMs produce "contextual" embeddings that can capture the semantic meaning over just tracking the keyword occurrences. 

 <br/>
 <br/>

<blockquote>
  <p>
    Head over to this demo and see how embedding models cluster sentences (projected onto a 2D grid, the actual embedding dimensions are much larger): <a href=https://huggingface.co/spaces/webml-community/ettin-embedding-webgpu>https://huggingface.co/spaces/webml-community/ettin-embedding-webgpu</a>
  </p>
</blockquote>

---
layout: figure-side
figureCaption: Cosine search with embeddings
figureFootnoteNumber: 1
figureUrl: cosine-similarity.png
hideInToc: true
---
# Cosine search with embeddings

Once we have the embedding vectors for the text chunks and the query, we can compute the similarity scores for each (query, text-chunk) pair. 

With normalized vectors, we can skip computing the magnitudes and compute only the dot/inner product.


---
layout: default
transition: fade-out
---
# Demo comparing BM25 and semantic search
- The BM25 results retrieve keywords, but fail to account for intent of the query. For the demo query, it worked well.

- But, that may not always be the case <sup>1</sup>:
<div class="center">
  <img src="/croissants-are-breads.png" alt="Semantic search can mistake croissants for breads!" class="center" style="width: 75%; height: 75%">
</div>

<Footnotes separator>
  <Footnote :number=1> <a rel="noreferrer" target="_blank">"From https://fika.bar/paoramen/local-first-search-01K1B0WM1X4P5SV5QAES0Z5N75"</a></Footnote>
</Footnotes>

---
layout: two-cols-header
---
# Which one to choose?

:: left ::
<v-click> 
<img src="/which-one-to-choose.jpg" alt="Which one to choose?" class="center" style="width: 75%; height: 75%">
</v-click>

:: right ::
<v-click> 
<img src="/both.jpg" alt="Both?" class="center" style="width: 75%; height: 75%">
</v-click>

---
layout: figure
figureCaption: Hybrid Search
figureFootnoteNumber: 1
figureUrl: hybrid-search.png
---
# Hybrid Search

<Footnotes separator>
  <Footnote :number=1><a href="https://milvus.io/docs/multi-vector-search.md" rel="noreferrer" target="_blank">"From Milvus documentation"</a></Footnote>
</Footnotes>



---
layout: default
---
# RAG prompt

Now that we have learnt how to retrieve rerlant context, here's a prompt template you can use for RAG:
```py
    completion = portkey_client.chat.completions.create(
        model="gemini-2.5-flash",
        temperature=2.0,
        messages=[
            {
                "role": "system",
                "content": """Human: You are an AI assistant. You are able to
                    find answers to the questions from the contextual passage
                    snippets provided.""",
            },
            {
                "role": "user",
                "content": f"""Use the following pieces of information enclosed
                    in <context>  tags to provide an answer to the question
                    enclosed in <question> tags.
                    <context> {context} </context>
                    <question> {args.query} </question> """,
            },
        ],
    )
```

---
layout: figure-side
figureCaption: Cross-encoder vs Bi-encoder
figureFootnoteNumber: 1
figureUrl: biencoder-vs-crossencoder.png
---

# Advanced: Cross-encoders for reranking 
- Bi-encoders are fast, but lose accuracy
- Cross-encoders are slow, but are more accurate. They do this by comparing the tokens directly.
- They can be used as a filter during the second stage of a retrieval pipeline.

<Footnotes separator>
  <Footnote :number=1><a href="https://zilliz.com/blog/augmented-sbert-data-augmentation-method-for-improving-bi-encoders" rel="noreferrer" target="_blank">"Cross-encoders vs Bi-encoders"</a></Footnote>
  <Footnote :number=2><a href="https://ben.clavie.eu/ragatouille/#longer-might-read" rel="noreferrer" target="_blank">"Quick overview of their pros and cons of widely-used retrieval approaches"</a></Footnote>
</Footnotes>


---
layout: figure-side
figureCaption: Two stage retrieval
figureFootnoteNumber: 1
figureUrl: 2-stage-retrieval.png
---

# Advanced: Two stage retrieval
- Instead of relying solely on the RRF reranker, use a cross-encoder to re-rank.
- Use a bi-encoder to cast a wide net, use a cross-encoder to narrow it down to a few high quality chunks!

<Footnotes separator>
  <Footnote :number=1><a href="https://weaviate.io/blog/cross-encoders-as-reranker" rel="noreferrer" target="_blank">"Using Cross-Encoders as reranker in multistage vector search"</a></Footnote>
</Footnotes>


---
layout: default
figureCaption: Vision Language Models
---
# Demo: Advanced processing pipeline with VLMs
- Until now, we've been looking at embedding text, but documents contain images, tables, charts, etc!
- Vision Language Models can take images and text as input and generate text as output. 
- We will be using this to augment the existing data processing pipeline with an image captioning step!

---
layout: default
figureCaption: What about fine tuning?
---

# What about fine-tuning?
- Fine-tuning is an option of last-resort.
- Fine-tuning (especially larger LLMs) is expensive and thus needs justification in the form of an evaluation.
- You can also look into fine-tuning the embedding models for your courpurs by providing contrastive examples! This can enhance the context passed to the LLM and improve the output without having to fine-tune the LLM.

 <br/>
 <br/>

<blockquote>
  <p>
    Refer to this presentation on the trade-offs between RAG and fine-tuning! <a href=https://parlance-labs.com/education/fine_tuning/emmanuel.html>https://parlance-labs.com/education/fine_tuning/emmanuel.html</a>
  </p>
</blockquote>



---
layout: figure
figureCaption: ScholarQA by AllenAI
figureFootnoteNumber: 1
figureUrl: scholar-qa-allenai.png
---
# Advanced RAG example
Here's a RAG pipeline which incorporates many advanced techniques! Try out the live demo at https://asta.allen.ai/chat

<Footnotes separator>
  <Footnote :number=1><a href="https://asta.allen.ai/synthesize?redirect_from=corpus-qa" rel="noreferrer" target="_blank">"Ai2 Scholar QA is a system for answering scientific queries and generating literature reviews by gathering evidence from multiple documents"</a></Footnote>
</Footnotes>


---
layout: default
---
# Hands-on session

- Perform RAG on a dataset relevant to you
- Extend the tutorial example with a relevance check by updating the prompt or using a different LLM
- Check for hallucinations in the output against the retrieved context with [LettuceDetect](https://krlabs.eu/LettuceDetect/)
- Try using the vision transformers available in Docling on your documents


---
layout: center
class: text-center
---

# Reach out to Research Technology Services, https://services.rt.nyu.edu/


<PoweredBySlidev mt-10 />

