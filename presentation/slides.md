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
  <a href="https://github.com/slidevjs/slidev" target="_blank" class="slidev-icon-btn">
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
# Why not pass all the context to the LLM

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
---

# Retrieval overview

- Subsets of documents (hereafter known as chunks) are encoded by an indexing algorithm
- These indices are then stored in a database
- Query is indexed by the same indexing algorithm
- Query index is compared with existing indices
- Document chunks that are similar to the 

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
layout: figure
figureCaption: Full text search with MilvusLite
figureFootnoteNumber: 1
figureUrl: milvus_bm25.png
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
- Docling is an library that can be used to process document URLs/PDFs/etc to chunks of text that can be indexed. 
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
- Use MilvusLite to create a BM25 index
- Try this yourself during the hands-on later!


---
layout: figure-side
figureCaption: Encoder-only transformer
figureFootnoteNumber: 1
figureUrl: embedding-models.png
---
# Encoder Transformers (embedding models)

- LLMs that generate text/images/audio/etc are decoder models, i.e. they are designed to generate content one token at a time. 
- Encoder LLMs on the other hand are used for pre defined tasks like generating embeddings

<Footnotes separator>
  <Footnote :number=1><a href="https://www.oreilly.com/library/view/hands-on-large-language/9781098150952/" rel="noreferrer" target="_blank">"We use an embedding model to convert textual input, such as documents, sentences, and phrases, to numerical representations, called embeddings."</a></Footnote>
</Footnotes>

---
layout: quote
transition: fade-out
---
# Encoder Transformers (embedding models)

- Traditional indexing techniques did not account for contexts for a word, wherever it occurred the word was indexed the same way.  
- LLMs produce "contextual" embeddings that can capture the semantic meaning over just tracking the keyword occurrences. 

 <br/>
 <br/>

> Head over to this demo and see how embedding models cluster sentences (projected onto a 2D grid, the actual embedding dimensions are much larger): https://huggingface.co/spaces/webml-community/ettin-embedding-webgpu

---
layout: figure-side
figureCaption: Cosine search with embeddings
figureFootnoteNumber: 1
figureUrl: cosine-similarity.png
---
# Cosine search with embeddings

---
layout: figure-side
figureCaption: Hybrid Search
figureFootnoteNumber: 1
figureUrl: cosine-similarity.png
---
# Cosine search with embeddings



---
layout: figure-side
figureCaption: Vision Transformers
figureFootnoteNumber: 1
figureUrl: milvus_bm25.png
---
# Vision Transformers

---
layout: figure-side
figureCaption: Hands-on
figureFootnoteNumber: 1
figureUrl: milvus_bm25.png
---
# Hands-on session


---
layout: center
class: text-center
---

# Learn More


<PoweredBySlidev mt-10 />

